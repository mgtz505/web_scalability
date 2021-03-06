# web_scalability
Notes from Web Scalability for Startup Engineers - Artur Ejsmont

# Chapter 1 Core Concepts
## What is Scalability?
<p> Scalability is an ability to adjust the capacity of the system to cost-efficiently fulfill the demands. Handles more users, clients, data, etc w/o affecting UX </p>

- Handling more data
- Handling higher concurrency levels (Concurrency measures how many clients your system can serve at the same time)
- Higher concurrencys means more open connections, more active threads, more messages being processed at the same time, and more CPU context switches
- Handling higher interaction rates (between system and client)
- Main challenge related to interaction rate is latency. As your interaction rate grows, you need to be able to serve responses quicker, requiring faster reads/writes and often drives requirements for higher concurrency levels. 

## Server Configurations

### Single-Server Configuration
<p>This is the simplest configuraiton possible and is how many small projects start. Assume the enyire application runs on a single machine.</p>

- In this scenario, users connect to the DNS to optain the IP address of the server where the website is hosted
- Once the IP address is obtained, they send HTTP requests directly to your web server
- A VPS, virtual private server, is a term used by hosting providers to describe a virtual machine for rent. When you purchase a VPS instance, it is hosted together w/ other VPS instances on a shared host machine. 
- VPS instances are cheap and can usually be upgraded instantly (add more RAM and/or CPU by paying more)

For sites with low traffic, a single-server configuration may be enough to handle the requests made by clients. That said, this single-server setup will not allow you to scale well:
- Userbase grows, increasing traffic - Each incremental user creates additional loas on the servers, and serving each user will consume more resources (memory, CPU time, I/O).
- Database grows, slowing down database querites due to extra CPU, memory and I/O requirements
- As functionality expands, your user interactions will require more system resources.

### Vertical Scaling
Vertical scalability is accomplished by upgrading the hardware and/or netowrk throughput. You simply upgrade your hardware!
- Addming more I/O capacity by adding more hard drives in Redundany Array of Independent Disks (RAID) array (I/O throughput and disk saturations are the main bottlenecks in database servers)
- Improve I/O access times by swithcing to solid-state drives (SSDs)
- Reduce I/O opertions by increasing RAM (memory size is especially important for efficiency of database servers)
- Improve network throughput by upgrading network interfaces or installing additional ones (i.e. if streaming lots of video / media)
- Switching to servers w/ more provessors or more vitual cores

Vertical scaling comes with serioous limitations, namely cost, becoming extremely expensive past a certain point. Secondly, there are hard limits to vertical scaling. It is not possible to be continually adding memory. Similar constraints apply to CPU speed, number of cores per server, and hard drive speed. LAstly, os design or the application itself may prevent vertical scaling beyond a threshold. For example, you cannot keep adding CPU to scale MySQL due to increasing lock contention. 

**Locks** are used to synchronize access between execution threads to shared resources like memory or files. 

**Lock Contention** is a performance bottleneck caused by inefficient lock management. 
Operations performed very often should have fine-grained locks; otherwise your application may sepdn most of its time waiting for locks to be released. Once you hit a lock contention bottleneck, adding more CPU cores does not increase the overall throughput. 

### Isolation of Services
At this stage in your startup, you're not limited to vertical scaling. 
**Isolation of services** is moving different parts of the system to seperate physical servers by installing each type of service on a separate physical machine
- This is a limited solution as once you deploy each service type on a separate machine you have no room to grow. 
- Isolation of services is a great next-step from a single-server setup as you can distribute the load among more machines than before and scale each of them vertically as needed. 
- This is a common configuration amond small websites and web dev agencies.
- The core concept behind isolation of services is that you should try to split your monolithic web application into a set of distinct funcitonal parts and host them independently. The process of dividing a system based on funcitonality to scale it independently is called **functional partitioning**

#### Other Tools
**Cache** is a server / service focused on reducing the latency and resources needed to generate the result by serving previously generated content; caching is a very important technique for scalability

