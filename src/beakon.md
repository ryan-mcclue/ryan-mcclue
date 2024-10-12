<!-- SPDX-License-Identifier: zlib-acknowledgement -->
CMS specialising in risk and compliance management.
  - contractor management:
     company/admin will create audits/assessments for workers
     so, document upload, managing statuses, time of checkins
     workers have accounts to fill these in
     TODO: main clients are contractor management, e.g. coca-cola security guards 

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
