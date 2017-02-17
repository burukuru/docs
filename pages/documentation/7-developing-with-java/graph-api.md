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

## 

