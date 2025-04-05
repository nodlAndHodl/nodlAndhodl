---
layout: post
title: Microservices and Shared Dependencies
date: 2025-04-04
categories: ["software architecture"]
---

### How to Best Share Dependencies In a Microservices Architecture
From my experience with microservices there are 3 main ways that dependencies can be shared between microservices. With this in mind we want to keep things as domain specific as possible, however there are situations where we want to reduce code duplication in something like a logger implementation or better yet authentication/authorization handling or middlewares that are used throughout each microservice.

### Monorepo
Monorepo's have definitely gained in popularity in the last few years and one startup I was at this was the norm. The backend was a scaled microservices architecture with an API gateway as the proxy. This worked well however the one issue we encountered on a couple occasions is deployments and not wanting to deploy everything and having some issues with deployment versions. Mind you this was a learning experience for both myself and the devops engineer at the time and I am sure it could have been better. What a monorepo allowed for though was to have some common share modules for things such as authentication/authorization. This in my opinion is the most convenient for sharing of these modules. 

### Git Submodules
This was another common practice and was convenient to some extent but started to pose issues when many changes were occurring. This was used in a situation where we had an ORM in use between several nodejs services/microservices but were not in a monorepo setup. This created situations where schema changes would occur and locally things would start to break without updating the branch on the submodules feature branch. If walking into the project it was not always clear as to what the issue was and which branch submodules should be pointed at. In my opinion this was the least favorable but could work with a smaller team.

### Nuget Packages or NPM Packages
This is another common practice and I think is perfectly fine and probably the most used from my experience when a team is not using a monorepo. The prior git submodules mention is similar but in my opinion created more issues than it solved and the use of standardized packages worked well. This means we can version our dependencies, and honestly as long as something like logging or authentication isn't changing much is probably the ideal if not using a monorepo. 

### My Preference Overall
Small teams can and usually should use a mono repo. It's adaptable, and shares modules easily. Many organizations that are large in scale will likely have their own packages to share across the organization and will likely come in the form of nuget packages or npm packages. And even a combination is preferable. 