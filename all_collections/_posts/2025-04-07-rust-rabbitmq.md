---
layout: post
title: Learning Rust - RabbitMq and Examples
date: 2025-04-07
categories: ["programming", "rust"]
---

### How to Connect to a Queue
Here is an example using `lapin` crate with `tokio` on how to create a connection to rabbitmq and establish a queue . 
```rust
use lapin::{options::*, types::FieldTable, BasicProperties, Connection, ConnectionProperties};
use tokio::task;
use futures_util::stream::StreamExt;

const QUEUE_NAME: &str = "rust_queue";

#[tokio::main]
async fn main() {
    let amqp_addr = "amqp://localhost:5672/%2f";
    
    // Connect to RabbitMQ
    let connection = Connection::connect(amqp_addr, ConnectionProperties::default())
        .await
        .expect("Failed to connect to RabbitMQ");
    
    let channel = connection.create_channel().await.expect("Failed to create channel");

    // Declare queue
    channel.queue_declare(
        QUEUE_NAME,
        QueueDeclareOptions::default(),
        FieldTable::default(),
    ).await.expect("Queue declaration failed");
}
```

As you can see we've established the connection using default properties, and the connection. Then using our `channel` we create the queue. Now what about a consumer? 

In this example we can clone our existing channel but normally you would have a separate process doing this. 
```rust
    // Spawn a consumer in a separate task
    let consumer_channel = channel.clone();
    task::spawn(async move {
        let consumer = consumer_channel
            .basic_consume(
                QUEUE_NAME,
                "rust_consumer",
                BasicConsumeOptions::default(),
                FieldTable::default(),
            )
            .await
            .expect("Failed to start consumer");

        println!("Waiting for messages...");
        consumer
            .for_each(|delivery| async {
                if let Ok(delivery) = delivery {
                    let message = String::from_utf8_lossy(&delivery.data);
                    println!("Received message: {}", message);
                    consumer_channel
                        .basic_ack(delivery.delivery_tag, BasicAckOptions::default())
                        .await
                        .expect("Failed to ack message");
                }
            })
            .await;
    });
```

And then to publish to our queue using the `lapin` crate we can do something like this
```rust
    // Publish a message
    let payload = b"Hello from Rust!";
    channel
        .basic_publish(
            "",
            QUEUE_NAME,
            BasicPublishOptions::default(),
            payload.to_vec(),
            BasicProperties::default(),
        )
        .await
        .expect("Failed to publish message");
```

### Handling Failures
This is a common situation where maybe something like network connection is inconsistent or the database connection is down, or a process restart. These need to be thought through and handled appropriately. 

Here are creating a simulated failure using the same crates. 

```rust
use lapin::{options::*, types::FieldTable, BasicProperties, Connection, ConnectionProperties};
use tokio::task;
use futures_util::stream::StreamExt;

const QUEUE_NAME: &str = "rust_queue";

#[tokio::main]
async fn main() {
    let amqp_addr = "amqp://localhost:5672/%2f";
    let connection = Connection::connect(amqp_addr, ConnectionProperties::default())
        .await
        .expect("Failed to connect to RabbitMQ");
    let channel = connection.create_channel().await.expect("Failed to create channel");

    // Declare queue
    channel
        .queue_declare(QUEUE_NAME, QueueDeclareOptions::default(), FieldTable::default())
        .await
        .expect("Queue declaration failed");

    // Spawn consumer task
    let consumer_channel = channel.clone();
    task::spawn(async move {
        let consumer = consumer_channel
            .basic_consume(QUEUE_NAME, "rust_consumer", BasicConsumeOptions::default(), FieldTable::default())
            .await
            .expect("Failed to start consumer");

        println!("Waiting for messages...");
        consumer
            .for_each(|delivery| async {
                if let Ok(delivery) = delivery {
                    let message = String::from_utf8_lossy(&delivery.data);
                    println!("Received message: {}", message);

                    let processing_result: Result<(), &'static str> = simulate_processing(&message);

                    match processing_result {
                        Ok(_) => {
                            println!("Processing succeeded. Acknowledging message.");
                            consumer_channel
                                .basic_ack(delivery.delivery_tag, BasicAckOptions::default())
                                .await
                                .expect("Failed to ack message");
                        }
                        Err(err) => {
                            println!("Processing failed: {}. Rejecting message.", err);
                            consumer_channel
                                .basic_nack(delivery.delivery_tag, false, true) // Requeue the message
                                .await
                                .expect("Failed to nack message");
                        }
                    }
                }
            })
            .await;
    });

    // Publish a message
    let payload = b"Hello from Rust!";
    channel
        .basic_publish(
            "",
            QUEUE_NAME,
            BasicPublishOptions::default(),
            payload.to_vec(),
            BasicProperties::default(),
        )
        .await
        .expect("Failed to publish message");

    println!("Message published!");

    tokio::time::sleep(std::time::Duration::from_secs(5)).await;
}

// Simulate message processing with a failure scenario
fn simulate_processing(message: &str) -> Result<(), &'static str> {
    if message.contains("fail") {
        Err("Simulated processing failure")
    } else {
        Ok(())
    }
}

```

Pretty basic situation here and things can definitely evolve to suit your needs but you can see where we use the `basic_ack` and our message would be removed from the queue. Otherwise we send `basic_nack(requeue=true)` and our message will go back to be retried. At some point if the message persists to fail we need to using something like a DeadLetterQueue to handle multiple failures. 

That's a bit more complex and would look something like this 
```rust
    const MAIN_QUEUE: &str = "rust_queue";
    const DLX_QUEUE: &str = "dead_letter_queue";
    const DLX_EXCHANGE: &str = "dlx_exchange";
    // Declare the Dead Letter Exchange (DLX)
    channel.exchange_declare(
        DLX_EXCHANGE,
        lapin::ExchangeKind::Direct,
        ExchangeDeclareOptions::default(),
        FieldTable::default(),
    ).await.expect("Failed to declare DLX exchange");

    // Declare the Dead Letter Queue
    channel.queue_declare(
        DLX_QUEUE,
        QueueDeclareOptions::default(),
        FieldTable::default(),
    ).await.expect("Failed to declare DLX queue");

    // Bind DLQ to DLX
    channel.queue_bind(
        DLX_QUEUE,
        DLX_EXCHANGE,
        DLX_QUEUE,
        QueueBindOptions::default(),
        FieldTable::default(),
    ).await.expect("Failed to bind DLQ");

    // Configure the main queue to send rejected messages to the DLX
    let mut args = FieldTable::default();
    args.insert("x-dead-letter-exchange".into(), DLX_EXCHANGE.into()); 
```

This way we can handle failures independantly and are not stuck in a retry with prior use of `basic_nack(requeue=true)`.

Next topic I will discuss is the use of the different types of exchanges. Here we are using an exchange with a single consumer. I will go over in more detail the different types we can set up and how they work and why next post. Thanks for reading!

