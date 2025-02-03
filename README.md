# Principles of System Design Usage  

<div style="text-align: right; margin: 5 auto;">
    <a href='./LEIAME.md'>🇧🇷 Versão em Português</a> | <a href='./README.md'>🇺🇸 English Version</a>
</div>

System Design (SD) is the foundation for creating **scalable, robust, and distributed systems**. A well-structured design helps avoid bottlenecks, optimize resource usage, and prepare the system to support a large volume of users and data. Additionally, it facilitates collaboration between development teams, reduces operational costs, and avoids rework by anticipating potential technical challenges. Investing time in architecture planning before coding is essential to create efficient, resilient, and growth-ready systems.  


One of the main mistakes when dealing with performance and scalability issues is believing that simply **adding or swapping libraries in the frontend or backend** will solve them. Often, the difficulties faced are related to the **architecture and overall behavior of the system**, rather than the specific technology used in development. In the context of SD, the **programming language or framework** is not the main focus; what really matters is ensuring **predictable, modular, and resilient behavior**, where system components can communicate efficiently and support growth without compromising the user experience. 

![Image title](/images/system-design.jpg)

To help understand the impact, we’ve built this guide with the main topics, along with a case study to apply the concepts in practice. We’ll divide the study into:  

- The [`Components`](#main-components) of SD, covering the more theoretical part.  
- The [`Fundamental Principles`](#fundamental-principles-of-system-design), which underpin the best practices of System Design.  
- The [`Main Dilemmas`](#main-dilemmas) faced when building a solution.  
- A dedicated section to understanding how [`DevOps can assist`](#how-devops-can-help-in-system-design), helping software engineers solve problems effectively.  


In this guide, we’ll use the [`Twitter Case Study`](#case-study) to illustrate the concepts presented. By the end, you’ll have a clear understanding of the [`Case Study Outcome`](#case-study-outcome), analyzing how Twitter was built and how the main SD concepts were applied to avoid scalability, availability, and consistency issues.  

During each SD topic, we’ll make some notes following the pattern below:  


- 📖 **Use Case**: practical application of the component/concept/dilema in the development of the Twitter system.  
- 🙋🏽‍♂️ **Answer**: questions for you to reflect on and answer, helping to solidify knowledge or encouraging further research.  
- 🪧 **Tip**: suggestions to aid in understanding the concept or avoiding common pitfalls.  
- 🚨 **Attention**: alerts to avoid critical errors or major issues.  

Before we proceed to the case study, it’s important to emphasize that, in the context of System Design, all decisions involve a **trade-off**. You may find different solutions to the same problem, because, in the end... *"There is no silver bullet!"* 

---

## **Case Study**

To facilitate understanding of the concepts, we’ll use the case of X (yes, the former Twitter), a classic example of a distributed system that handles millions of users and tweets daily. It’s a complex system that faces various challenges, such as scalability, availability, consistency, and fault tolerance. Additionally, it needs to handle the geographical distribution of content, including images, videos, and text.  

### **1. Requirements**

To start, some questions need to be answered to understand the system’s purpose, scope, and expected behavior. In software development and management, there are several ways to identify the necessary requirements for delivering the expected value of the system. Here, we’ll adopt the classic division into two categories of requirements:  

- **Functional Requirements**: Define *what* the system should do, i.e., which problems and needs should be addressed and resolved by the software through functions or services.  

- **Non-Functional Requirements**: Define *how* the system should behave. While functional requirements focus on *what will be done*, non-functional requirements describe *how it will be done*.

#### 🙋🏽‍♂️ **Answer**

1. What functional and non-functional requirements would you add to the Twitter system?

2. Should the system be able to handle geographical and language distribution issues?


#### 📖  "Case Study"
    
For our tweet case study, we’ll consider the following functional and non-functional requirements:

**Functional Requirements**

- Post a tweet
- Follow a user
- View the tweet feed
- Retweet a tweet
- Search for tweets
- Receive notifications

**Non-Functional Requirements**

- **Scalability**: The system must handle a large number of users and tweets.
- **Availability**: The system must be available 24/7.
- **Consistency**: The system must ensure data is up-to-date, but not necessarily in real-time.
- **Fault Tolerance**: The system must handle hardware and software failures.

## **Main Components**

Now that we understand the system requirements, let’s address the main concepts of System Design, essential for creating scalable and efficient systems.  

### **1. Scaling** 

![Image title](/images/system-design-horizontal-vs-vertical.png)

Scaling is the ability of a system to handle an increase in workload. For example, think about how Twitter processes millions of requests, handling regional and temporal peaks, as well as the possibility of a sudden increase in new users. There are two main approaches to scaling a system to meet these demands:  

- **Horizontal Scaling**: Adding more servers to divide the load, i.e., replicas.  
    * Allocate servers in the infrastructure to distribute the load.  
    * Easier to scale and generally more cost-effective.  
    * Requires a distributed system and can be complex when adding [load balancers](#3-load-balancing) and [caches](#2-caching).  

- **Vertical Scaling**: Increasing the resources (CPU, memory) of a single server.  
    * Add more CPU, memory, or storage to a single server.  
    * Easier to implement but has limitations.  
    * Can be more expensive and less flexible.  

In many cases, the choice of scaling model depends not only on the system requirements but also on the financial resources available for the project. Initial projects, such as POCs (*Proof of Concepts*), tend to use horizontal scaling due to its cost-effectiveness.  

#### 📖 "Answer"

1. How does your company handle workload increases in a system? 

2. What is the scaling structure of your current system?

#### 🪧 "Tip"

Some infrastructure solutions allow for more efficient use of horizontal scaling, such as [Docker Swarm](#3-load-balancing) or a hybrid approach with [Kubernetes](https://www.geeksforgeeks.org/kubernetes-horizontal-vs-vertical-scaling/).  

Some infrastructure solutions allow you to work more efficiently with horizontal scaling, such as using [Docker Swarm](#3-load-balancing) or a mixed approach like [Kubernetes](https://www.geeksforgeeks.org/kubernetes-horizontal-vs-vertical-scaling/).
    
#### 📖 **Use Case**

In the case of Twitter, we use the concepts of vertical and horizontal scaling to scale the system. The database and application servers are scaled differently:  

- The **database** is scaled **vertically** to increase storage capacity and perform [replication](#411-replication).  
- The **application servers** are scaled **horizontally** to handle user traffic.  

**Note**: We’ll discuss the database and APIs in more detail in the following topics.

----

### **2. Caching**  

![Image title](/images/system-design-cache.jpg)

Have you ever encountered requests being executed in your backend multiple times to fetch the same information within a short period? This type of request is a strong candidate for using a caching system, allowing for faster delivery and reducing the effort of your API, backend, and database. However, the choice of caching strategy depends on various factors intrinsically linked to your application, such as the size and volatility of the data, access frequency, storage capacity, and performance goals. Well-planned implementations can significantly improve scalability, reduce resource consumption, and provide a better user experience.  

Caching is appropriate for:  

- [x] Temporarily storing data to improve performance.  
- [x] Reducing latency between client and server. 

There are various solutions in the market, such as `Redis` and `Memcached`, which can be used. However, more important than choosing the right tool is **understanding** the cache eviction policy and the write mode that will be adopted in your application.  

#### 2.1 LRU/LFU Policies

In an ideal world, we could cache all data and maintain the lowest possible latency in the application and database. However, there are always resource limitations, and we need to make choices. When we talk about **LRU/LFU**, we’re referring to the strategies to decide which data to keep in cache when the available space runs out:  

- **Least Recently Used (LRU)**: Removes the oldest keys to make room for new keys and data.
- **Least Frequently Used (LFU)**: Removes the least recently used keys to make room for new data.


#### 2.2 Cache Write-through/Write-behind

Now that we understand how data remains in cache, we need to discuss how it is persisted and the write strategy. Think about the scenario of inserting tweets: it’s certainly more important for a tweet to be saved in the database than in the cache, while reading a user’s feed can tolerate the absence of some tweets temporarily.  .

- **Write-through**: Data is written to the cache and the original source simultaneously.
- **Write-behind**: Data is written to the cache first and then to the original source.
- **Refresh-ahead**: Data is updated in the cache before it expires.

The choice of write policy depends on your system and application. However, it’s important to keep in mind that **Write-through** is safer but can be slower, while **Write-behind** is faster but less secure. As you’ve already noticed, the main *trade-off* here is the **importance of data consistency in writing**, i.e., whether consistency is critical for the application.  


###### 🚨 **Attention**

When caching and write policies are not implemented correctly, your application may suffer from high latency and data inconsistency! Be careful when defining these strategies.  

##### 📖 **Use Case**

In the case of Twitter, we use caching to store recent tweets, user feeds, and trends, improving performance and reducing latency. The adopted strategy is as follows:  

- For the [`Timeline Service`](#33-timeline-service), responsible for creating the timeline within the [`Read API`](#22-read-api), we use the **LFU** policy, prioritizing in cache the tweets most frequently accessed by users.  

- For generating a specific user’s feed via [`Timeline Service`](#33-timeline-service), we opt for the **Write-behind** policy, as consistency (if a tweet doesn’t appear immediately) is not as critical.  

- For creating tweets, through the [`Fan Out Service`](#34-fan-out-service), within the [`Write API`](#23-write-api), we adopt the **Write-through** policy, ensuring tweet consistency in the database, even if it results in a longer response time.  

- For derived services, such as the [`User Graph Service`](#35-user-graph-service) and the [`Search Service`](#36-search-api), we use the **Write-behind** policy, as data consistency in these services is not as critical.  
    
----

### **3. Load Balancing**  

![Image title](/images/system-design-loader-balance.jpg)

Load balancers distribute client requests to computing resources, such as application servers, APIs, and databases. In each case, the load balancer returns the response from the computing resource to the requesting client. 

Load balancers are effective in:

- [x] Helping to eliminate a [single point of failure](#1-failovers-and-spof).  
- [x] Preventing requests from being sent to unhealthy servers.  
- [x] Avoiding overloading resources like databases and static content. 

The distribution of requests among multiple servers to ensure high availability and performance is applied in horizontally distributed systems. However, this approach introduces some complexities, such as the need for server cloning. Keep in mind that:  

- [`Stateful`](#3-stateful-vs-stateless) servers are harder to scale, as they require additional costs for address management and state synchronization.  
- Sessions should be stored in a database and/or a persistent cache to ensure data consistency.  

#### **3.1 Load Balancing Algorithms**

![Image title](/images/system-design-loader-balance-algoritmos.gif)

Before talking about load balancing algorithms, have you thought about how complex it can be to distribute the load among servers? This problem is more difficult than it seems. Imagine a scenario where you have two servers, a third is added, and the first one stops responding. How do you distribute the requests?  

To better understand this problem, I recommend first reading the concept of [Consistent Hashing](#2-consistent-hashing) in section 6, as it helps maintain consistency in load distribution.  

Now, back to the distribution question, there are various load balancing algorithms, each with its advantages and disadvantages. Some of the most common are:  

- **Random**: Distributes requests randomly among healthy servers, but does not guarantee uniform distribution;
- **Least loaded**: Distributes requests to the server with the least load, but can lead to overloading faster servers;
- **Session/cookies-based**: Distributes requests based on sessions or cookies, but can lead to scalability and consistency issues.
- **Round robin and weighted round robin**: Distributes requests in a uniform or non-uniform circular order (when weighted);
- **Least connections**: Distributes requests to the server with the least number of connections, but can lead to overloading faster servers.

Notice that the choice of algorithm directly influences load distribution. There are various market solutions to implement *load balancing*, such as `Nginx`, `HAProxy`, `AWS ELB`, `Google Cloud Load Balancer`, `Azure Load Balancer`, among others.  
    
The Load Balancer distributes the workload among multiple servers. Let’s compare it with some popular solutions:  

**Load Balancer vs. Docker Swarm**  
*Load balancing* is a technique that distributes the workload among servers, while Docker Swarm is a container orchestration solution that allows creating and managing container clusters.  

**Load Balancer vs. Kubernetes**  
Kubernetes is an open-source container orchestration platform that automates the deployment, scaling, and operation of containerized applications.


##### 🙋🏽‍♂️ **Answer**  

How would you solve the *load balancing* problem with the configuration below?  

- Currently, there are 4 healthy servers operating: [`s1`, `s2`, `s3`, `s4`]
- User `u1` was the last to make a request and was previously directed to server `s3`.  
- In the next second (`t1`), `u1` will make a new request.  
- A new user, `u2`, who has not yet been distributed, will make their first request in the following second (`t2`).  
- In the third second (`t3`), `u1` will make a second request.  


Based on this scenario, answer the following questions considering different *load balancing* algorithms:  


1. To which server will user `u1` be directed in each *load balancing* algorithm?  

2. To which server will `u2` be allocated in each algorithm?  

3. If server `s3` becomes unavailable, to which server will the `t3` request from user `u1` be allocated?  

4. If a new server `s5` is added between seconds `t2` and `t3`, how would this affect the allocation of the `t3` request from user `u1` in each algorithm?

    
#### 📖 **Use Case**
    
User requests (*Client*) will be mediated by the [`Load Balancer`](#12-load-balancer), which will distribute the requests among the healthy [`Web Servers`](#21-web-server) of the application. We’ll use the *Round Robin* algorithm for our [`Load Balancer`](#12-load-balancer) service. This will help distribute the load evenly and avoid overloading individual servers. 

----

### **4. Database**

Databases are the foundation of many applications, and choosing the right database is crucial for the system’s performance, scalability, and availability. In general, databases are divided into two main types:  

- **Relational Database**: Stores data in a structured format, organized in tables with rows and columns, with support for ACID transactions.

- **NoSQL Database**: Stores data in documents, columns, or key-value pairs and graphs, focusing on scalability, availability, and partitioning, but without guaranteeing ACID.

#### **4.1 Relational Database**

Relational databases are based on the concept of **ACID**, ensuring reliability in transactions:  

- **Atomicity**: All operations in a transaction are executed successfully, or none are applied.  
- **Consistency**: Any transaction takes the database from one valid state to another, without violating integrity rules.  
- **Isolation**: Transactions are isolated from each other until completed, avoiding side effects.  
- **Durability**: After confirmation (*commit*), changes are permanent, even in case of system failure.  

##### **4.1.1 Replication**

To avoid scalability and availability issues, a common strategy is **data replication**, increasing fault tolerance and read capacity. Some of the main approaches include:  

- **Master-Slave Replication**: A single copy of data is replicated to multiple read servers, meaning there is potential for data loss if the master fails before newly written data can be replicated to the slaves. Additionally, if there are many writes, the read replicas may become overwhelmed with replaying writes and cannot perform as many reads.

- **Master-Master Replication**: Data is replicated between two servers for reading and writing, allowing for read scalability and fault tolerance. However, many master-master systems violate `ACID` and increase latency in operations. The critical point is noticed in conflict resolution as more write nodes are added and latency increases. 

- **Sharding**: Sharding distributes data across different databases, so each database only manages a subset of the data. Taking a user database as an example, as the number of users increases, more shards are added to the cluster.

Note that both strategies are used to increase availability and fault tolerance, but each has its advantages and disadvantages in project applicability.


###### 🙋🏽‍♂️ **Answer**

How would you handle a scenario involving millions of users and their followers? How would you model this relationship in a relational database?  

#### **4.2 NoSQL Database**

NoSQL databases are an alternative to relational databases and are designed to meet requirements involving **denormalized data and grouping information into a single document**. In general, NoSQL databases are described as **non-normalized**, where data grouping is done by the application, prioritizing **availability and partitioning**.  


- **Key-Value Stores**: Stores data in key-value pairs.
- **Document Stores**: Stores data in JSON-like documents.
- **Column-Family Stores**: Stores data in columns instead of rows.


##### 📖 **Use Case**

We’ll use a relational database with replication, called [`SQL Write Master-Slave`](#43-sql-write-master-slave), to increase availability and fault tolerance. We’ll use the **master-slave replication** strategy to distribute data efficiently.  

Additionally, we’ll employ a NoSQL database called [`NoSQL Database`](#42-nosql-database) to store **relationship graph data**, facilitating the search for users and followers.  
    
----

### **5. Content Delivery Networks (CDN)**  


![Image title](/images/system-design-cdn.jpg)

How do I deliver my content quickly to a user in China, knowing that my server is in São Paulo? This question guides the existence of **CDNs (Content Delivery Networks)**, geographically distributed networks of servers that accelerate the delivery of static content to users.  

Using CDNs requires modifying applications to consider the user’s context, ensuring that static content URLs point to a specific CDN.  

Using CDNs promotes:

- [x] Lower latency between user requests and fulfillment
- [x] Reduced network traffic and costs
- [x] API servers do not need to handle requests that the CDN serves, such as static content (images, videos, etc.)


#### **5.1 CDN Push/Pull**

CDNs can be configured to operate in two main modes:

- **Push CDN**: Content is sent to the CDN and cached before being requested.
- **Pull CDN**: Content is requested from the CDN and cached only when requested.


##### 🪧 **Tip**

CDN costs can be significant, depending on traffic volume. However, they should be compared to the additional costs you would incur by **not using** a CDN, such as higher bandwidth consumption and server overload.  

##### 🚨 **Attention**

Configuring the **TTL (Time to Live)** in the CDN is essential to avoid distributing outdated content. A TTL that is too high may result in displaying old versions of content, while a TTL that is too low may increase the load on origin servers.

##### 📖 **Use Case**

We’ll use the CDN strategy to store **user profile images, videos, and static content**, replicating them regionally to improve performance and availability. This way, the same content can be consumed by thousands of users in different parts of the world with **lower latency and greater efficiency**.  

## **Fundamental Principles of System Design**

### **1. Failovers and SPOF**  

Failover is a redundancy mechanism that ensures service continuity in case of failures in systems or software components. This concept is fundamental in critical environments, where availability and reliability are essential.  


- [x] Continuity of the system must be guaranteed in case of failures.  
- [x] Includes automatic switching to backups or active replicas.

Some common failover strategies have already been covered in previous topics, such as:  

- [**Loading Balancing**](#3-load-balancing): Load distribution among servers to ensure high availability and performance.
- [**Database Replication**](#411-replication): Data duplication to increase availability and fault tolerance.

#### **1.1 Single Point of Failure (SPOF)**

![Image title](/images/system-design-SPOF.jpg)

The discussion about failover begins with identifying the system’s critical failure points. The **Single Point of Failure (SPOF)** refers to any component whose failure can compromise the entire system.


##### 🙋🏽‍♂️ **Answer**

1. What are the possible failure points you’ve identified in your current system?

2. How would you handle an SPOF in a distributed system?

##### 📖 **Use Case**

To avoid failures, we’ll use **database replication** to ensure data availability and **load balancing** to distribute requests among application servers. This configuration must consider costs and geographical availability.  

----

### **2. Consistent Hashing**  

![Image title](/images/system-design-consistent-hashing.png)

Consistent Hashing is an efficient technique for distributing data among servers, minimizing redistributions when there are changes in the cluster structure. Understanding this concept is essential to understanding how servers are allocated and how the workload is distributed.  

📖 For further reading, check out [The Simple Magic of Consistent Hashing](https://www.paperplanes.de/2011/12/9/the-magic-of-consistent-hashing.html).  


----

### **3. Consistency patterns**

Consistency refers to the guarantee that all nodes in a system have the same data. There are several patterns for synchronizing data and ensuring that clients have a consistent view:  

- **Eventual Consistency**: After a write, reads will eventually see it (usually in milliseconds). Data is replicated asynchronously.
- **Strong Consistency**: After a write, readers will see it. Data is replicated synchronously. 
- **Weak consistency**: After a write, readers may or may not see it. A best-effort approach is adopted.

#### 📖 **Use Case**

For the [`Timeline Service`](#33-timeline-service), we’ll use **eventual consistency**, allowing the tweet feed to be built and stored in database replicas.  

For the [`Fan Out Service`](#34-fan-out-service), we’ll apply **strong consistency**, ensuring that tweets are immediately persisted in the database.  

----

### **4. Availability patterns**

Availability is the guarantee that a system is always available to users. There are several availability patterns that can be used to ensure system availability:

- **Active-Passive**: One active server and one passive server. The passive server takes over when the active server fails.

- **Active-Active**: Both servers are active and can handle requests. If one fails, the other takes over.

For databases, the concept of availability is demonstrated through [`replication`](#411-replication).

----

### **5. Asynchronism and Queues**

Asynchronous flows help reduce the response time of complex operations that would otherwise be executed synchronously. **Queues** are used to store and process requests at opportune times, ensuring scalability and reliability.  

#### **5.1 Message and Task queues**

Message queues are used to temporarily store messages until they can be processed. Messages are sent to the queue and processed by one or more consumers. Message queues are useful for:

- **Decoupling**: Decoupling system components.
- **Scalability**: Handling traffic spikes.
- **Reliability**: Ensuring messages are processed.

Task queues are used to store tasks that need to be executed at a later time, but the principle is the same as message queues.

#### **5.2 Back pressure**

If a queue grows excessively, it can consume too much memory, causing cache failures, frequent disk reads, and degrading performance. **Back pressure** strategies help mitigate this risk.  

##### 🪧 **Tip**

Message queues are useful for decoupling system components, as in the case of Twitter. If you’re posting a tweet, the tweet can be posted instantly to your timeline, but it may take some time for your tweet to be delivered to all your followers.

##### 📖 **Use Case**

We’ll use **message queues** for services like [`User Graph Service`](#35-user-graph-service) and [`Search Service`](#37-search-service), ensuring updates (e.g., new tweets, profile changes) are propagated correctly.  

For the [`Notification Service`](#38-notification-service) and [`Timeline Service`](#33-timeline-service), we’ll use a **pub/sub** model for real-time delivery. This allows for greater scalability and efficiency, accepting some latency.  

## **Main Dilemmas**

### **1. Performance vs scalability**


![Image title](/images/system-design-performance-vs-scalability.png)

It’s not always easy to distinguish between **performance** and **scalability**, but understanding this difference is essential:  

* If you have a **performance** problem, your system is slow for a single user.
* If you have a **scalability** problem, your system is fast for a single user but slow under heavy load.

Generally, you should aim for maximum scalability with acceptable performance.

### **2. Latency vs throughput**

![Image title](/images/system-design-latency-throughput-bandwidth.png)

Another common dilemma is latency versus throughput, where latency is the time it takes for a request to be processed, and throughput is the number of requests a system can process in a given period.

* **Latency**: The time it takes for a request to be processed.
* **Throughput**: The number of requests a system can process in a given period.

Generally, you should aim for maximum throughput with acceptable latency.

----

### **2. Availability vs consistency**


Think about the problem between availability and consistency, where availability is the guarantee that a distributed system is always available to users, and consistency is the guarantee that all nodes in a database system have the same data at the same time. How to ensure this? This is a known problem called the [CAP Theorem](#21-cap-theorem).

#### **2.1 CAP Theorem**  

![Image title](/images/system-design-cap-teorm.png)

The CAP theorem states that a distributed system can only provide two of three desired characteristics: consistency, availability, and partition tolerance (the 'C', 'A', and 'P' in CAP).

Theorem that describes the balance between:  

1. **Consistency (Consistency)**: Each request receives the most recent response, which may not reflect changes from other requests. 

2. **Availability (Availability)**: Each request receives a response about whether it was successful or failed.

3. **Partition Tolerance (Partition Tolerance)**:  The system continues to function even when one or more components fail.


----

### **3. Stateful vs Stateless**

![Image title](/images/system-design-stateful-vs-stateless.jpg)

The difference between **stateful** and **stateless** is fundamental for modern architectures, such as microservices and distributed systems. A key differentiating characteristic is the ability to store information about the current state of user interactions, i.e., the ability to store information about the current state of user interactions, while the other treats each request as an independent and isolated transaction.


-  **Stateful** allows users to store, write, and query procedures and information already established over the Internet. The server monitors the state of each user session and stores information about old requests and interactions.

- **Stateless** does not retain information about previous user interactions. There is no knowledge or reference stored about previous transactions.

Most applications we use daily are stateful, however, the advancement of other technologies, such as microservices and containers, has made it easier to build and deploy stateless applications. Understanding the scope of the application, whether it is `stateful` or `stateless`, is fundamental to understanding the feasibility of the strategy in [load balancing](#3-load-balancing) that we need. 

#### 🪧 **Tip**

Stateful applications tend to use the same servers every time they process a request from a user to avoid distributed cache inconsistency.


#### 📖 **Use Case**

We’ll adopt a hybrid model:  
**Stateful** to store critical information, such as user profiles and tweets.  
**Stateless** for services reading and retrieving public data.

----

### **4. SQL vs NoSQL**

The dilemma of which database to use is a common question in SD. Here are some considerations for using one or the other:

- Reasons for SQL:

    - [x] Data is considered structured and has a strict schema.
    - [x] There is a high relationship between data and the need for complex joins.
    - [x] Transactions with the [ACID](#411-replication) structure are necessary.

- Reasons for NoSQL:
    - [x] Semi-structured data with dynamic or flexible information.
    - [x] The relationship between data is not as important, and there may be no need for complex joins.
    - [x] The storage will be many TB (or PB) of data.


#### 📖 **Use Case**
    We’ll use a hybrid approach, where we’ll use a relational database to store structured data, such as user information and tweets, and a NoSQL database to store semi-structured data, such as user feeds and trends.

----

### **5. APIs (REST/gRPC/GraphQL)**  

![Image title](/images/system-design-REST-gRPC-GraphQL.png)


The choice of data delivery service or communication between services is fundamental to the system’s performance and scalability. However, the choice of API type depends on the type of application and performance requirements. Here are some considerations for choosing between different types of APIs:


- **REST**: Widely used model, but the data structure is rigid and can be inefficient for communication between services.
- **gRPC**: Uses Protobufs for efficient and low-latency communication between services, but should only be used in internal environments.
- **GraphQL**: An alternative to REST that allows clients to request only the necessary data, but may be more complex to implement.

#### 📖 **Use Case**

In this case, we’ll choose to use GraphQL because it is a query language that allows accessing data more efficiently, returning only what is desired. Remember that GraphQL is ideal for complex, interrelated, and large data sources.

---

## **How DevOps Can Help in System Design?**


**DevOps** is an approach that integrates **development, operations, and security** to improve collaboration, automation, and system efficiency.  

A common mistake is to think that **DevOps is limited to infrastructure automation and only comes into play at the end of development**. In reality, many **architecture decisions directly affect performance and scalability**, making DevOps involvement essential from the beginning of **System Design (SD)**.  


Incorporating DevOps from the early stages of SD offers several benefits:  

- **Automate infrastructure deployment and management**: Tools like Terraform can be used to automate infrastructure deployment and management, directly impacting system scalability and availability.

- **Monitor and optimize system performance**: Monitoring tools like Prometheus and Grafana can be used to monitor performance and identify bottlenecks, aiding in system refinement.

- **Ensure security and compliance**: Security tools like Vault and compliance tools like Chef InSpec can be used to ensure system security and compliance.


DevOps is not just about **automation**, but about **integration, monitoring, and continuous security**. When incorporated into **System Design**, it improves system **scalability, reliability, and efficiency**, reducing costs and preventing critical issues before they occur. 🚀  


## **Case Study Results**

After exploring the key components, theories, and dilemmas of **System Design (SD)**, we arrived at our **scalable architecture for Twitter**. The focus is on **architecture and system component interactions**, rather than the specific implementation details of the frontend or backend. This is because SD aims to ensure **scalability, performance, availability, and reliability**, defining how components communicate, store, and process data efficiently. Details such as programming languages, frameworks, or the internal logic of each service are decisions that can vary depending on the team and project requirements.  

Below, we present a diagram illustrating the **System Design process** and the final result of our implementation:  

![System Design](/images/system-design-result.png)  
[Open Diagram - System Design](/images/system-design-result.png)  

Now, let's summarize how our architecture is structured into **layers and specialized services** to ensure **performance, availability, and scalability**.  

### **Application Layers**

Our application is divided into **four main layers**, each with specialized services:  

#### 1. Entry Layer

##### 1.1 **DNS & CDN:**  

We adopted **CDNs** to improve content distribution, such as images, videos, and audio, reducing global latency.  

##### 1.2 **Load Balancer**  
Distributes requests among multiple **Web Servers**, ensuring **high availability** and **load balancing**.  

#### 2. Application Layer

##### **2.1 Web Server**  

Processes client requests and routes them to the **appropriate APIs** for reading and writing.  

##### **2.2 Read API**  

Handles **fast and optimized** read operations.  

##### **2.3 Write API**  

Manages write operations, ensuring **consistency and security**.  

#### 3. Backend Services Layer

##### **3.1 Tweet Info Service**  

Manages specific tweet information, storing and retrieving tweet-related data.  

##### **3.2 User Info Service**  
Manages user-specific information.  

##### **3.3 Timeline Service**  
Builds users' timelines, optimizing tweet reads from the cache.  

##### **3.4 Fan Out Service**  
Distributes new content to followers, optimizing tweet dissemination via a `Pub/Sub` service.  

##### **3.5 User Graph Service**  
Maintains the structure of user connections, such as followers, followings, and similarity services.  

##### **3.6 Search API**  
Enables efficient searching for tweets and users.  

##### **3.7 Search Service**  
Indexes and searches tweets to provide fast and accurate results.  

##### **3.8 Notification Service**  
Manages user notifications.  

#### 4. Cache & Storage Layer

##### **4.1 Memory Cache**  
A caching service that reduces latency by storing frequently accessed data.  

##### **4.2 SQL Read Replicas**  

##### **4.3 SQL Write Master-Slave**  
A replication strategy for scalability and data consistency.  

##### **4.4 Object Store**  
Stores large volumes of media, such as images and videos.  

### **Meeting the Requirements**  

Now, let's analyze how the architecture meets the **functional and non-functional requirements** of Twitter.  

#### Functional Requirements  

1. **Posting a tweet**  
    - The **Write API** receives the request and stores the data in the **SQL Write Master-Slave** database while triggering the **Fan Out Service** to distribute the tweet to followers.  
    - The **Fan Out Service** operates as a parallel service, distributing the new tweet to followers and services such as `Pub/Sub`, search, and other components.  
    - The **Memory Cache** stores recent tweets to reduce retrieval time.  

2. **Following a user**  
    - The **User Graph Service** manages user connections, providing the necessary structure and intelligence.  
    - When a user follows another, this relationship is stored in the database and reflected in the cache.  

3. **Viewing the tweet feed**  
    - The **Read API** is a specialized service for optimized read queries.  
    - The **Timeline Service** composes the user's feed by retrieving tweets from both the cache and database.  

4. **Retweeting a tweet**  
    - The **Write API** records the action in the database without duplicating the tweet.  
    - The **Fan Out Service**, a distributed parallel service, redistributes the retweet to the followers of the user who retweeted it.  

5. **Searching for tweets**  
    - The **Search API** processes the query and interacts with the **Search Service**, which returns the most relevant results from the indexed tweets.  

6. **Receiving notifications**  
    - The **Notification Service** manages and delivers real-time notifications via the `Pub/Sub` service when there are new tweets or content from followed users.  

---

#### Non-Functional Requirements  

1. **Scalability**  
    - **Load Balancer** distributes the load.  
    - Separation between read and write APIs (`Write API` and `Read API`) enhances performance.  
    - **Memory Cache** speeds up frequent access.  
    - **SQL Read Replicas** increase read capacity.  

2. **Availability**  
    - Multiple **Web Servers** and **distributed APIs** ensure high availability.  
    - **CDN** improves global content delivery.  

3. **Consistency**  
    - **SQL Write Master-Slave** ensures data integrity.  
    - The system adopts an **eventual consistency model** for scalability without compromising the user experience.  

4. **Fault Tolerance**  
    - **Database replication** prevents data loss.  
    - **Memory Cache** reduces the impact of failures in core services.  
    - Distributed components ensure that failures in one service do not affect the entire system.  

Our architecture is designed to ensure **efficiency, performance, and reliability**, addressing both **functional** and **non-functional** requirements of the application.  

However, **System Design is an ongoing process** and should be revisited as the system grows and new needs arise.  

Based on the provided diagram and explanations, you can further explore and enhance the architecture by adding **new features, services, and optimizations**.  

Intentionally, we left some aspects open-ended, such as the **regional distribution of static content** and **database replicas**.  

How would you approach these challenges?  

👉 **How would you solve these challenges?**  

---

### **Conclusion**  

As we have seen, the principles of **System Design (SD)** are fundamental for building **scalable, robust, and distributed** systems. The primary focus of SD is not on the technology itself but on the **architecture and the desired behavior of the system**. The choice of technologies, frameworks, and messaging mechanisms should be based on **architectural needs** and the specific challenges that need to be addressed—not the other way around.  

Moreover, SD is not a static process. It requires a **continuous and iterative** approach, where decisions need to be revisited and adjusted as the system grows and new challenges emerge. Architectural choices always involve **trade-offs**, and understanding these compromises is essential for maintaining a balance between **performance, scalability, reliability, and cost**.  

Therefore, developing an effective system is not solely dependent on software engineers but requires a **multidisciplinary collaboration** between teams responsible for **planning, development, security, and infrastructure maintenance**. Only with this holistic vision is it possible to build systems that are **efficient, resilient, and future-proof**. Take advantage of the [questions](#case-study-results) and open-ended topics we intentionally left for you to explore the [references](#references) and deepen your understanding of each subject.  

---

### **References**  

This post is based on years of experience working with **System Design**, as well as various reference materials that have helped consolidate the knowledge shared here. The goal was to provide a broad overview of the key challenges and concepts in SD, but it is important to emphasize that each topic covered has **much greater depth and nuances**. Therefore, **it is highly recommended that the reader explore the cited references**, delve deeper into the topics, and apply the concepts in their own context. Learning about System Design is an ongoing process and becomes increasingly valuable as new technologies and architectures emerge.  

[IBM - CAP Theorem: What is the CAP Theorem?](https://www.ibm.com/br-pt/topics/cap-theorem)  

[Understanding Load Balancer Algorithms for Server Allocation](https://www.youtube.com/watch?v=_JHWUKaZ2Do&list=PLFmBNfkCoVHJq7FfY8MyZfEZGE4JIaUkW&index=5)  

[Consistent Hashing Explained - System Design Fundamentals](https://www.youtube.com/watch?v=qqqdAPM1l2M)  

[What is Single Point of Failure?](https://profissaocloud.com.br/glossario/o-que-e-single-point-of-failure)  

[The System Design Primer](https://github.com/donnemartin/system-design-primer)  

[Functional and Non-Functional Requirements: What Are They?](https://www.mestresdaweb.com.br/tecnologias/requisitos-funcionais-e-nao-funcionais-o-que-sao)  

[Stateful vs Stateless](https://www.redhat.com/pt-br/topics/cloud-native-apps/stateful-vs-stateless)  

[Twitter System Design](https://www.youtube.com/watch?v=o5n85GRKuzk&list=PLot-Xpze53le35rQuIbRET3YwEtrcJfdt&index=3)  

[Lecture 9 Scalability - Harvard Web Development - David Malan](https://www.youtube.com/watch?v=-W9F__D3oY4)  

[Definitive Guide for System Design Interviews](https://www.systemdesignhandbook.com/guides/system-design-interview/)  

[How to Effectively Use Caching to Improve Microservices Performance](https://dev.to/amplication/how-to-effectively-use-caching-to-improve-microservices-performance-21c1)  

---

### **Glossary of Terms**  

Some words and terms were highlighted throughout the text; here are their definitions:  

**trade-off**: A *trade-off* is a decision where choosing one option comes at the expense of another. For it to be considered a trade-off, the individual must necessarily forgo an alternative in their choice.  

**scaling**: *Scaling* refers to a system's ability to handle an increase in workload.  

**SD**: System Design.  

**ACID**: Atomicity, Consistency, Isolation, and Durability.  