**Content Delivery Network**  
As applications grow and you obtain more users, it becomes beneficial to offload some of the traffic to a 3rd party CDN service.
What's a CDN?
- A CDN is a hosted service that takes care of global distribution of static files like images, Javascript, CSS and videos
- It works as an HTTP proxy
- Clients that need to download images, JS, CSS or videos content connect to one of the servers owned by the CDN provider instead of your servers
- If the CDN server does not have a the requested content yet, it asks your server for it and caches it from then on
- Once the file is cached by the CDN, subsequent clients are servered w.o contacting your servers at all

By integrating your web application with a CDN provider, you significantly reduce the amount of bandwidth your servers need. You also need fewer web servers to serve static content. Moreover, resource locality may improve as CDN providers are usually global companies w/ data centers around the world. CDNs can speed up page load times as they'll serve static content from the closest data centers.

### Horizontal Scaling
Horizontal scaling is much harder to achieve and oftentimes must be considered before the application has been built. In rare cases it can be "added" ;ater by making modificaations to the application aritecture, but this often requires significnat effort. 

**Horizontal scaling** is accomplished by a number of methods to allow increased capacity by adding more servers. Horizontal scalability is considered the holy grail of scalability as it overcomes the increasing cost of capacity uinit associated w. vertical scaling. You'll never reach a limit to how many servers you can add. Moreover, horizontally scaling using 3rd party services is usually not only very cost-effective, but also transparent.

- The thing that distinguishes horizontally scalable systems form the previous evolution stages is that each server role in our data center can be scaled by adding more server
- True horizontal scalabiltiy is usually difficult and expensive
- Systems should start by horizontally scaling in areas where it is the easist to achieve (such as web servers and caches) and then tackle more difficult areas, such as databases or other persistance stores

**Round-Robin DNS** is a DNS server deature allowing you to resolve a single domain name to one of many IP addresses.
- The regular DNS server takes a domain name and resolves it a single IP address
- Thus, RR DNS allows you to map the domain name to multiple IP addresses, each IP address poining to a unique machine
- Then, each time a client asks for the name resolution, DNS responds w/ one of the different IP addresses
- Goal is to direct traffic from each client to one of the web servers
- Once a client recieves an IP address, it will oonly communicate w/ the selected server

**GeoDNS** is a DNS service that allows domain names to be resolved to IP addresses based on the customer's location.
- GeoDNS may serve different IP addresses based on the client location
- The goal is to minimize network latency 

**Edge Cache** is an HTTP cache server located near the customer, allowing the customer to partially cache the HTTP traffic
- Requests from the customer's browser go to the edge-cache server
- The server can then decide to serve the page from the cache, or it can decide to assemble the missing pieces of the page by sending background requests to your web servers
- It can also decide that the page is uncacheable and delegate fully to your web servers
- Edge cache servers can serve entire pages or cache fragments of HTTP responses

### Overview of a Data Center Infrastructure
Layer-by-layer

#### The Front Line
Front Line is the first part of our web stack. It's a set of components that users directly interact with. These components do not have any business logic and their main purpose is to increase the cacpacity and allow scalability.

**Load Balancer**
- A load blaancer is a software or hardware component that distributes traffinc coming to a single IP address to multiple servers, which are "hidden" behind the load balancer
- Load balancers are used to share the load evenly among multiple servers and to allow dynamic addition/removal of machines
- Since clients can only see the load balancer, web servers can be added at any time w/o service disruption

From the top, clients' requests go to the geoDNS server to resolve the domain names. DNS decides which data center is the closest to the cleint and responds w/ an IP address of a corresponding load balancer. It then gets distributed evenly over to front cache servers or directly over front-end web application servers. Front cache servers are optional; they can be deploted in remote locations outside of the data center or can be skipped entirely. In some cases it may be beneficial to have a layer of front-end cache servers to reduce the amount of load put on the rest of the infrastructure. 
- It is common to use third-party services such as LBs, CDN and reerse proxy servers
- In such cases, it may be common for this layer to be hosted entirely by third-party providers.

