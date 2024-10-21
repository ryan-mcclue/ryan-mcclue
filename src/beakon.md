<!-- SPDX-License-Identifier: zlib-acknowledgement -->
CMS specialising in risk and compliance management.
  - contractor management:
     company/admin will create audits/assessments for workers
     so, document upload, managing statuses, time of checkins
     workers have accounts to fill these in

18th November free 5 days a week (minus exams)

so 4 and half weeks of 3days per week
exams 22 November - 5 December

what is principle that gives distinct edge over competitors?
what is tech stack?
what's an average day like?
TODO: main clients are contractor management, e.g. coca-cola security guards 




day1:
  - bring id etc.
  - take notes even on admin things (bring notepad, headphones)
  - perhaps bring an ergonomic keyboard later
  - ask what systems I'll need access to and get access
  - no harm asking questions, however google it first
  - give a daily status report of work done
  - ask clarifying questions, don't just be a yes man
  - git pull; ide; look through codebase; UI; rest API; 
    look at any long files for potential refactoring
  - look at ticket PRs and try and add some comments





REST:
  urls are resources
  query url params for identification and filtering, e.g. users/123?category=electronics
  POST and GET on same url different

try getting understanding of whole application flow, i.e. end-to-end
draw diagram and realise might not understand certain parts.
  - how to scale server and databases?
  - how to design bit.ly url shortener? how would the api work, e.g read-link, create-link etc.
    how would you scale this up? what type of database using? are using db sharding?
    how web using CDN or load balancer?
  - how would you design twitter? how would database look like to aggregate feeds from all people following efficiently?
  - for mobile how to perform caching? offline access? how to fetch data?

how does user uthentication work?
how does offline access work?
how is data being fed from server and how does it propagate?
TODO: graphQL is one-way and immutable?

more scaling, more reliable, more secure, more redundant, more stable
fast reads or fast writes important?
is uptime more important than 

databases how long for writes to propagate?
say for banking want strong consistency, i.e. don't want to get stale results

where are bottlenecks in the system?

databases reading from slaves, writing to master, recieving cache

Scaling Connectivity (web server):
Load balancer:
  Introduce a server in between DNS and actual server.
  This server just redirects requests to a number of servers to actually handle request.
  e.g. NGINX, AWS Elastic Load Balancer (round-robin, least load algorithms)
  Could also do this at the application level, e.g. the app talks to say a db server, a chat server etc.
Scaling Data (database server):
Caching:
  Insert a server between webservers and database server.
  e.g. memcached, redis
  Just storing keyvalue pairs, so can distribute amongst many machines.
  Often storing in memory, not on harddrive, so could lose on power outage (so only store redundant data)
  Could also use it to say store rate limit count for user login attempts
CDN (caching static assets):
  When requesting from a CDN domain, it will select best geographic location.
  e.g. AWS cloudfront, IBM softlayer 
  Could append say app.js?version=1 to ensure CDN cache will reload file

Indexing:
  Seems just add an index to whatever field querying a lot of? (BST under the hood)
  noSQL based on key-value pairs can scale better but can't do range queries like in a relational database
  storing binary data would just store say filename, which would reference a distributed file system?
   
Database Replication (scaling up reads):
Have read-only/slave databases which are copies of single write/master database
This works for distributing read-heavy applications

(everything prior to this is mainly scaling reads as most common)
Database Sharding (scaling up writes):
table(id, name, age)
Vertical sharding:
  - table0(id, name)
    table1(id, age)
    each on separate server
Horizontal sharding:
  amazon dynamodb
  - switch (id % 100)
    (0, 33) == server0
    (34, 66) == server1
    (67, 100) == server2


amazon elastic file system (distributed filesystem say for many user image files)
protobuf key-value binary like JSON, however rigid structure with .proto file 

NoSQL:
just key value pairs.
generally doesn't have ACID, e.g. transactions
scales automatically for lots of reads and writes (i.e. no need for sharding)

API Design:
db cursor to prevent seeing duplicate entries?
having more server logic may allow for greater updates, e.g. don't have to wait for lengthy app store approval times

Mobile Design:
To have independent teams, have unified common interface
A lot of triggering with event manager and pulling off queue in a one-way, e.g. `eventManager.trigger("buy", data); .register("buy", func);`
As using retained mode, would require say refresh() event listener to update changes.
(When logging out, want to invalidate refreshes)
For offline access, just abstract network retrieval to go through cache first.
