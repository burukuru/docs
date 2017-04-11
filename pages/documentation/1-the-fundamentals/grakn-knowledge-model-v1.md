---
title: Grakn Knowledge Model
keywords: setup, getting started, basics
last_updated: December 15th, 2016
tags: [getting-started, reasoning, graql]
summary: "Introducing the basics of the Grakn knowledge model."
sidebar: documentation_sidebar
permalink: /documentation/the-fundamentals/grakn-knowledge-model.html
folder: documentation
comment_issue_id: 17
---

{% include warning.html content="Please note that this page is in progress and subject to revision." %}

In Grakn, a graph is made of two layers: the ontology layer and the data layer.

## Ontology

The ontology is a formal specification of all the relevant concepts and their meaningful associations in a given application domain. It allows objects and relationships to be categorised into distinct types, and for generic properties about those types to be expressed. Specifying the ontology enables automated reasoning over the represented knowledge, such as the extraction of implicit information from explicit data (inference) or discovery of inconsistencies in the data (validation).   

Grakn ontologies use four types of `concepts` for modeling domain knowledge:   

* **Entity Types**: Types of objects or things in the domain.
	* For example, `person`, `man`, `woman`.    

* **Relation Types**: Types of n-ary relationships between different domain instances.
	* For example, `marriage`, which is typically a relationship between two instances of entity types (`woman` and `man`), playing roles of `wife` and `husband`, respectively.

* **Role Types**: Types of roles involved in specific relationships.
	* For example, `wife`, `husband`.     

* **Resource Types**: Types of attributes associated with domain instances.
	* For example, `name`, or `age`.    

The categorization of concept types is effectively enforced in the Grakn knowledge model by declaring every concept type as a subtype (i.e. an extension) of exactly one of the four corresponding, built-in concept types:

* `entity`
	* e.g. `person sub entity` 	
* `relation`
	* e.g. `marriage sub relation`	 
* `role`
	* e.g. `wife sub role`
* `resource`
	* e.g. `name sub resource`


Different concept types can be related to each other via four constructs (see Figure 1):

* **sub**:  expressing that a concept type is a subtype (i.e., an extension) of another one.
	* For example, `woman sub person`, `man sub person`.
* **has-resource**: expressing that a concept type can be associated with a given resource type.
	* For example, `person has-resource name`.
* **plays-role**: expressing that instances of a given concept type are allowed to play a specific role.
	* For example, `woman plays-role wife`, `man plays-role husband`.
* **relates**: expressing that a given relation type involves a specific role.
	* For example, `marriage relates wife`, `marriage relates husband`.

Additionally, all concept types can be declared as `abstract`, meaning that they cannot have any direct instances. For example, `person is-abstract` expresses that the only instances of `person` can be those that belong to more specialised subtypes of `person`, e.g., `man` or `woman`.

![The concept types and their permitted associations via Grakn ontology constructs.](/images/knowledge-model-fig1.png)

*Figure 1: The concept types and their permitted associations via Grakn ontology constructs.*

<!-- This section may possibly be better if moved into a different, more low level page -->
A well-formed Grakn ontology is required to satisfy the following structural properties (see examples in the following section):

* each concept type can have at most one direct supertype,
* each role type must be involved in exactly one relation type,
* each relation type must involve at least two distinct role types, <!-- I think an example would help here-->
* each relation type must involve the same number of roles as its direct supertype. In such a case, every role type involved in the subtype relation must be a (possibly indirect) subtype of exactly one role type involved in the supertype relation.   <!-- definitely need an example or diagram -->



### Inheritance

As in object-oriented programming, the inheritance mechanism in Grakn enables subtypes to automatically take on some of the properties of their supertypes. This simplifies the construction of ontologies and helps keep them succinct. The Grakn knowledge model imposes inheritance of all `has-resource` and `plays-role` constraints on entity, relation and resource types.  

For example, in the ontology depicted in Figure 2, the entity types `woman` and `man` inherit the constraints `has-resource name`, `plays-role partner1`, `plays-role partner2` from their supertype `person`. Similarly, the relation type `marriage` inherits the constraint `has-resource since` from `partnership`.


![A sample of a Grakn ontology.](/images/knowledge-model-fig2.png)

*Figure 2: An example of Grakn ontology.*

The ontology in Figure 2 consists of the following constraints:

**Type declarations** (omitted in the figure):

* `person sub entity`
* `partnership sub relation`
* `partner1 sub role`
* `partner2 sub role`
* `name sub resource`
* `since sub resource`

Note, that because of the transitivity of the `sub` construct, the type declarations for the remaining concept types in the ontology follow implicitly.


**Explicit constraints**:

* `person has-resource name`
* `person plays-role partner1`
* `person plays-role partner2`
* `woman sub person`
* `woman plays-role wife`
* `man sub person`
* `man plays-role husband`
* `wife sub partner1`
* `husband sub partner2`
* `partnership has-resource since`
* `partnership relates partner1`
* `partnership relates partner2`
* `marriage sub partnership`
* `marriage relates wife`
* `marriage relates husband`

**Inherited constraints**:

