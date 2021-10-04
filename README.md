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