#### Web Application Layer
Second layer ofg the stack is the web application layer. It consits of web app servers responsible for generating the actual HTML of our web applicaiton servers and handling clients' HTTP requests.
- These machines often use a lightweight web framework w/ a minimum amount of business logic since the main duty of these servers is to render the UI
- The simpler and "dumber" the web app layer, the better
- By pushing most of your biz logic to web services, you allow more reuse and reduce the number of changes needed, since the presentation layer (web application) should be the one that undergoes the greatest number of changes
- Web application servers are usually easy to scale since they should be completely stateless; if developerd ina. stateless manner, adding more capacity is as simple as addming more servers to the load balancer pool

#### Web Services Layer
This is a critical layer as it contains most of our applicaiton logic. We keep front-end servers simple and free of biz logic since we want to decouple the presentaiton layer from the business logic. By creating web services, we also make it easier to create funcitonal partiions. We can create web services specializing in certain funcitonality and scale them independently
- The communication protocol used between front-end applications and web serives is usually REST or Simpel Object Acess Protocol (SOAP) over HTTP
- Depending on implementation, web services sould be relatively simple to scale
- As long as they remian stateless, scaling horizontally is as easy as adding more machines to the pool as it's the deeper data layers that are more difficult to scale

#### Additional Components
Since both front-end servers and web services should be stateless, web applications often deploy additional components
**Object Cache Servers** are used for both front-end application servers adn web services to reduce the load put on the data stores and speed up responses by storing partially precomputed results

**Message Queues** are used to postpone some of the processing to a later stage and to delegate work to queue worker machines
- Messages are often sent to message queus from both front-end applications and web service machines and are processed by dedicated queue worker machines

Sometimes web applications also have clusters of batch-processing servers or jobs running on schecdule. These machines are not involved in generating responses to users' requests; they are offline job-processing servers providing features likes asynchronous notigications, order fulfillment, and other high-latency functions

#### Data Persistance Layers
This is usually the most difficult layer to scale horizontally. The data layer has become increasingly more exciting in the past decade and the days of a single monolithic SQL databased are long gone. 

### Data Structure Infrastructure
Using the above structure, we now have the ability to share the load among multiple servers. Each of the above compontnes has a certain funciton and should help scale your application for 1mm+ users
- The layered structure of the components is deliberate and helps to reduce the load on the slower components; traffic coming to the load balancer is split equally over all the front-end cache servers. Since some requests are "cache hits", traffic is reduced and only part of it reaches front-end servers
- Here application-level cache and message queus help reduce traffic even-futher so that fewer requests reach back-end web services
- The web service can use message queus and cache servers as well
- Only if necesscary, the wb service later contacts search engines and the main data stores to read/write requisite info

It's crucual to remmember that not all of these components are needed to scale. There's a trafe-off between their functionality and complexity / maintenance costs

### Overview of Application Architecture
Let's look at the application itself. The app architecture should not revolve around a framework or any particular technology. Architecure should evolve around the business model. By putting that model in the center of our architecture, we make sure that other components surrounding it serve the biz and not vice-versa

**Domain Model** is created to represent the core functionality of the application in the words of biz people, not technical people
- The DM explains key terms, actors and operations w/o caring about technical implmenetation
-  the DM is a tool to create our mental pucture of the biz problem that our app is supposed to solve

#### The Front End
The FE should have the sole responsability of becoming the user interface; could be via web pages, mobile applications, or web srevice calls. The FE should not be considered a heart or the center of the system. In general, the FE should stay "as dumb as possible". By keeping the FE dumb, we can reuse more of the biz logic. Because the logic will only live in the web services layer, we avoid the risk of coupling it w/ our presentation layer. We will also be able to scale our FE servers independently as they will not need to perform complex processing, but may still be exposed to high concurrency challenges.
- FE applicaiotns will ahve to be developed in a way that they will allow communication over HTTP, incl AJAX and web sessions. 
- By hiding that within the FE layer, we can keep our services layer simpler and focused solely on the business logic, not on presentaiotn and web-specific technologies
- You can think of the FE application as a plugin that can be removed / rewritten in a different programming language and plugged back-in. You additionally should be able to remove the HTTP based FE and plug in a "mobile application" FE or evena. CLI FE.
- Projects that allow biz logic in the FE suffer from low code reuse and high complexity

