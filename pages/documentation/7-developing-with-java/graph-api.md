---
title: Graph API
keywords: java
last_updated: Development 2016
tags: [java]
summary: "The Graph API."
sidebar: documentation_sidebar
permalink: /documentation/developing-with-java/graph-api.html
folder: documentation
---

## The Java Graph API

The Java Graph API is the low level API which encapsulates the Grakn knowledge model we discussed in the [Knowledge Model](../the-basics/grakn-knowledge-model.html) section.
It provides java object constructs for both Ontological Elements (Entity Types, Relation Types, etc. . . ) and Data Instances (Entities, Relations, etc . . .).

Using these constructs allows you to build a graph programmatically. 
To setup using this API please read through our [Setup Guide](../developing-with-java/development-setup.html) .

## Graph API Vs Graql

In this section we will be focusing primarily on the methods provided by the `GraknGaph` interface. 
Interacting with Graql queries via `GraknGraph.graql()` will be discussed [here](../developing-with-java/java-graql.html).

You may choose to operate exclusively via graql statements.
However, all graph mutation operations executed by Graql statements use the `GraknGaph` interface.
So if you are primarily interested in mutating the graph pragmatically as well as doing simple concept lookups the `GraknGaph` interface will be sufficient. 
On the other hand, advanced querying is better suited for Graql Statments

## Building An Ontology With The Graph API

In the [Basic Ontology documentation](./basic-ontology.html) section we introduced a simple ontology built using graql.
Lets see how we can build the same ontology exclusively via the graph API.
First we need a graph. For this example lets just use an in memory graph:

```java
GraknGraph graph = Grakn.factory(Grakn.IN_MEMORY, "MyGraph").getGraph();
```

Because we are using an object orientated API which needs to compile we need to define our constructs before we can use them.    
So lets begin by defining our Resource Types since they are used everywhere:

    
Now lets take a look at our Role and Relation Types first. In Graql these are:

```graql
  identifier sub resource datatype string;
  firstname sub resource datatype string;
  surname sub resource datatype string;
  middlename sub resource datatype string;
  picture sub resource datatype string;
  age sub resource datatype long;
  birth-date sub resource datatype string;
  death-date sub resource datatype string;
  gender sub resource datatype string;
```

These resource types can be built with the Graph API with:

```java
ResourceType identifier = graph.putResourceType("identifier", ResourceType.DataType.STRING);
ResourceType firstname = graph.putResourceType("firstname", ResourceType.DataType.STRING);
ResourceType surname = graph.putResourceType("surname", ResourceType.DataType.STRING);
ResourceType picture = graph.putResourceType("picture", ResourceType.DataType.STRING);
ResourceType age = graph.putResourceType("age", ResourceType.DataType.LONG);
ResourceType birthDate = graph.putResourceType("birth-date", ResourceType.DataType.STRING);
ResourceType deathDate = graph.putResourceType("death-date", ResourceType.DataType.STRING);
ResourceType gender = graph.putResourceType("gender", ResourceType.DataType.STRING);
```

Now lets take a look at our Role and Relation Types:

```graql
  marriage sub relation
    has-role spouse1
    has-role spouse2
    has-resource picture;

  spouse1 sub role;
  spouse2 sub role;

  parentship sub relation
    has-role parent
    has-role child;

  parent sub role;
  child sub role;
```

Using the graph API this is built with: 

```java
RoleType spouse1 = graph.putRoleType("spouse1");
RoleType spouse2 = graph.putRoleType("spouse2");
RelationType marriage = graph.putRelationType("marriage")
                            .hasRole(spouse1);
                            .hasRole(spouse2);
marriage.hasResource(picture);
                           
RoleType parent = graph.putRoleType("parent");
RoleType child = graph.putRoleType("child");
RelationType marriage = graph.putRelationType("marriage")
                            .hasRole(spouse1);
                            .hasRole(spouse2);

```

Now our Entity Types:

```
  person sub entity
    has-resource identifier
    has-resource firstname
    has-resource surname
    has-resource middlename
    has-resource picture
    has-resource age
    has-resource birth-date
    has-resource death-date
    has-resource gender
    plays-role parent
    plays-role child
    plays-role spouse1
    plays-role spouse2;
```

Again, with the graph API this is:

```java
EntityType person = graph.putEntityType("person")
                        .plasRole(parent)
                        .plasRole(child)
                        .plasRole(spouse1)
                        .plasRole(spouse2);
                        
person.hasResource(identifier);
person.hasResource(firstname);
person.hasResource(surname);
person.hasResource(middlename);
person.hasResource(picture);
person.hasResource(age);
person.hasResource(birth-date);
person.hasResource(death-date);
person.hasResource(gender);
```
Now lets commit our ontology:

```java
graph.commitOnClose();
graph.close();
```