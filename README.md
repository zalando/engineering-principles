Zalando Tech's "Rules of Play"
========================================
This document focuses initially around services, because the principles for interoperating between services are quite mature and stable. 

**A note before we begin**: A service is an application—but not all applications are services. For example, a frontend is not a service. Its requirements are fundamentally harder to meet because of aesthetic and user experience concerns. And the fast-moving set of technologies around the browser bring less maturity and more complexity.

Starting from Scratch
------------------------------------------------------------
We strive to build applications that are:
- resilient
- extensible
- maintainable
- with quality built-in and 
- scalable to adjust to demand. 

These properties lead to architectural principles that guide the choices we have to make.

Architecture
------------------------------------------------------------
We prefer loosely coupled services. They are more resilient when it comes to remote dependency failures. We aim to develop autonomous isolated services that can be independently deployed and that are centered around defined business capabilities. 
###How to build a loosely-coupled system
####Asynchronous communication
Synchronous calls to remote systems can lead to threads in waiting state until the call times out. This can completely paralyze a system, as more and more threads move into that state until the system can no longer react to new requests. Further, synchronous calls are blocking and prevent the thread from doing anything else.

We want to reduce the impact of remote failures by calling them asynchronously. We achieve this by choosing an **event-driven architecture** based on queues (typically wrapped by REST interfaces). Communication with other systems becomes non-blocking. While functionality might be compromised, the system continues working.
####Service Degradation
We react to remote dependency failures by degrading a service until the remote dependency is working again. This means that your system must be aware of remote dependency health. It needs to detect failure, and it needs to notice when an external dependency becomes healthy again.
####Use Low-Tech Coupling
Low-tech coupling reduces issues resulting from changes in communicating systems, and can reduce complexity and dependencies. An example of low-tech coupling is service discovery via DNS. Communication should be done over interoperable protocols like HTTP instead of, for example, RMI.
####REST and JSON
Because of their weak type system, we prefer REST-based APIs with JSON payloads to SOAP and its strong type system. We prefer systems to be truly RESTful (including HATEOS), not just JSON RPC, because the goals of REST match ours: to build interoperating distributed systems that can be evolved in parallel by different teams while continuing to work.

REST makes it possible to evolve APIs safely and without breaking them, and it brings high-level simplicity across all our APIs.

[The API Guild](https://tech.zalando.com/blog/on-apis-and-the-zalando-api-guild/) provides structure around the details of our API strategy.
###How to structure your services
Build services around business entities with state and behavior—for examples, “orders,” “payments,” or “prices.” In REST terms, these are “resources.”  
####Service size
A service should be big enough to offer a valid business capability, but small enough to be handled by a team that can be fed by two pizzas (Amazon’s Two-Pizza Team rule)—i.e., from two to 12 people. In practice, a Two-Pizza Team may be able to own and run a large number of small services, or a smaller number of larger services. 

All things considered, we prefer smaller services written in expressive programming languages with minimal code whenever possible.
####Service Layers
A service typically includes several layers of the tech stack—entrypoint, business logic and data storage—and offers a clean API as an integration point. Teams have a lot of freedom to choose the technology they use to create a service, though we have internal resources that provide structure to technology choices.
####Autonomy
A service:
- should be as autonomous as possible.
- should run in its own process and be independently deployable.
- should start up and be resilient when its dependencies are not available.
- should not share its data storage or code repository with any other service, so that changes do not affect other systems.
should not share libraries with other services, unless those libraries are open-source. Shared internal dependencies lead to a large-scale complexity over time. We prefer to stop this practice immediately.
- should not provide a client library. The core API and its data model are expressed as REST and JSON.

####APIs
Our APIs form the purest expression of what our systems do. But API design is hard work and takes time. We prefer peer-reviewed, API First APIs designed and developed outside code (using Swagger, for example), to avoid the complexity and cost of making big changes. We prefer ongoing documentation to be generated from the code itself.

Our APIs need to last for a long time, so they must evolve in certain ways. Our APIs should all be similar in tone; we establish and agree to standards for how to do this. We will host API documentation for all our APIs in a central, searchable place. Documentation should always provide examples.

Our APIs should obey [Postel's Law—aka "the Robustness Principle"](https://en.wikipedia.org/wiki/Robustness_principle): Be conservative in what you send, be liberal in what you accept.

####Some Good Reads: 
- [Architectural Styles and the Design of Network-based Software Architectures](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)
- [InfoQ eMag: Web APIs: From Start to Finish](www.infoq.com/minibooks/emag-web-api)
- [Thoughts on RESTful API Design](https://restful-api-design.readthedocs.org/en/latest/)
- [Build APIs You Won't Hate](https://leanpub.com/build-apis-you-wont-hate)

####SaaS
Build your services so that it’s possible to offer them as a SaaS solution to third parties. In fact, consider any other system a third party with regards to API structure, resilience and service level. This is easier to do than it was a few years ago: AWS pushes us this way, the Internet model scales, and our security model is geared toward allowing our services to be on the open Internet. 

We want to offers services in ways we never imagined or expected. This is part of being a platform. In some cases, this means being multi-tenant from the start.

####Security
Always use SSL and make sure the caller of your service is authenticated and authorized.  

###General guidelines
####Stateless
When possible, be stateless. If you can’t, keep state separate from application logic. For example, use a separate database instead of, say, writing to a file.

####Immutable
Strive for immutability whenever possible. This is a key concept from Effective Java, and languages like Scala and Clojure have stronger support for this than Java. (See Does Scala == Effective Java?) 

Immutability tends to result in fewer bugs and makes it easier to prove a program correct. Immutable things are automatically thread-safe, with no synchronization required.

####Idempotent
