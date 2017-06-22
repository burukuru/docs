---
title: XML Migration to Grakn
keywords: setup, getting started
last_updated: February 2017
tags: [migration]
summary: "This document will teach you how to migrate XML data into Grakn."
sidebar: documentation_sidebar
permalink: /documentation/migration/XML-migration.html
folder: documentation
comment_issue_id: 32
---

## Introduction
This is reference documenation for how to migrate XML data into grakn. Make sure you have set up the [Grakn environment](../get-started/setup-guide.html) before continuing. 

## Migration Shell Script for JSON
The migration shell script can be found in */bin* directory of your Grakn environment. We will illustrate its usage in an example below:

```bash
usage: migration.sh xml -template <arg> -input <arg> -keyspace <arg> -element <arg> -schema <arg> [-help] [-no] [-batch <arg>] [-active <arg>] [-uri <arg>] [-retry <arg>] [-verbose]
 
OPTIONS
 -a,--active <arg>     Number of tasks (batches) running on the server at
                       any one time. Default 25.
 -b,--batch <arg>      Number of rows to execute in one Grakn transaction.
                       Default 25.
 -c,--config <arg>     Configuration file.
 -e,--element <arg>    The name of the XML element to migrate - all others
                       will be ignored.
 -h,--help             Print usage message.
 -i,--input <arg>      Input XML data file or directory.
 -k,--keyspace <arg>   Grakn graph. Required.
 -n,--no               Write to standard out.
 -r,--retry <arg>      Retry sending tasks if engine is not available
 -s,--schema <arg>     The XML Schema file name, usually .xsd extension
                       defining with type information about the data.
 -t,--template <arg>   Graql template to apply to the data.
 -u,--uri <arg>        Location of Grakn Engine.
 -v,--verbose          Print counts of migrated data.
```

## XML Migration Basics

The steps to migrate XML data to GRAKN.AI are:

* define an ontology for the data to derive the full benefit of a knowledge graph
* create templated Graql to map the XML data to the ontology
* invoke the Grakn migrator through the shell script or Java API. The XML migrator will apply the template to each instance of the specified element `-element`, replacing the sections indicated in the template with provided data: the XML tags are the key to be used in the brackets `<>` and the content of each tag are the value of that key.

{% include note.html content="XML Migration makes heavy use of the Graql templating language. You will need a foundation in Graql templating before continuing, so please read through our [templating documentation](../graql/graql-templating.html) to find out more." %}

## Example: Plants

This example will migrate a single plant entity, along with various reosurces, from XML data into Grakn. This is a snippet of the XML data ([full example](https://www.w3schools.com/xml/plant_catalog.xml)): 

```xml
<CATALOG>
    <PLANT>
        <COMMON>Bloodroot</COMMON>
        <BOTANICAL>Sanguinaria canadensis</BOTANICAL>
        <ZONE>4</ZONE>
        <LIGHT>Mostly Shady</LIGHT>
        <PRICE>$2.44</PRICE>
        <AVAILABILITY>031599</AVAILABILITY>
    </PLANT>
    <PLANT>
        <COMMON>Columbine</COMMON>
        <BOTANICAL>Aquilegia canadensis</BOTANICAL>
        <ZONE>3</ZONE>
        <LIGHT>Mostly Shady</LIGHT>
        <PRICE>$9.37</PRICE>
        <AVAILABILITY>030699</AVAILABILITY>
    </PLANT>
    <PLANT>
        <COMMON>Marsh Marigold</COMMON>
        <BOTANICAL>Caltha palustris</BOTANICAL>
        <ZONE>4</ZONE>
        <LIGHT>Mostly Sunny</LIGHT>
        <PRICE>$6.81</PRICE>
        <AVAILABILITY>051799</AVAILABILITY>
    </PLANT>
    ...
</CATALOG>
```

Our ontology contains a single plant entity and various resources that are associated with it:

```gql
insert

plant sub entity
    has common
    has botanical
    has zone
    has light
    has price
    has availability;

name sub resource datatype string;
common sub name;
botanical sub name;
zone sub resource datatype string;
light sub resource datatype string;
price sub resource datatype double;
availability sub resource datatype long;
```

We want to insert one `plant` entity in the graph per `PLANT` tag in the XML data. This means that me must specify main element of the XML migration to be "PLANT". In the migration script you would do so by adding the parameter: `-element PLANT`. 

The XML template will be applied to each of the specified elements:

```
insert

$plant isa plant
    has common <COMMON.textContent>
    has botanical <BOTANICAL.textContent>
    has zone <ZONE.textContent>
    has light <LIGHT.textContent>
    has availability @long(<AVAILABILITY.textContent>);
```

This template contains a single insert query. As it will be applied to each element of the XML, you will have one insert query per `PLANT` element. 

The brackets are used to specify nested elements: in the XML `<BOTANICAL>` is a direct child of `<PLANT>` and so can be used whenever in the `PLANT` scope in the templant. 

Specifying the `textContent` returns the text from the selected element.

Before running any migration you need to ensure that the ontology has been loaded to the graph: 

```
./<grakn-install-location>/bin/graql.sh -f plants-ontology.gql -k plants
```

At this point you can run the XML migration script on the resources we have desribed above: 

```
./<grakn-install-location>/bin/migration.sh xml -i plants.xml -t plants-template.gql -e PLANT -k plants 
```

Adding the `-n` tag to the migration script will print the resolved graql statements to system out. This is a useful tactic to get used to the Graql langauge and know exactly what is being inserted into your graph:

```graql
insert $plant0 isa plant has common "Bloodroot" has botanical "Sanguinaria canadensis" has zone "4" has light "Mostly Shady" has availability 31599;
insert $plant0 isa plant has common "Columbine" has botanical "Aquilegia canadensis" has zone "3" has light "Mostly Shady" has availability 30699;
insert $plant0 isa plant has common "Marsh Marigold" has botanical "Caltha palustris" has zone "4" has light "Mostly Sunny" has availability 51799;
```

## Where Next?
You can find further documentation about migration in our API reference documentation (which is in the */docs* directory of the distribution zip file, and also online [here](https://grakn.ai/javadocs.html). An example of JSON migration using the Java API can be found on [Github](https://github.com/graknlabs/sample-projects/tree/master/example-json-migration-giphy).

{% include links.html %}

## Comments
Want to leave a comment? Visit <a href="https://github.com/graknlabs/docs/issues/32" target="_blank">the issues on Github for this page</a> (you'll need a GitHub account). You are also welcome to contribute to our documentation directly via the "Edit me" button at the top of the page.