#### Web Services
Web services are where most of the processing happens, and also where most of the biz logic should live

**Service-Oriented Architecture (SOA)** us architecture centered on loosely coupled and highly autonomous servuces focused on solving biz needs.
- In SOA it is preferred that all the servuces have clearly defined contracts and use the same communication protocols

**Multilayer Architecure** is a way to divide functionality into a set of layers. 
- Components in the lower layers epxose an API that can be consumerd by clients resigin the layers above, but you can never allow lower layers to depend on the functionality provided by upper layers. 
- An important side effect of layered architecture is increased stability as you go deeper into the layers
- You can change the API of upper layers easily since few things depend on them
- Conversely, yo chancing the API for lower layers may be expensive becuase there may be a lot of code depending on the exisiting API

**Hexagonal Architecture*** assumes that the biz logic is in the center of the architecture and all the interactions witht he data stores, clients and other system are equal
- There is a ocntract between the biz logic and every non-biz compoennt, but there is no distinciton between the layers above and below
- In hexagonal architecture, users interacting w/ the application are no differnt form the database system that the app interacts with.
- They both reside outside the the app biz logic and both deserve a strict contract
- By defining these boundaries, you can then replace the person w/ an automated test driver or replace the database witha. differnt storage engine w/o affecting the core of the sytem

**Event-driven Architecture**
- Simply put, EDA is a different way of thinking about actions
- Event-driven architecture, as the name implies, is about reacting to events that have already happened
- As name implies, reacting to events that have already happened
- Traditional architecture is about responding to requests and requesting work to be done; in traditional programming model, we think og ourselves as a person asking for something to be done, for example calling a funciton. We expect this operation to be performed while we're waiting for a result, and when we get the result, we continue our processing
- In EDA we don't wait for things to be done; Whenever we have to interact w/ other components, we announce things that have already happened and proceed w/ our own processing

No matter the syle of architecture of the system, all architectures will provide a benefit grom being divided into smaller independent functional units. The prupose is to build higher abstractions that hide complexity, limit dependencies and allow you to scale each part indepenedently, and make parallel development of each part practical

#### Supporting Technologies
- Inclusive og message queue, application cache, main data store and search engine
- These are isolated since that are usually implmented in different technologies and most often as 3rd party software products confiured to work with our system
- Given that these are 3rd party offerings, they can be treated as black-boxes int eh context of architecture
- Databases are traded as an implementation detail; from the app architecture POV, the data store is something that lets us qrite and read data. We don't care how many servers it needs, how it handles scalability, replication or even how it perssits data
- Think of the sta store as how you think of caches, search engines and message queues, as plug-and-play extensions
- If you switch to a different persistence store to to exchange your caching back ends, you should be able to do it by replacing the connectivity components, leaving the overall architecure intact
- By abstracting the data store, you free your mind from using a given database engine
- If the app logic has differnt requrements, consider a NoSQL data store or in-memory solution
- The data store is not the central piece of the architecture, and it should not dictate the way your system evolves

# Chapter Two - Principles of Good Software Design

The most important principle is keeping things simple! Simplicity isn't about using shortcuts; it's about making what would be easiest for other engineers to use in the future. It's also about being able to comprehend the system as it grows increasingly large and complex. 

## Hiding Complexity and Building Abstraction
As your system grows, you will not be able to build a mental picture of the entire system as it'll have too many details. Strive for local simplicity! 
**Local Simplicity** is achieved by ensuring that you can look at any single class, module, or application and quickly understand what its purpose is and how it works. You should only have to comprehend the class at hand to fully understand its behavior. 
- A good general rule is that no class should dpeend on more than a few other interfaces or classes

## Avoid Over-Engineering 
Overengineering is building a solution that is much more complex than is really needed. 
- Ask yourself: Can this be any simpler and still allow flexibility in the future?
- Ask yourself: What tradeoffs am I making here or Will I really need this later?
- Engineering is often a game of tradeoffs!