* `woman has-resource name`
* `woman plays-role partner1`
* `woman plays-role partner2`
* `man has-resource name`
* `man plays-role partner1`
* `man plays-role partner2`
* `marriage has-resource since`


## Data

The data is expressed by instantiating specific types of entities, relations, and concrete resources they are associated with, and assigning roles to the instances played for particular relations. There are three types of data instances:

* **Entities**: instances of entity types, for example, `insert $x isa woman` creates an instance of the entity type `woman`,

* **Relations**: instances of relation types, for example, `insert (wife:$x, husband:$y) isa marriage` creates an instance of the relation type `marriage` between some instances `$x`, playing the role of `wife`, and `$y`, playing the role of `husband`.

* **Resources**: instances of resource types being associated with particular instances, for example, `insert $x has-name "Elisabeth Niesz"` creates an instance of the resource type `name` with value "Elizabeth Niesz" associated with some instance `$x`. The unique identifiers for all instances are defined internally within the Grakn system.

Note, that unlike in the case of entity, relation and resource types, there are no instances of role types in the data layer.


### Validation

To ensure data is correctly structured (i.e. consistent) with respect to the ontology, all data instances are validated against the ontology constraints. All the explicitly represented ontology constraints, together with the inherited ones, form complete schema templates for particular concept types, which guide the validation and, consequently, the data entry process. For example, the ontology in Figure 2 establishes the following schema templates for the concept types `woman` and `marriage`:

```
insert
woman sub entity;

woman has-resource name,
	plays-role wife,
	plays-role partner1,
	plays-role partner2;

marriage sub entity;

marriage has-resource since,
	relates wife,
	relates husband;
```

 A data instance `inst` of a concept type `C` is valid, whenever the following conditions are satisfied:

 * `inst` instantiates only `C` (called the base type of `inst`) and indirectly all supertypes of `C`,
 * all types of resources associated with `inst` belong to the schema template of `C`,
 * for every role type `R` that `inst` plays in any relation instance in the data, there is a constraint `plays-role R` in the schema template of `C`,
 * whenever `inst` is a relation instance involving the role type `R` there is a constraint `relates R` in the schema template of `C`.

For example, data illustrated in Figure 3 violates the ontology from Figure 2, by:

1. associating a resource `'Smith'` of type `maiden-name` with an instance `inst1` of the base type `woman` (which is not allowed in the ontology).
2. by using an instance `inst2` of the base type `person` in the `husband` role (while only those of type `man` are accepted by the ontology).

![Validation of data against the ontology.](/images/knowledge-model-fig3.png)

*Figure 3: Validation of data against the ontology.*

## Rule and Sub-Type Inference

Inference is a process of extracting implicit information from explicit data. Grakn supports two inference mechanisms:

1. type inference, based on the semantics of the `sub` hierarchies included in the ontology
2. rule-based inference involving user-defined IF-THEN rules.

Both mechanisms can be employed when querying the knowledge graph with Graql, thus supporting retrieval of both explicit and implicit information at query time.      

### Type Inference
The type inference is based on a simple graph traversal along the `sub` edges. Every instance of a given concept type is automatically classified as an (indirect) instance of all (possibly indirect) supertypes of that type. For example, whenever `woman sub person` is in the ontology, every instance of `woman` will be retrieved on the query `match $x isa person`. In case of roles, every instance playing a given role is inferred to also play all its (possibly indirect) super-roles. For example, whenever `inst` plays the role of wife in a relation of the type `marriage`, the system will infer that `inst` plays also the role of `partner1` in that relation, given the ontology from Figure 2.

The type inference is set ON by default when querying Grakn.  

### Rule-Based Inference
The rule-based inference exploits a set of user-defined datalog rules and is conducted by means of the  reasoner built naitively into Grakn. Every rule is declared as an instance of a built-in Grakn type `inference-rule`.

A rule is an expression of the form `lhs G1 rhs G2`, where `G1` and `G2` are a pair of Graql patterns. Whenever the left-hand-side (lhs) pattern `G1` is found in the data, the right-hand-side (rhs) pattern `G2` can be assumed to exist and optionally materialised (inserted). For example:


```
lhs
(child:$x, mother:$y) isa motherhood;
(sister:$y brother:$z) isa siblings;
rhs
(nephew:$x, uncle:$z) isa uncle-relation;
```

The rule above expresses that, whenever instance `inst1` has a mother `inst2` in the data, and `inst2` has a brother `inst3`, one can infer that `inst3` is an uncle of `inst1`, while `inst1` is a nephew of `inst3`.

The rule-based inference is set OFF by default when querying Grakn, and can be activated on request. For more detailed documentation on rules see [Graql Rules](../graql/graql-rules.html).


## Where Next?
Our [Quickstart Tutorial](../get-started/quickstart-tutorial.html) will show you how to load an ontology, rules and data into Grakn using Graql, and to make basic queries.

You can find additional example code and documentation on this portal. We are always adding more and welcome ideas and improvement suggestions. Please get in touch!

{% include links.html %}

## Comments
Want to leave a comment? Visit <a href="https://github.com/graknlabs/docs/issues/17" target="_blank">the issues on Github for this page</a> (you'll need a GitHub account). You are also welcome to contribute to our documentation directly via the "Edit me" button at the top of the page.
