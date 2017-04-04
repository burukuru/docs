---
title: Advanced Java Development
keywords: java
last_updated: February 2017
tags: [java]
summary: "Transaction Processing & Multithreading in Java"
sidebar: documentation_sidebar
permalink: /documentation/developing-with-java/advanced-java.html
folder: documentation
---

{% include warning.html content="Please note that this page is in progress and subject to revision." %}

In this section we will be focusing on using the java api in a multi-threaded environment.   
Specifically, we will show you how to create multiple transaction which can affect the graph concurrently.

## Creating Concurrent Transactions

Transactions in Grakn are thread bound. 
Which means for a specific keyspace and thread only one transaction can be open at anytime.
So the following would result in an exception:

```java
graph1 = Grakn.session(Grakn.DEFAULT_URI, "MyGraph").open(GraknTxType.Write);
graph2 = Grakn.session(Grakn.DEFAULT_URI, "MyGraph").open(GraknTxType.Write);
```

This is because you never closed the first transaction `graph1`.

If you require multiple transactions open at the same time then you must do so on different threads. 
This is best demonstrated by an example. 
Let's say that you wish to create 100 entities of a specific type concurrently.  
The following will achieve that:

```java
GraknSession session = Grakn.session(Grakn.DEFAULT_URI, "MyGraph");
Set<Future> futures = new HashSet<>();
ExecutorService pool = Executors.newFixedThreadPool(10);

//Create sample ontology
GraknGraph graph = session.open(GraknTxType.WRITE);
EntityType entityType = graph.putEntityType("Some Entity Type");
graph.commit();

//Load the data concurrently
for(int i = 0; i < 100; i ++){
    futures.add(pool.submit(() -> {
        GraknGraph innerGraph = session.open(GraknTxType.WRITE);
        entityType.addEntity();
        innerGraph.commit();
    }));
}

for(Future f: futures){
    f.get();
}
```

As you can see each thread opened it's own transaction to work with.
Also note that we were able to safely pass `entityType` into different threads but this was only possible because:
1. We committed `entityType` before passing it around
2. We opened the transaction in each thread before trying to access `entityType`

## Issues With Concurrent Mutations 

### Locking Exceptions

When mutating the graph concurrently and attempting to load the same data simultaneously it is possible to encounter a `GraknLockingException`.  
When this exception is thrown on `commit()` it means that two or more transactions are attempting to mutate the same thing.
If this occurs it is recommended you retry the transaction.

### Validation Exceptions

Validation exceptions may also occur when mutating the graph concurrently. 
For example two transactions may be trying to create the exact same relationship and one of them may fail.
When this occurs it is recommended retrying the transaction. 
If the same exception occurs again then it is likely that the transaction contains a validation error that would have still occurred even in a single threaded environment.

## Comments
Want to leave a comment? Visit <a href="https://github.com/graknlabs/docs/issues/23" target="_blank">the issues on Github for this page</a> (you'll need a GitHub account). You are also welcome to contribute to our documentation directly via the "Edit me" button at the top of the page.


{% include links.html %}
