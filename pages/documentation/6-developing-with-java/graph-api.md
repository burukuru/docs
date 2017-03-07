---
title: Graph API
keywords: java
last_updated: March 2017
tags: [java]
summary: "The Graph API."
sidebar: documentation_sidebar
permalink: /documentation/developing-with-java/graph-api.html
folder: documentation
---

{% include warning.html content="Please note that this page is in progress and subject to revision." %}



<!--
What is the difference between the Graph API and Java Graql API?
Use the Graph API for:

* Manipulating and inserting
* A more efficient but advanced construction API for building API

Java Graql API?
Used for Querying - for traversals
-->





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

In the [Basic Ontology documentation](../building-an-ontology/basic-ontology.html) section we introduced a simple ontology built using graql.
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
## Loading Data

Now that we have created a we can load in some data using the graph api. 
Lets compare how a graql statement maps to a graph api call:

```
insert $x isa person has firstname "Bob";
```
    
With the java graph API will be:    

```java
Resource bobName = firstname.putResource("Bob"); //Create the resource
person.addEntity().hasResource(bobName); //Link it to an entity
```   

What if we want to create a relation between some entities. 
In graql we know we can do the following:

```
insert
    $x isa person has firstname "Bob";
    $y isa person has firstname "Alice";
    $z (spouse1: $x, spouse2: $y) isa marriage;
```

With the Graph API this would be:

```java
//Create the resources
Resource bobName = firstname.putResource("Bob"); 
Resource aliceName = firstname.putResource("Alice");

//Create the entities
Entity bob = person.addEntity();
Entity alice = person.addEntity();

//Create the actualt relationships
Relation bobAndAliceMarriage = marriage.addRelation().putRolePlayer(spouse1, bob).putRolePlayer(spouse2, alice);
```

Ooops we forgot to add a picture:

```
$z has picture "www.LocationOfMyPicture.com";
```

ooops, we wanted to do that using the graph API:

```java
Resource bobAndAlicePicture = picture.putResource("www.LocationOfMyPicture.com");
bobAndAliceMarriage.hasResource(bobAndAlicePicture);
```

The above should enable you to load data into any ontology you create.

## Building A Hierarchical Ontology  

In the [Hierarchical Ontology documentation](../building-an-ontology/hierachical-ontology.html) section we discussed how it is possible to create more expressive ontologies by creating a type hierarchy.
How can we create the same hierarchy using the graph API.  
Lets look at a quick example, this graql statement:

```
insert 
    event sub entity;
    wedding sub event;
```

becomes the following with the Graph API:

```java 
EntityType event = graph.putEntityType("event");
EntityType wedding = graph.putEntityType("event").superType(event);
```

Simple, from there all operations remain the same. 
It is worth remembering that adding a type hierarchy does allow you to create a more expressive database but you will need to follow more validation rules.  
Please checkout [this](../advanced-grakn/validation.html) section for more details.

### Rule Java API
All rule instances are of type inference-rule which can be retrieved by:

```java
RuleType inferenceRule = graknGraph.getMetaRuleInference();
```

Rule instances can be added to the graph both through the Graph API as well as through Graql. Let's consider an example:

```graql
$R1 isa inference-rule,
lhs {
    (parent: $p, child: $c) isa Parent;
},
rhs {
    (ancestor: $p, descendant: $c) isa Ancestor;
};

$R2 isa inference-rule,
lhs {
    (parent: $p, child: $c) isa Parent;
    (ancestor: $c, descendant: $d) isa Ancestor;
},
rhs {
    (ancestor: $p, descendant: $d) isa Ancestor;
};
```

As there is more than one way to define Graql patterns through the API, there are several ways to construct rules. One options is through the Pattern factory:

```java
Pattern rule1LHS = var().rel("parent", "p").rel("child", "c").isa("Parent");
Pattern rule1RHS = var().rel("ancestor", "p").rel("descendant", "c").isa("Ancestor");

Pattern rule2LHS = and(
        var().rel("parent", "p").rel("child", "c").isa("Parent')"),
        var().rel("ancestor", "c").rel("descendant", "d").isa("Ancestor")
);
Pattern rule2RHS = var().rel("ancestor", "p").rel("descendant", "d").isa("Ancestor");
```

If we have a specific `GraknGraph graph` already defined, we can use the Graql pattern parser:

```java
Pattern rule1LHS = and(graph.graql().parsePatterns("(parent: $p, child: $c) isa Parent;"));
Pattern rule1RHS = and(graph.graql().parsePatterns("(ancestor: $p, descendant: $c) isa Ancestor;"));

Pattern rule2LHS = and(graph.graql().parsePatterns("(parent: $p, child: $c) isa Parent;(ancestor: $c, descendant: $d) isa Ancestor;"));
Pattern rule2RHS = and(graph.graql().parsePatterns("(ancestor: $p, descendant: $d) isa Ancestor;"));
```

We conclude the rule creation with defining the rules from their constituent patterns:

```java
Rule rule1 = inferenceRule.addRule(rule1LHS, rule1RHS);
Rule rule2 = inferenceRule.addRule(rule2LHS, rule2RHS);
```


## Comments
Want to leave a comment? Visit <a href="https://github.com/graknlabs/docs/issues/23" target="_blank">the issues on Github for this page</a> (you'll need a GitHub account). You are also welcome to contribute to our documentation directly via the "Edit me" button at the top of the page.


{% include links.html %}