## Try Test-Driven Development
**TTD** is a set of practices where engineers write tests first and then implement the actual functionality. 
- The main beenfit is that there is no code w/o unit tests, i.e. no spare code / no unused functionality 
- Tests can additionally serve as a type of documentation as they can show you how the code was meant to be used anad what the expected behavior was.
- Froma. cleint persepctive, you must have a much more spartan approach; think about how a user would actually engage with the application, then wirte your tests accordingly
- When wiritng code, whehther using TTD or not, think about it from the perspective of a client. 

## Loose-Coupling
**Coupling** is a measure of how much two components know about and depend on one another. The higher the coupling, the stronger the dependency. Loose coupling refers to a situation where different components know as little as necessary about each other, whereas no coupling between components means that they are completely unaware of each other's existence.

- Keeping coupling low in your system is important for the health of the system and ability to scale. 
- High coupling means that changing a single piece of code requires you to inspect in detail multiple parts of the system; higher the coupling, the more unexpected the dependencies and higher chance of introducing bugs into your code.
- Low coupling promotes keeping complexity localized. Multiple engineers can work on the code base simultaneously. 
- Promote losse compiling by carefully managing dependecnies amoung classes, modules and applications.

## Practice DRY Code
- Be cautious with copy + paste code; be cautious with duplication of code.

## Single Responsability Principle
- Classes should have a single responsability and nothering more
- Following this principle will result in producing small and simple classes that can be easily refactored and reused. 

To promote the single responsability principle:
- Keep class length below two to four screens of code
- Ensure that a class depends on no more than five other interfaces / classes
- Ensure that a class has a specific goal/purpose
- Summarize the responsability of the class in a single sentence and put it in a common on top of the class name.

## Open-Closed Principle
- OC principle is about creating code that does not have to be modified when requirements chance or when new use cases arise. 
- Open for extenstion and closed for modification. 
- This option lets us leave more options available and delay decisions about the details; it also reduces the need to change existing code

## Dependency Injection
- Dependency injection is a simple technique that reduces coupling and promotes the open-closed principle.
- DI provides references to objects that the class depends on, instead of allowing the class to gather the dependencies itself. 
- At it's core DI is about knowing as little as possible; it allows classes to not-know how their dependencies are assembled, where they come from or what actual implementations are fulfilling their contracts. 
- In practice, DI can be summarized as not using the "new" keyword in your classes and demamnding instances of your dependencies to be provided to your class by its clients

## Inversion of Control (IOC)
- Inversion of control is a method of removing responsabilities from a class to make it simpler and less coupled from the rest of the system. 
- IOC used heavily by some frameworks (Sprint, Rails, etc); instead of you being in control of creating instances of your objects and invoking methods, you become the creator of plugins or extensions to the framework
- IOC is referred to as "Hollywood principle": subject of IOC is being told "don't call us, we'll call you"
- IOC is a universal concept. You can create an inversion of control framework for any type of application, and it does not have to be related to MVC or web requests. 

Components of a good IOC framework include the following: 
- You can create plugins for your framework
- Each plugin is independent and can be added or removed at any point in time
- Your framework can auto-detect these plugins or there's a way of configuring whcih plugin should be used and how
- Your framework defines the interface for each plugin type and is not coupled to plugins themselves

## Desigining for Scale

To make sure you do not overengineer by preparing for scale that you will never need, you should first carefully estimate the most realistic sclability needs of your system and then design accordingly. Only a limited number of startups reach the size where horizontal scalability is needed. 

As you learn more about scalability, you may realize that many of the scalability solutions can be boiled-down to three basic deisgn techniques:
- Adding more clones (adding indistinguishable components)
- Functional Partitioning (Dividing the system into smaller subsystems based on functionality)
- Data partitioning (Keeping a subset of the data on each machine)

### Adding more Clones
This is the easiest and most common scaling technique. Build the system in a wat that allows you to scale by adding clones.
A **clone** here is an exact copy of a component or server. Anytime you look at two clones, they must be interchangeable and each of them should be equally qualified to serve an incoming request. In other words, you should be able to send a request to a random clone and get a correct result. 

