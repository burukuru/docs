---
title: Graql Rules
keywords: graql, reasoner
last_updated: January 2017
tags: [graql, reasoning]
summary: "Graql Rules"
sidebar: documentation_sidebar
permalink: /documentation/graql/graql-rules.html
folder: documentation
---

Grakn supports Graql-native, rule-based reasoning to allow automated capture and evolution of patterns within the graph. Graql reasoning is performed at query time and is guaranteed to be complete.

Thanks to the reasoning facility, common patterns in the graph can be defined and associated with existing ontology elements.
The association happens by means of rules. This not only allows to compress and simplify typical queries but offers the ability to derive new non-trivial information by combining defined patterns.

Provided the reasoning is turned on, once a given query is executed, Graql will not only query the graph for exact matches but will also inspect the defined rules to check whether additional information can be found (inferred) by combining the patterns defined in the rules. The completeness property of Graql reasoning guarantees that for a given content of the graph and the defined rule set, the query result shall contain all possible answers derived by combining database lookups and rule applications.

In this section we shall briefly describe the logics behind the rules as well as how can we define pattern associations by suitably defined rules.

## Graql Rules

Graql rules assume the following general form:

```
if [rule-body] then [rule-head]
```
People familiar with Prolog/Datalog, may recognise it as similar:

```
[rule-head] :- [rule-body].
```

In logical terms, we restrict the rules to be definite Horn clauses. These can be defined either in terms of a disjunction with at most one unnegated atom or an implication with the consequent consisting of a single atom. Atoms are considered atomic first-order predicates - ones that cannot be decomposed to simpler constructs.

In our system we define both the head and the body of rules as Graql patterns. Consequently, the rules are statements of the form:

```
q1 ∧ q2 ∧ ... ∧ qn → p
```

where qs and the p are atoms that each correspond to a single Graql statement. The left-hand-side of the statement (antecedent) then corresponds to the rule body with the right-hand-side (consequent) corresponding to the rule head.

The implication form of Horn clauses aligns more naturally with Graql semantics as we define the rules in terms of the left-hand-side and right-hand-side which directly correspond to the antecedent and consequent of the implication respectively.

## Graql Rule Syntax
In Graql we refer to the body of the rule as the left-hand-side of the rule (antecedent of the implication) and the head as the right-hand-side of the rule (consequent of the implication). Therefore, in Graql terms, we define rule objects in the following way:

```graql
$optional-name isa inference-rule,
lhs {
    ...;
    ...;
    ...;
},
rhs {
    ...;
};
```
where each dotted line corresponds to a single Graql Var. The rule variable is optional and can be omitted. It is useful however if we want to be able to refer to and identify particular rules in the graph. This way, as inference-rule is a concept, we can attach resources to it:

```graql
$myRule isa inference-rule,
lhs {
    ...;
    ...;
    ...;
},
rhs {
    ...;
};
$myRule has description 'this is my rule';
```

In Graql the left-hand-side of the rule is required to be a conjunctive pattern, whereas the right-hand-side should contain a single pattern. If your use case requires a rule with a disjunction on the left-hand side, please notice that using the disjunctive normal form it can be always decomposed into series of conjunctive rules.

A classic reasoning example is the ancestor example: the two Graql rules R1 and R2 stated below define the ancestor relationship which can be understood as either happening between two generations directly between a parent and a child or between three generations when the first generation hop is expressed via a parentship relation and the second generation hop is captured by an ancestor relation.

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

When adding rules such as those defined above with Graql, we simply use an `insert` statement, and load the rules, saved as a *.gql* file, into the graph in a standard manner, much as for an ontology.

Defining the above rules in terms of predicates and assuming left-to-right directionality of the roles, we can summarise them in the implication form as:

```
R1: parent(X, Y) → ancestor(X, Y)  
R2: parent(X, Z) ∧ ancestor(Z, Y) → ancestor(X, Y)
```

## Allowed Graql Constructs in Rules
The tables below summarise Graql constructs that are allowed to appear in LHS
and RHS of rules.   

We define atomic queries as queries that contain at most one potentially rule-resolvable statement.
That means atomic queries contain at most one statement that can potentially appear in the right-hand-side of any rule.

### Queries

| Description        | LHS | RHS
| -------------------- |:--|:--|
| atomic queries | ✓ | ✓ |
| conjunctive queries        | ✓ | x |
| disjunctive queries        | x | x |  

### Variable Patterns

| Description        | Pattern Example           | LHS | RHS
| -------------------- |:--- |:--|:--|
| `isa` | `$x isa person;` | ✓ | x |
| `id`  | `$x id "264597";` | ✓ | variable needs to be bound within the RHS  |
| `value` | `$x value contains "Bar";`  | ✓ | indirect only  |
| `has` | `$x has age < 20;` | ✓ | ✓ |
| `relation` | `(parent: $x, child: $y) isa parentship;` | ✓ | ✓ |
| resource comparison | `$x value > $y;`  | ✓ | x |
| `!=` | `$x != $y;` | ✓ | x |
| `has-scope` | `($x, $y) has-scope $z;$x has-scope $y;`  | ✓ | x |

### Type Properties

| Description        | Pattern Example   | LHS | RHS
| -------------------- |:---|:--|:--|
| `sub`        | `$x sub type;` | ✓| x |
| `plays-role` | `$x plays-role parent;` |✓| x |
| `has-resource`        | `$x has-resource firsname;` | ✓ | x |  
| `has-role`   | `marriage has-role $x;` | ✓ | x |
| `is-abstract` | `$x is-abstract;` | ✓ | x |
| `datatype` | `$x isa resource, datatype string;` | ✓| x |
| `regex` | `$x isa resource, regex /hello/;` | ✓ | x |

## Configuration options
Graql offers certain degrees of freedom in deciding how and if reasoning should be performed. Namely it offers two options:

* **whether reasoning should be on**. This option is self-explanatory. If the reasoning is not turned on, the rules will not be triggered and no knowledge will be inferred.
* **whether inferred knowledge should be materialised (persisted to the graph) or stored in memory**. Persisting to graph has a huge impact on performance when compared to in-memory inference, and, for larger graphs, materialisation should either be avoided or queries be limited by employing the _limit_ modifier, which allows termination in sensible time.


## Comments
Want to leave a comment? Visit <a href="https://github.com/graknlabs/docs/issues/42" target="_blank">the issues on Github for this page</a> (you'll need a GitHub account). You are also welcome to contribute to our documentation directly via the "Edit me" button at the top of the page.

{% include links.html %}
