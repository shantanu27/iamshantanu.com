---
layout: inner
title: 'Distributed Key-Value Stores with Strong (Causal) and Eventual Consistencies'
date: 2018-06-04 08:00:00
categories: Tech
tags: Java Multi-Threading Consistency AWS
featured_image: '/img/posts/thumbnails/04_consistency.png'
project_link: 'https://github.com/shantanu27/Multithreading-and-Consistency'
lead_text: 'Distributed Key-Value Stores with Strong (Causal) and Eventual Consistencies'
---

# Distributed Key-Value Stores with Strong (Causal) and Eventual Consistencies

### Scenario

It was required to build a key-value store which could handle storing and retrieving of data at scale. It was a cross-regional replicated key-value store. Since it spanned multiple regions, there were latencies between each region. When a GET request was sent by a client, the value was served by the nearest key-value store to the client. To handle the influx of more users, there were distributed Coordinators for each corresponding Datastore.

### What did I use?

- Java
- Multi-Threading
- AWS (For deploying Coordinators and Datastores in different regions)

### Consistency Models Implemented

Since the datastore was cross-regional, each datastore needed to have its own Coordinator which would be in-charge of a specific set of data. In particular, the coordinator was implemented to have the following behavior:
<br>
**1. ``GET`` Requests:** Send the request to the nearest datastore center to fetch the latest value for that key. 
<br>
**2. ``PUT`` Requests:** Check if it is the primary co-ordinator for the key. 
- If yes, then send out a ``PRECOMMIT`` to other datacenters so that they can block the ``GET`` requests on that key. 
- If no, then send out ``PRECOMMIT`` to other datacenters, and forward the request to the correct Coordinator. 

To accomodate different usecases for fetching and retrieving data, different consistency models were implemented.
<br><br>
#### Strong (Causal) Consistency

This consistency model ensured the following properties were true at all times:
<br>
**1. Strict Consistency:** From the client's perspective, the same key had the same value across all datacenter replicas
<br>
**2. Strict Ordering:** The order in which requests arrived at the coordinator were the order in which they were fulfilled for each key
<br>
**3. Atomic Operations:** All requests were atomic. The data was updated either at all datacenters or none.
<br>
**4. Controlled Access:** While a request was being fulfilled for a key, (either read or write), no other requests for that key could be done in any datacenter until the pending request was completed. Control for each key would need to be managed separately, so operations for two different keys would proceed unhindered by each other.

The Coordinators were implemented to be able to handle *non-causally* related requests concurrently. There were *priority queues* for each key ordered by timestamp. These queues were updated whenever a new ``GET``, ``PUT``, or ``PRECOMMIT`` operation came in. For each new request, it checked if the request could be served concurrently without blocking, and returned the result if it was possible. Else, it put the request in the queue and served the response when the queue cleared up.

<br>
#### Eventual Consistency 

Eventual consistency emphasizes performance over consistency. It makes no guarantee about the ordering of the incoming requests, and makes no guarantees that the values returned by the ``GET`` request across all replicas will be consistent with one another. The only condition which it maintained was that a newly received write operation must have had a higher timestamp than the last updated timestamp. If there was a request with a lower timestamp which came later, it just ignored that request. So, the datastores became *eventually* consistent. This was fairly easy to implement as there were less stringent checks on write operations, and had to only compare timestamps per key. 

### Conclusion

This was a very challenging project to implement because of the complexities involved in ensuring Strong Consistency. Although it didn't have any new technology involved, it required me to understand the synchronization concepts to ensure that I wrote *race-condition-free* code. It definitely helped me get a lot of perspective into the complexities involved in managing distributed datastores. 