To scale by adding clones, your goal is to have a set of perfectly interchangeable web servers and distribute the load equally among them all. For this set-up. the load (web requests) is usually distriubted among the clones using a load balancer. Ideally whenever this load balancer recieves a request, it should be able to send it to any of ther serveres w/o needing to know where the previous request went. 

When scaling via clone addition, you need to pay close attention to where you keep the application state and how you propegate state changes among your clones. Scacling via clone addition works best for stateless services as there's no state to synchronize. 
- If your web servers are stateless, then a new server is exactly the same as a server that is already serving requests. 
  - In this case, you can increase capacity by simply adding more servers to the load balancer pool
 
 **Stateless Service** is a term used to indicate that a service does not depend on the local state, so processing requests does not addect the way the service behaves. 
 The main challenge with scaling by adding clones is that it is difficult to scale stateful servers this way, as you need to find ways yo synchronize their state to make them interchangeable.
 
 ### Functional Partitioning 
 The main thought behind FP us to look for parts of the system focused on a specific functionality and create independent subsystems out of them. In a more advanced form, functional partitioning is dividing a system into self-sufficient applications. It is applied most-often in the web services layer and it is one of the key practives fo service-oriented architecture (SOA). 
 
 Functional partitioning is most often applied on a low level where you break your application down into modules and deploy different types of software to differnt servers (i.e. databases on different servers than web services). In a larger company, it is common to use FP on a higher level of abstraction by creating independent services. In such cases, you can split monolithic application into a set of smaller funcitonal services. 
 
 An additional benefit of FP is that you can have multiple teams working in parallel on independent codebases and gaining more flexibility in scaling each services as differing services have different scalability needs. 
 
 Drawbacks of FP:
- FPs are independent and usually require more management and effort to start with. 
- - There is also a limited number of FPs that you can implment, eventually limiting your ability to use this technique

### Data Partitioning
DP is partitioning the data to keep subsets of it on each machine instead of cloning the entire data set onto each machine. 
- This is a manifestation fo the share-nothing principle as each server has its own subset of data, which it can control independently.
- Share-Nothing is an architectural principle where each node is fully autonomous. As a result, each node can make its own decisions about its state w/o need to propegate state changes to peers.
- Not sharing state means that there is no data synchronization, no need for locking and that failures can be isolated bc nodes do not depend on one another.

Simple Data Partitioning:
- Each server gets a subset of the data for which it is solely responsible (by having less data on each server, I can process it faster and store more of it in memory)
- If I need to increase capacity even further, I could add more servers and modify the mapping to redistribute the data

Data partioning, if applied correctly in tandem with adding clones, effecitvely allows for endless scalability. If you partition the data correctly, you can alwayhs add more users, handle more parallel connections, collect more data and deploy your system onto more servers. That said, data partitioning is alwso the most complex and expensive technique. The biggest challenge is that you need to be able to locate the partiion on which the data loves before sending queries to the servers adn that queries spanning multiple partitions may become very inefficient and difficult to implement. 

## Design for Self-Healing
Designing software for high-availability and self-healing.
A system is considered to be available as long as it performs its functions as exected from the client's perspective. It does nto matter if the system is experiencing internal partial failure as long as it does not affect the bahavior that clients depend on. In other words, you want to ensure your system appears as if all of its components were functioning perfectly even when things break and during maintenance periods.

A **highly-available** system is a system that is epxected to be available to its clients most of the time. There's no exact measure of high-availability (depends on biz needs). Systems are commonly measured in "number of nines":

A system with two nines is available ~99% of the time, roughly 3.5 days of outage every year (365 * 0.01). A system with five nines would be available 99.999% of the time, i.e. unavailable for five minutes per year.

The main point to remember, is that the larger your system gets, the higher the chance of failure. If you need to contact five webservices and each connects to three data stroes, you are depending on 15 components. If a single one fails, you might become unavailable unless you can handle failure gracefully or fail-over transparently. when designing for high availability, you need to hope for the best but prepare for the worst, always considering what else can fail and in what order. 

