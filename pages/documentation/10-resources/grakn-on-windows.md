---
title: Using GRAKN.AI on Windows
keywords: setup, getting started, download
last_updated: December 2016
tags: [getting-started]
summary: "A guide to setting up GRAKN.AI on Windows."
sidebar: documentation_sidebar
permalink: /documentation/resources/grakn-on-windows.html
folder: documentation
comment_issue_id: 25
---

## Instructions
Grakn needs Java 8 and Cassandra 2.2.x (http://cassandra.apache.org/) at the moment. Our distribution comes bundled with Cassandra and we start everything through Unix shell scripts which haven’t been ported to Windows yet. So our wrapped up setup does not work out of the box on Windows.  

From a fresh Windows 10 install:

1. Install Java 8 and make sure the JAVA_HOME environment variable is pointing to its location.
2. Install Cassandra 2.2.8 from http://www.apache.org/dyn/closer.lua/cassandra/2.2.8/apache-cassandra-2.2.8-bin.tar.gz as follows:
 
  - Unzip somewhere
  - In CASSANDRA_HOME\bin, I modified the cassandra.ps1 script (PowerShell script) to include the JAVA_HOME and JAVA_BIN environment variables in the main function as follows:

```
Function Main
{
    ValidateArguments
    $env:JAVA_HOME='C:\Program Files\Java\jre1.8.0_101'
    $env:JAVA_BIN='C:\Program Files\Java\jre1.8.0_101\bin\java.exe'
```

Sadly, Cassandra’s own startup scripts did not work out of the box for Windows due to the missing JAVA_BIN (JAVA_HOME I added just for good measure :). 

3. Start Cassandra by running “CASSANDRA_HOME\bin\cassandra.bat -f” from a command prompt. 

4. Modify CASSANDRA_HOME\bin\nodetool.bat to include the same environment variables:

```
set JAVA_HOME=C:\Program Files\Java\jre1.8.0_101
set JAVA_BIN=%JAVA_HOME%\bin\java.exe
```

5.  Run “CASSANDRA_HOME\bin\nodetool enablethrift” in another command prompt, while Cassandra is currently running. Note that every time you restart Cassandra, you need to rerun that ‘nodetool enablethrift’ command. 

With this, Cassandra should be ready for Grakn.

6. [Download](https://grakn.ai/pages/documentation/get-started/setup-guide.html) and unzip Grakn.

7. Rename the attached file to a ‘.zip’ and unzip into the GRAKN_HOME/bin directory. It contains a few simplified command line scripts just to be able to run things and kill them with Ctrl-C from the console.

<grakncmd.zpbin>

8. In a command prompt, run GRAKN_HOME/bin/grakn.cmd

9. Go to http://localhost:4567 to make sure it’s running

## Comments
Want to leave a comment? Visit <a href="https://github.com/graknlabs/docs/issues/25" target="_blank">the issues on Github for this page</a> (you'll need a GitHub account). You are also welcome to contribute to our documentation directly via the "Edit me" button at the top of the page.

{% include links.html %}