# Zalando's Engineering and Architecture Principles

#### The Principles, Briefly

As part of [Radical Agility](https://jobs.zalando.com/tech/blog/so-youve-heard-about-radical-agility...-video/), implemented by Zalando's technology team in March 2015, we have adopted this set of principles for tech and architecture:
- microservices
- API First
- REST
- Cloud
- Software as a Service (SaaS)

This document focuses primarily on services, as the principles for interoperating services are quite mature
and stable. Note: A (micro-) service is an application, but not all applications are services. For example, a frontend
is not a service. Its requirements are fundamentally harder to meet because of aesthetic and user experience concerns.
And the fast-moving set of technologies around the browser bring less maturity and more complexity.

## Starting from Scratch

We strive to build applications that are:
- resilient
- extensible
- maintainable
- with quality built-in and
- scalable to adjust to demand.

These properties lead to architectural principles that guide the choices we have to make.

## Architecture

We prefer loosely coupled services. They are more resilient when it comes to remote dependency failures. We aim to develop autonomous isolated services that can be independently deployed and that are centered around defined business capabilities.

### How to build a loosely-coupled system

#### Asynchronous Communication
Synchronous calls to remote systems can lead to threads in waiting state until the call times out. This can completely paralyze a system, as more and more threads move into that state until the system can no longer react to new requests. Further synchronous calls are blocking and prevent the thread from doing anything else.

We reduce the impact of remote failures by communicating asynchronously via events, where possible. Microservices publish streams of events to an [event broker](https://github.com/zalando/nakadi), which interested microservices consume asynchronously. Communication with other systems becomes non-blocking: even when some functionality is affected temporarily, the system continues working.

#### Service Degradation
In synchronous calls, we react to remote dependency failures by degrading a service until the remote dependency is working again. This means that your system must be aware of remote dependency health. It needs to detect failure and also notice when an external dependency becomes healthy again.

In asynchronous processing scenarios, you usually have more flexibility in processing service delivery time. Hence, you can live with temporary remote failures and delay processing with retries. However, service degradation is also an option here, if otherwise processing delays due to remote failures are not acceptable.

#### Use Low-Tech Coupling
Low-tech coupling reduces issues resulting from changes in communicating systems, and can reduce complexity and dependencies. An example of low-tech coupling is service discovery via DNS. Communication should be done over interoperable protocols like HTTP instead of, for example, RMI.

#### RESTful APIs with JSON payload
We prefer REST-based APIs with JSON payloads to SOAP. Distributed SOAs following the REST style have a looser coupling
between client and server implementations and comes with less rigid client/server contracts that do not break if either
side make certain changes. Hence it is easier to build interoperating distributed systems that can be evolved in parallel
by different teams while continuing to work. REST-like APIs with JSON payload is the most widely accepted and used service
interfacing style in the internet web service industry.

#### API First
The [API Guild](https://tech.zalando.com/blog/on-apis-and-the-zalando-api-guild/) provides structure around the
details of our API strategy. In April 2016, Guild members released this comprehensive model set
of [RESTful API guidelines](http://zalando.github.io/restful-api-guidelines/),
which define standards to successfully establish “consistent API look and feel” quality.

We also adopted ["API First"](https://zalando.github.io/restful-api-guidelines/general-guidelines/GeneralGuidelines.html)
and ["API as a Product"](https://zalando.github.io/restful-api-guidelines/design-principles/DesignPrinciples.html)
as key engineering principles. In a nutshell, API First encompasses a set of quality-related standards
(including the API guidelines and tooling) and fosters a peer review culture; it requires two aspects:

- define APIs outside the code first using a standard specification language (Open API 2.0)
- get early review feedback from peers and client developers (following a lightweight [API review procedure](https://pages.github.bus.zalan.do/ApiGuild/ApiReviewProcedure/))

#### Microservice Size
Build services based on domain model and around business entities with state and behavior —- for example “orders”, “payments” or “prices”. In REST terms, these are “resources”.

A service should be big enough to offer a valid business capability, but small enough to be handled by a team that can be fed by two pizzas (Amazon’s Two-Pizza Team rule) -- from two to 12 people. In practice, a Two-Pizza Team may be able to own and run a large number of small services, or a smaller number of larger services.

All things considered, we prefer smaller services written in expressive programming languages with minimal code whenever possible.

#### Technology Selection
A service typically includes several layers of the tech stack -— entrypoint, business logic and data storage -- and offers a clean API as an integration point. Teams have a lot of freedom to choose the best technologies for each layer. To balance innovation with economies of scale, we maintain the [Zalando Tech Radar](https://zalando.github.io/tech-radar/) as an internal tool for decision support and knowledge sharing.

#### Autonomy
A service:
- should be as autonomous as possible.
- should run in its own process and be independently deployable.
- should start up and be resilient when its dependencies are not available.
- should not share its data storage or code repository with any other service, so that changes do not affect other systems.
- should not share libraries with other services, unless those libraries are open-source. Shared internal dependencies lead to a large-scale complexity over time. We prefer to stop this practice immediately.
- should not provide a client library. The core API and its data model are expressed as REST and JSON.

#### APIs
Our APIs form the purest expression of what our systems do. But API design is hard work and takes time. We prefer peer-reviewed APIs which are designed in an "API First" way and developed outside code (using OpenAPI, for example), to avoid the complexity and cost of making big changes. We prefer ongoing documentation to be generated from the code itself.

Our APIs need to last for a long time, so they must evolve in certain ways. Our APIs should all be similar in tone; we establish and agree to standards for how to do this. We will host API documentation for all our APIs in a central, searchable place. Documentation should always provide examples.

Our APIs should obey [Postel's Law -— a.k.a. "the Robustness Principle"](https://en.wikipedia.org/wiki/Robustness_principle): Be conservative in what you send, be liberal in what you accept. APIs must be evolved without breaking any consumersF(.

#### Some Good Reads:
- [RESTful API Guidelines](https://zalando.github.io/restful-api-guidelines/TOC.html) by Zalando's API Guild
- [Architectural Styles and the Design of Network-based Software Architectures](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)
- [InfoQ eMag: Web APIs: From Start to Finish](https://www.infoq.com/minibooks/emag-web-api)
- [Thoughts on RESTful API Design](https://restful-api-design.readthedocs.org/en/latest/)
- [Build APIs You Won't Hate](https://leanpub.com/build-apis-you-wont-hate)

#### SaaS
Build your services so that it’s possible to offer them as a SaaS solution to third parties. In fact, consider any other system a third party with regards to API structure, resilience and service level. This is easier to do than it was a few years ago: AWS pushes us this way, the Internet model scales, and our security model is geared toward allowing our services to be on the open Internet.

We want to offers services in ways we never imagined or expected. This is part of being a platform. In some cases, this means being multi-tenant from the start.

#### Security
Always use SSL and make sure the caller of your service is authenticated and authorized. SSL actually means "HTTPS everywhere, not HTTP."

### General guidelines

#### Stateless
When possible, be stateless. If you can’t, persist state outside the address space of the application, for example in a database.

#### Immutable
Strive for immutability whenever possible. An object is immutable if its state cannot be modified. Immutable things are automatically thread-safe, without requiring synchronization. Overall, immutability tends to result in fewer bugs and makes it easier to prove a program correct.

#### Idempotent
Whenever possible and reasonable, make service endpoints [idempotent](https://en.wikipedia.org/wiki/Idempotence#Computer_science_meaning), so that an operation produces the same result even when it’s executed multiple times. This allows clients to safely retry operations in case of timeouts due to service processing or network failures.

### Development
Some general guidelines for how we think a development team should work.

#### Radical Agility
The core of our software development approach is [Radical Agility](https://jobs.zalando.com/tech/blog/radical-agility-study-notes/), which is based on three main pillars: Autonomy, Mastery, and Purpose; all held together and bound by organizational trust. The aim is to allow engineers to get work done, while management gets out of the way.

#### Agile > Process
We don't care which agile collaboration process (Scrum, Kanban etc.) you follow. Don’t focus too much on the process, focus on the outcome! Unfortunately, a defined process is required to satisfy our company audit requirements, but we like to keep it as minimal as possible.

#### Projects
We prefer that (almost) all work is done around some kind of conceptual “project.”

A project should have a clear purpose or goal. If it’s customer-facing, it should have some minimal business justification for why we are doing it. Assembling this information is typically the role of a product owner, but sometimes engineers need to do this themselves.

Having a first-class, cross-team notion of “project” is nice for a lot of reasons. It ultimately helps us to build automation that minimizes auditing and controlling overhead. It also helps us to report what we do for tax purposes -- and getting this right can save a lot of money.

#### No Micromanagement
If you feel like you’re being micromanaged, push back. We don’t do that here. On the other hand, it’s fine to ask for detailed support -- but it shouldn’t ever come as unwanted. The team — not the Delivery Lead — decides on who builds what and how it’s done.

#### Peer Review
Don’t wait until you’re done to ask for code review: It’s the best way to catch defects early. Create a pull request at the start of your work, not at the end. This pulls people into an ongoing conversation about your code, from Day One.

Code review is expensive in some ways, so get the most out of it. Reviewing code is a great way to learn about style, get help with idioms, and grow as a programmer and reviewer.

Code review can be hard when the culture around it isn’t supportive and constructive. It takes practice to learn how to accept code reviews without getting defensive, and to review code without focusing on trivial things. Don’t [bike shed](https://en.wikipedia.org/wiki/Law_of_triviality).

Peer review gets easier when you have a good attitude about it. Everybody around you is smart, and you are smart. We’re all smart in different ways.

Depending on the team and its codebases, it might be required that at least one person reviews code before it goes live. This is especially true for systems that touch customer or financial data. In general, though, we don’t want to focus about when code review is or isn’t required: The system works best when people decide on their own that code review is valuable, and seek it out.

Architectural decisions should be made as a team, and the team should ask for help if it’s unsure. Ask your Delivery Lead, People Lead, and/or Engineering Head, or even experts from other teams (if it makes sense). Embrace open discussions and alternate opinions.

#### Quality
Quality is related to mindset, and it’s part of engineering. Systems that support multi-billion-Euro companies must be engineered for high quality. Usually this means:
- writing unit tests early on
mocking external systems so you can test against them while they’re not running, and also so that you can simulate various - failure scenarios from the service and the network between it
- striving for automation

Automate testing whenever possible. It’s not always possible, but life is almost always better if you invest in automated tests of your code. (See Martin Fowler's [Testing Strategies in a Microservice Architecture](http://martinfowler.com/articles/microservice-testing/).)

We’re not going to require you to test your code, but expect your peers to challenge you if you don’t. For the most part, a dedicated QA team is a thing of the past. You and your team are responsible for your code’s behavior: There’s no other safety net.

Years ago, we didn’t build systems this way. Now we must. Fortunately, the tooling is pretty amazing.

#### Continuous Delivery
Strive for very short release cycles, optimally deploying daily; automating the delivery pipeline makes this possible. Small releases tend to have fewer bugs. Use canary testing for your new deployments to identify problems early.

Best practices for Continuous Delivery and Jenkins-as-a-Service are available for voluntary usage.

#### Source Code Management
We support Stash and GitHub as SCM to check in your code. You might want to use local git hooks for checking references to specifications in commit messages or checks.

#### Documentation
Document the architecture of your APIs and applications. Make it clear, concise, and current. Use inline documentation for more complex code fragments.

#### Open Source
Zalando's strategy has evolved from “Open Source All The Things!” to a focus on mastery, and a commitment to not just releasing code but building communities around open source. Open Source is a way of working and thinking that pushes forward not just great code but a great set of values and ways of collaborating that we believe are enormously engaging and powerful, and we strive to use them for the benefit of all. [Here](https://github.com/zalando/zalando-howto-open-source) is a detailed guide to open-sourcing projects at Zalando.

### Deployment

#### Cloud vs. On-Premise
AWS is our default choice for new projects, so that we can take full advantage of the flexibility and scalability of the cloud and its rich set of integrated services. We continue to run dedicated hardware (both on premise and in partner data centers) for some special or legacy use cases.

#### Docker
We favour containerised application development and our current deployment tool of choice is Docker. Both our internal Platform-as-a-Service (PaaS) offerings are based on it:

- For new services, we recommend to use our Continuous Deployment Platform (CDP) in combination with our hosted [Kubernetes](https://kubernetes.io/) service.

- Many existing services use [STUPS](http://stups.io/), which provides a convenient and audit-compliant way to manage and configure a dedicated AWS account per team.

#### Monitoring and Logging
We use both [Scalyr](https://www.scalyr.com/) and [ZMON](https://zmon.io/), our open-source inhouse monitoring solution, to track business KPIs and other metrics.

### Managing legacy
In transitioning to a microservices architecture, we must maintain and transform our legacy applications. Take following guidelines into account.

#### Reduce focus on sprocs
Sprocs are stored procedures. For us they have been a crucial component for scaling PostgreSQL horizontally. They are not, however, the right solution for every database problem.

Sprocs bring a lot of power, but also introduce a lot of complexity—particularly around testing, maintainability, refactoring, and transparency. Sometimes this is worth it, particularly when we have to shard. But this approach should be used only if circumstances genuinely warrant it.

#### New functionality becomes a new service
When introducing new functionality, think of designing it as a new service instead of adding it to an existing legacy application. This allows you to leverage new technologies.

#### Wrap it with Docker
Use Docker to package your application. STUPS.io will help you to deploy your Docker images to the existing infrastructure.

#### Migration to AWS
Move legacy code to AWS whenever possible. Sometimes it will seem hard, but others here have done it. Ask for help.

### Closing Words
#### The Joy of Programming
The authors love code. Building simple systems that work efficiently and quickly brings us joy. Seeing these systems interoperate cleanly and harmoniously gives us pleasure. We do this because we love it. If we didn’t have to work, we’d probably still do this. And we know we’re not alone.

Building software systems can produce substantial existential pleasure. When the conditions are just right, programming is a reliable path to [Flow](https://en.wikipedia.org/wiki/Flow_%28psychology%29): a state almost beyond pleasure. We want to get there, and stay there, and we want you to join us there. We hope these principles help.

### License

We have published these guidelines under the [Creative Commons Attribution 4.0 (CC-BY)](LICENSE.txt) license.
