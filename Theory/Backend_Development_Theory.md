# Backend Development

Main Resource: [Backend Roadmap](https://roadmap.sh/backend), [Full Stack Development](https://roadmap.sh/full-stack)

## JWT

<b>JWT (JSON Web Token)</b> is an open standard for securely transmitting information between parties as a JSON object. It consists of three parts: 
* <b>Header</b>: specifies the token type and algorithm used for signing
* <b>Payload</b>: contains the claims or the data being transmitted
* <b>Signature</b>: is used to verify the token’s integrity and authenticity 

JWTs are commonly used for <b>authentication</b> and <b>authorization</b> purposes, allowing users to <b>securely transmit</b> and validate their <b>identity</b> and permissions across web applications and APIs. They are compact, self-contained, and can be easily transmitted in <b>HTTP headers</b>, making them popular for modern web and mobile applications.

## Domain Name

It translates to an IP address, which is a numerical identifier used by computers to locate and connect to servers. A domain name consists of two main parts: 
* <b>Top-level domain</b>: (e.g., “.com” in “example.com”) 
* <b>Second-level domain</b>: (e.g., “example” in “example.com”) 

Domain names are managed by domain name registrars and are essential for establishing a web presence, providing a user-friendly way to navigate to websites instead of using numeric IP addresses.


## DNS (Domain Name System)

* <b>Hierarchical</b>, <b>decentralized</b> naming system for computers, services, or other resources connected to the Internet or a private network. 
* It <b>translates</b> human-readable <b>domain names</b> (like www.example.com) into <b>IP addresses</b> (like 192.0.2.1) that computers use to identify each other. 
* The system uses a <b>tree-like</b> structure with root servers at the top, followed by top-level domain servers (.com, .org, etc.), authoritative name servers for specific domains, and local DNS servers. 

DNS servers distributed worldwide work together to resolve these queries, forming a global directory service. DNS is crucial for the functioning of the Internet, enabling users to access websites and services using memorable names instead of numerical IP addresses. It also supports <b>email routing</b>, <b>service discovery</b>, and other network protocols.

## PostgreSQL

* [PostgreSQL](https://www.postgresql.org/) is an advanced, open-source <b>relational</b> database management system (RDBMS) 
* It supports complex queries, foreign keys, and full-text search. 
* Allows users to define custom data types, operators, and functions. It supports ACID (Atomicity, Consistency, Isolation, Durability) properties for reliable transaction processing and offers strong support for <b>concurrency</b> and data integrity. 

### Relational Databases

* Type of database management system (DBMS) that organizes data into <b>structured</b> tables with rows and columns, using a <b>schema</b> to define data relationships and constraints. 
* They employ <b>Structured Query Language (SQL)</b> for querying and managing data, supporting operations such as data retrieval, insertion, updating, and deletion. 
* Relational databases enforce data integrity through keys (<b>primary</b> and <b>foreign</b>) and <b>constraints</b> (such as <b>unique</b> and <b>not-null</b>).

Examples of relational databases include MySQL, MariaDB, MS SQL, SQLite, PostgreSQL, and Oracle Database.


## Redis

* [Redis](https://redis.io/) is an open-source, <b>in-memory data structure</b> store known for its speed and versatility. 
* Data types: strings, lists, sets, hashes, and sorted sets and more.
* Provides functionalities such as <b>caching</b>, <b>session management</b>, real-time analytics, and message brokering. 
* Redis operates as a <b>key-value</b> store, allowing for rapid read and write operations, and is often used to enhance performance and scalability in applications. 
* Supports persistence options to <b>save data to disk</b>, replication for high availability, and clustering for horizontal scaling. Redis is widely used for scenarios requiring <b>low-latency access</b> to data and high-throughput performance.

### Memcached - Redis Alternative

Memcached (pronounced variously mem-cash-dee or mem-cashed) is a general-purpose <b>distributed memory-caching system</b>. It is often used to speed up dynamic database-driven websites by caching data and objects in RAM to reduce the number of times an external data source (such as a database or API) must be read. 
* Free and open-source software
* It depends on the <b>libevent</b> library. 
* Memcached’s APIs provide a very large hash table distributed across multiple machines. When the table is full, subsequent inserts cause older data to be purged in the least recently used (LRU) order.

Applications using Memcached typically layer requests and additions into RAM before falling back on a slower backing store, such as a database.


...TO BE CONTINUED...

