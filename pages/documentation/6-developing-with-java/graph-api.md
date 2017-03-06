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

{% include warning.html content="Please note that this page is in progress and subject to revision." %}



<!--
What is the difference between the Graph API and Java Graql API?
Use the Graph API for:

* Manipulating and inserting
* A more efficient but advanced construction API for building API

Java Graql API?
Used for Querying - for traversals
-->

<!-- From Jo: @Filipe - please include something about rules and the graph API. This from earlier documentation - feel free to re-use or update... -->

### Rule Java API
All rule instances are of type inference-rule which can be retrieved by:

```java
RuleType inferenceRule = graknGraph.getMetaRuleInference();
```

Rule instances can be added to the graph both through the Graph API as well as through Graql. Let's consider the ancestor example:

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
As there is more than one way to define Graql patterns through the API, there are several ways to construct rules.

Through the Pattern factory:

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
