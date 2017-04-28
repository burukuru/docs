---
title: Graql Analytics using Java
keywords: analytics, machine-learning
last_updated: April 2017
tags: [analytics, examples, java]
summary: "How to use Graql analytics methods in Java"
sidebar: documentation_sidebar
permalink: /documentation/examples/java-analytics.html
folder: documentation
comment_issue_id: 27
---

If you have not yet set up the Grakn environment, please see the [Setup guide](../get-started/setup-guide.html).

## Introduction
In this example, we are going to introduce the analytics package using the basic genealogy example. You can find *basic-genealogy.gql* in the *examples* directory of the Grakn distribution zip. You can also find it on [Github](https://github.com/graknlabs/grakn/blob/master/grakn-dist/src/examples/basic-genealogy.gql). 

The first step is to load it into Grakn using the terminal. Start Grakn, and load the file as follows:

```bash
<relative-path-to-Grakn>/bin/grakn.sh start
<relative-path-to-Grakn>/bin/graql.sh -f ./examples/basic-genealogy.gql -k "genealogy"
```

You can test in the Graql shell that all has loaded correctly. For example:

```bash
<relative-path-to-Grakn>/bin/graql.sh -k genealogy
>>>match $p isa person, has identifier $i;
```


## Using Java APIs to Determine Clusters and Degree

We are going to determine clusters of people who are related by marriage and how large those clusters are using two graph algorithms:

* the cluster algorithm will identify which people are related 
* the degree algorithm will count the size of the cluster.

The maven dependencies for this project are:

```java
<dependency>
<groupId>ai.grakn</groupId>
<artifactId>titan-factory</artifactId>
<version>${grakn.version}</version>
</dependency>
<dependency>
<groupId>org.slf4j</groupId>
<artifactId>slf4j-nop</artifactId>
<version>1.7.20</version>
</dependency>
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-tomcat</artifactId>
<version>1.2.4.RELEASE</version>
</dependency>
```

The Titan factory is required because engine is configured with Titan as the backend. The `slf4j-nop` dependency is a work around because we are using a later version of Spark. The spring framework dependency is a current bug workaround in GRAKN.AI version 0.12.0.

## Connect to Grakn Engine

The first thing to do is connect to the running engine instance and check that the graph contains the data that we expect. This can be achieved with the following code:

```java
private static void testConnection() {
}
```

## Compute the Clusters

Now that we have established the connection to Grakn engine works we can obtain the graph object and compute the clusters that we are interested in:

```java
private static Map<String, Set<String>> computeClusters() {
}
```

Here we have used the `in` syntax to specify that we want to compute the clusters while only considering instances of the types marriage and person. The graph that we are effectively working on can be seen in the image below. We can see three clusters, but there are more in the basic genealogy example. The code above will have found all of the clusters and returned a `Map` from the unique cluster label to a set containing the ids of the instances in the cluster.

If we had not used the members syntax we would only know the size of the clusters not the members.

```graql
match
$x isa cluster has degree 3;
$y isa cluster has degree 5;
limit 2;
```

![Visualiser UI](/images/java-analytics-1.png)


## Persist the Cluster Information

Now that we have information about the clusters, it would be useful to add it to the graph so that we can visualise it. We can do this by creating a new entity type called `cluster` and a new relation type called `grouping` with the roles `group` and `member`. The ontology can be mutated as follows:

```java
private static void mutateOntology() {
}
```

We also have to ensure that the existing types in the ontology can play the specified roles.

Now all that is left is to populate the graph with the clusters and grouping relationships. To do this we can create a cluster for each result from the analytics query, which can then be attached to each of the concepts returned in the set of members:

```java
private static void persistClusters(Map<String, Set<String>> results) {
}
```

We can now switch to the visualiser, by browsing to [localhost:4567](http://localhost:4567), and execute this query:

```graql
match $x isa cluster;
```

If you explore the results you can now see the entities that are members of a given cluster, similar to the image shown below.

![Visualiser UI](/images/java-analytics-2.png)

## Compute the degrees

When using the visualiser it is probably quite useful to be able to see the size of the cluster before clicking on it. In order to add this information to the graph we will use another analytics function degree. The code to compute the degree is:

```java
private static Map<Long, Set<String>> degreeOfClusters() {
}
```

The `in` syntax has again been used here to restrict the algorithm to the graph shown below. Further, the `of` syntax has been used to compute the degree for the cluster alone. What is returned from this calculation is a `Map` from the degree which is a `long` to the ids of the instances that have that degree.

```graql
match $x isa cluster; $y ($x);
```

## Persist the Degrees

As we did when computing the clusters, we need to put the information back into the graph. This time we will attach a resource called `degree` to the cluster entity. The ontology mutation and persisting of the degrees is performed in a single method:


```java
private static void persistDegrees(Map<Long, Set<String>> degrees) {
}
```

We used a single transaction for this because it is a small graph.

We can now query for all the clusters in the visualiser, see their size and click on them to find the members. 

<!-- IMAGE TO DO -->

## Summary

The complete working code can be found here in our `sample-projects` repo on [Github](https://github.com/graknlabs/sample-projects/tree/master/example-analytics-genealogy).