Crash-Only Concept
- CC approach says taht a system should always be ready to crash, and whenever it reboots, it should be able to continue to work w/o human interaction. 
- This means that the system needs to be able to detect the faulure, fix the broken data if needed-be, and then start work as normal. 
- Continuously testing differnt failure scnearios is a great way to improve the resilience of your system and promote high availability
- Inpractice, ensuring high availability is mainly about removing single points of failure and graceful failover.

** Single Point of Failure ** is any piece of infrastructure that is needed for the system to work properly. An example of a single point of failure is can be a DNS server if you only have one. 

A simple way to identify single points of fialure os to draw your data center diagram w/ every single device (routers, servers, switches) and ask yourself what would happen if you shut them down one at a time. Addresss with redundancy

Redundancy is having more than one copy of each piece or each compoennt of the infrastructure. Systems that are not redundant need special attention and it is abest practice to prepare a disaster recovery plan with recovery proceudres for all critical pieces of infrastructure. 

**Self-Healing** is a property going beyond graceful failure handling: it's the ability to detect and fiz problems automatically w/o human intervnention.
- Holy grail of web operations, but very difficult and expensive to build

# Chapter 3: Building the Front End Layer

The front end layer spans multiple components. It includes the client (usually a web browser), network components between the client and your data center and parts of your data center that respond directly to clients' connections). 
The FE componentns will recieve the most traffic; all user interaction, connections and responses must go through the front-end layer in some form or another. This in turn causes the front-end layer to have the highest throughput and concurrency rate demands, making its scalability crucial.

Mist of today's websites are built as tranditional multipage web applications, SPAs or hybrids of these tow approaches,

## Traditional Multipage Web Apps
- Clicking a link and/or button initiates a new web request and results in the browser reloading an entire page w/ the response recieved from the server. Very simple approach.
## SPAs
- These execute the most biz logic in the browser, moreso than hybrid or tranditional applications
- Primarily built w/ JS with web servers often reduced to providing a data API and a security layer
- In this model, anytome you performa n action in the UI (say clicking a link or typing text) JS code may initiate asynchronous calls to the server to load/save data
- The main benefit of SPAs is a richer UI, but users also benefit from a smalelr network footprint and lower latencies between user interactions
## Hybird Applications
- Built on blend of SPAs and traditional multipage applciations

## Managing State
- Carefully managing state is the most important aspect of scaling the front end of your web application
**Statelessness** is a property of a service, server, or object indicating that it does not hold any data (state)
- Statelessness makes instances of the same type interchangeable, allowing better scalability
- In contrast, stateful services keep some "knowledge" between requests, which is not available to every instance of the service

### Managing HTTP Sessions
Since the HTTP protocol is stateless itself, web applications developed techniques to create a concept of a session on top of HTTP so that servers could recognize multiple requests from the same user as parts of a more complex and longer lasting sequence (the user session)

From a technical POV, sessions are implemented using cookies. 
1. When a user sends a request to the web server w/o a session cookie, the server can decided to start a new session by sending a response w/ a new session cookie header
2. The HTTP contract says that all cookies that are still active need to be included in all consecutive calls
3. By using cookies the server can now recognize which requests are part of the same sequence of events

Even if multiple browsers connected to the web server from the same IP address, cookies allow the web server to figure out whoch requests belong to a particular user, This in turn allows for implementation of user login functionality and other similar features,

When you login to a website, a web application would usually store your user identifier and additional data in the web sesison scope. The web framewoek or the application container toulf then be responsbile for storing the web session scope "somewhere" so that the data stores in the web session scope would be available to the web applicaiton on each HTTP request. 

The key point to overse is that any data you put into the session should be stored outside of the web server itself to be available from any web server.  There are three common wats to solve this problem:
- Store session state in cookies
- Delegate the session storage to an external data store
- Use a load balancer that support sticky sessions

If you decide to store session, situation is fairly simple. In the application, use session scope as normal, then before sending a response to the client, the framework serializes the session data, encrypts it, and includes it in the response headers as a new value of the session data cookie. Main advantage of this approach is that session state is not stored anywhere in your data center.

The practical challenge with using cookies for session storage is that session storage becomes expensive. Cookies are sent by the browser w/ every single request, regardless of the typoe of resource being requested

