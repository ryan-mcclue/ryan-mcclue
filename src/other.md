<!-- SPDX-License-Identifier: zlib-acknowledgement -->
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
