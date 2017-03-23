---
title: Grakn Setup Guide
keywords: setup, getting started
last_updated: August 10, 2016
tags: [getting-started, graql]
summary: "This document will teach you how to set up a Grakn environment, start it up and load a simple example."
sidebar: documentation_sidebar
permalink: /documentation/get-started/setup-guide.html
folder: documentation
comment_issue_id: 16
---


## First: Download GRAKN.AI

[![download](/images/download.png)](https://grakn.ai/download/latest)

For more information on how to download older versions of GRAKN.AI, compile from source code, or import the Grakn Java API library as a development dependency, please visit our [Downloads page](../resources/downloads.html).

## Install GRAKN.AI
{% include note.html content="**Prerequisites**   <br />
GRAKN.AI requires Java 8 (Standard Edition) with the `$JAVA_HOME` set accordingly. If you don't already have this installed, you can find it [here](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html).  
If you intend to build Grakn from source code, or develop on top of it, you will also need Maven 3." %}

Unzip the download into your preferred location and run the following in the terminal to start Grakn:

```bash
cd [your Grakn install directory]
./bin/grakn.sh start
```

This will start:

* an instance of Cassandra, which serves as the supported backend for Grakn.
* Grakn Engine, which is an HTTP server providing batch loading, monitoring and the browser dashboard.

{% include note.html content="**Useful commands**  <br />
To start Grakn, run `grakn.sh start`.   
To stop Grakn, run `grakn.sh stop`.    
To remove all graphs from Grakn, run `grakn.sh clean`" %}

Grakn Engine is configured by default to use port 4567, but this can be changed in the *grakn-engine.properties* file, found within the */conf* directory of the installation.

## Test the Graql Shell

To test that the installation is working correctly, we will load a simple ontology and some data from a file, *basic-genealogy.gql* and test it in the Graql shell and Grakn Visualiser. The *basic-genealogy.gql* file will be included in the */examples* folder of the Grakn installation zip from release 0.11.0 and onwards. You can also download it from the [Grakn repo on Github](https://github.com/graknlabs/grakn/blob/master/grakn-dist/src/examples/basic-genealogy.gql). In the code below, we assume that the file is in the */examples* folder. 

Type in the following to load the example graph. This starts the Graql shell in non-interactive mode, loading the specified file and exiting after the load is complete.

```bash
./bin/graql.sh -f ./examples/basic-genealogy.gql
```

Then type the following to start the Graql shell in its interactive (REPL) mode, type:

```bash
./bin/graql.sh
```

The Graql shell starts and you see a `>>>` prompt. Graql is our knowledge-oriented query language, which allows you to interface with Grakn. We will type in a query to check that everything is working. 


```graql   
match $x isa person, has identifier $n;
```

You should see a printout of a number of lines of text, each of which includes a name, such as "William Sanford Titus" or "Elizabeth Niesz".

If you see the above output then congratulations! You have set up Grakn.

## Test the Visualiser

The [Grakn visualiser](../grakn-dashboard/visualiser.html) provides a graphical tool to inspect and query your graph data. You can open the visualiser by navigating to [localhost:4567](http://localhost:4567) in your web browser. The visualiser allows you to make queries or simply browse the knowledge ontology within the graph. The screenshot below shows a basic query (`match $x isa person; offset 0; limit 100`) typed into the form at the top of the main pane, and visualised by pressing ">" to submit the query:

![Person query](/images/match-$x-isa-person.png)

You can zoom the display in and out, and move the nodes around for better visibility. Please see our [Grakn visualiser](../grakn-dashboard/visualiser.html) documentation for further details.

### Troubleshooting  
If you are having trouble getting Grakn running, please check our [FAQ page](../resources/faq.html), and if you have any questions, do ask them on our [discussion forum](http://discuss.grakn.ai), on [Stack Overflow](http://stackoverflow.com) or on our [Slack channel](https://grakn.ai/slack.html).


## Where next?
Our [Quickstart Tutorial](./quickstart-tutorial.html) will go into more detail about using Grakn and Graql.

You can find additional example code and documentation on this portal. We are always adding more and welcome ideas and improvement suggestions. [Please get in touch](https://grakn.ai/community.html)!

{% include links.html %}

## Comments
Want to leave a comment? Visit <a href="https://github.com/graknlabs/docs/issues/16" target="_blank">the issues on Github for this page</a> (you'll need a GitHub account). You are also welcome to contribute to our documentation directly via the "Edit me" button at the top of the page.
