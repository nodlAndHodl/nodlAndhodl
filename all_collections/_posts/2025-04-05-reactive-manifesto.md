---
layout: post
title: Reading the Reactive Manifesto
date: 2025-04-05
categories: ["software architecture"]
---

### Reading the Reactive Manifesto
I had not yet read the ["Reactive Manifesto"](https://www.reactivemanifesto.org) until today. It was originally published in 2014 which in the world of technology is ancient, but it did spawn the current day movement towards highly scalable and highly available services using microservices. 

Basically it's a manifesto about how to create reactive systems and a need to create systems that are reactive. 

### Reactive Systems Are
By the way of the manifesto systems should have the following properties: 
 * Responsive
 * Resilient
 * Elastic
 * Message Driven

#### Responsive
In other words, the system should be consistent in response times and typically in my opinion under ~500ms and ideally ~200ms. Consistency in response times is also mentioned as well as effective error handling. 

#### Resilient
Systems should not be brought to their knees by failures. Rather the manifesto make mention of the ability to have replication, containment of problems(isolation) and decoupling, and delegation. This makes a lot of sense in the ideas of domain driven design and having separated services that make domain sense. 

#### Elastic 
The system should scale under workload and have the ability to add or remove resources. There are different algorithms that can be used to accomplish this but it must scale and be responsive to demand.

#### Message Driven
I thought this was interesting especially as it relates to the time it was written. Asynchronous message passing is to be non-blocking in processes and delegate failures as messages. In a microservices architecture this is crucial. And as always domain specific. 

### Design Is Critical
To get these principles right is hard and demands that design be taken seriously. 