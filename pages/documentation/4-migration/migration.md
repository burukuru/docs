---
title: Migrating to Grakn
keywords: setup, getting started
last_updated: August 10, 2016
tags: [getting-started, graql, migration]
summary: "Landing page for documentation about loading data in different formats to populate a graph in Grakn."
sidebar: documentation_sidebar
permalink: /documentation/migration/migration.html
folder: documentation
comment_issue_id: 32
---

## Introduction
This page introduces migration of data, stored in different formats, to populate a graph in Grakn. If you have not yet set up the Grakn environment, please see the [setup guide](../get-started/setup-guide.html).

## Migration Shell Script
The migration shell script can be found in `mindmaps-dist/bin` after it has been unzipped. Usage is specific to the type of migration being performed:

### SQL Migration

```bash
usage: ./migration.sh sql -driver <jdbcDriver> -user <username> -pass <password> -database <url> -graph <graphname> [engine <url>]
       -driver           JDBC driver
       -user             username for SQL database
       -pass             password for SQL database
       -database         URL to SQL database
       -graph            graph name (or defaults to the default keyspace)
       -engine           Grakn engine URL, default localhost
```

Please see the [SQL migration documentation](./SQL-migration.html) for further information.

### CSV Migration

```bash
usage: ./migration.sh csv -template <arg> -input <arg> [-help] [-no] [-separator <arg>] [-batch <arg>] [-uri <arg>] [-keyspace <arg>]

OPTIONS
   -b,--batch <arg>       number of row to load at once
   -d,--delimiter <arg>   delimiter of columns in input file
   -f,--file <arg>        csv file
   -h,--help              print usage message
   -k,--keyspace <arg>    keyspace to use
   -t,--template <arg>    graql template to apply over data
   -u,--uri <arg>         uri to engine endpoint
```

Please see the [CSV migration documentation](./CSV-migration.html) for further information.

### JSON Migration

```bash
usage: migration.sh json -template <arg> -input <arg> [-help] [-no] [-batch <arg>] [-uri <arg>] [-keyspace <arg>]
 -b,--batch <arg>      number of row to load at once
 -h,--help             print usage message
 -i,--input <arg>      input json data file or directory
 -k,--keyspace <arg>   keyspace to use
 -n,--no               dry run- write to standard out
 -t,--template <arg>   graql template to apply over data
 -u,--uri <arg>        uri to engine endpoint
```

### Export 

```bash
usage: migration.sh export -data -ontology [-help] [-no] [-batch <arg>] [-uri <arg>] [-keyspace <arg>]
 -data                 export data
 -ontology             export ontology
 -h,--help             print usage message
 -k,--keyspace <arg>   keyspace to use
 -n,--no               dry run- write to standard out
 -u,--uri <arg>        uri to engine endpoint
```

Export will always redirect to standard out. 

### OWL Migration

```bash
usage: migration.sh owl -input <arg> [-help] [-no] [-batch <arg>] [-uri <arg>] [-keyspace <arg>]
 -h,--help             print usage message
 -i,--input <arg>      input owl file
 -k,--keyspace <arg>   keyspace to use
 -n,--no               dry run- write to standard out
 -u,--uri <arg>        uri to engine endpoint
```

Please see the [OWL migration documentation](./OWL-migration.html) for further information.

NOTE: `-no` is not supported by OWL migration at the moment

## Where Next?
You can find further documentation about migration in our API reference documentation (which is in the `/docs` directory of the distribution zip file, and also online [here](https://grakn.ai/pages/api-reference/latest/index.html).

Please take a look at our examples to further illustrate [SQL migration](../examples/SQL-migration.html) and [OWL migration](../examples/OWL-migration.html).
{% include links.html %}


## Comments
Want to leave a comment? Visit <a href="https://github.com/graknlabs/docs/issues/32" target="_blank">the issues on Github for this page</a> (you'll need a GitHub account). You are also welcome to contribute to our documentation directly via the "Edit me" button at the top of the page.