---
title: Graql Templating
keywords: graql, java
last_updated: November 4, 2016
tags: [graql]
summary: "How to use templating on top of Graql"
sidebar: documentation_sidebar
permalink: /documentation/graql/graql-templating.html
folder: documentation
---

## Basic Syntax
 
Graql templates are Graql statements with the addition of functions and variables. Graql templates are primarily used for migrating data into Grakn, although as a component of Graql, they can be used whevever writing Graql in Java. You can find more examples of their usage in the [Migration](../migration/migration-overview.html) section of the documentation. 

{% include note.html content=" For the moment, Graql Template usage is limited to Graql Java. It will not work in the Graql shell. " %}

Templates are used to expand Graql queries, given data. They allow you to control the flow of information.

Accessing a single value:

```graql-template
$x isa thing has val <value>
```

Value in a nested context:

```graql-template
$x isa person has name <person.name.firstName>
```

## Replacement

Replacement occurs inside the `<` `>` characters or when executing a macro that is not a condition. 

{% include note.html content=" When replacing values in the output, the templating engine will automatically surround any `string` in quotes. It will not for any `int`, `double` or `boolean` " %}

A quick example of what the replacement looks like:

```graql-template
first is a <string>, second a <long>, third a <double>, fourth a <boolean>
```

```text
first is a "string", second a 40, third a 0.001, fourth a false
```

## Logic 

### Expressions

Expressions can be used as the conditions in `if`/`elseif` statements or as the contents of `macro` statements. 

#### Boolean expressions

Boolean expressions operate over `true` and `false`.

|  Expression | Usage | 
|-------------|---| --- |
| `and`       |  `if (<this> and <that>) do { ... }` |
| `or`        |  `if (<this> or <that>) do { ... }` |
| `not`       |  `if (not <this>) do { ... }` | 

#### Comparisons

|  Expression | Usage | Notes
|-------------|---| --- |
| `=`         |  `if (<this> = <that>) do { ... }` |
| `!=`        |  `if (<this> != <that>) do { ... }` |
| `>`         |  `if (<this> > <that>) do { ... }` | operates on numbers
| `>=`        |  `if (<this> >= <that>) do { ... }` | operates on numbers
| `<`         |  `if (<this> < <that>) do { ... }` | operates on numbers
| `<=`        |  `if (<this> <= <that>) do { ... }` | operates on numbers

### Conditionals 

`if`, `else` and `elseif` are the included commands that provide conditional logic. 

**`if` statements**:

```graql-template
if (<surname> != null)
do { insert $x has surname <surname>; }
```

**`if`...`else`**

```graql-template
if ( <born> != null) 
do { insert $x has birth-date <born>; } 
else { insert $x; }
```

#### Groups

Parenthesis can be used to group conditionals together. 

```graql-template
if( (first <= second) or (not (third <= second))) 
do { insert isa y; } 
else { insert isa z; }
```

### Iteration

Graql Templates allow you to iterate over maps or lists.

**For loop over a list**

```graql-template
insert
    for (name in <names>)
    do { 
        $x has name <name>; 
    }
```

**For loop over a map**

```graql-template
insert $x isa person;
    for (address in <addresses>) do {
        $x has street <address.street>
    };
```

**Enhanced `for` loop over a map** In this type of loop it is not required to provide the item name. The properties within the `do` block context are inferred to be the first level children of the property in the `for` statement.

```graql-template
insert $x isa person;
    for (<addresses>) do {
        $x has street <street> ;
    }
```

**Doubly nested `for`**

```graql-template
insert

for (<people>) do { 
$x isa person has name <name>;
    for (<addresses>) do {
    $y isa address ;
        $y has street <street> ;
        $y has number <number> ;
        ($x, $y) isa resides;
    }
}
```

### Macros

Macros are denoted by an `@` symbol prefixing the name of the macro function. 

**`noescp`**  is short for "no escape". This function will not add quotes or escape the characters inside the value when doing replacement. Exactly one argument accepted. Returns a string.

```graql-template
insert $x isa @noescp(<species>)-species;
```

**`int`** converts the contents of the data to an integer. Exactly one argument accepted. 

```graql-template
match $x isa thing has val @int(<value>);
```

**`long`** converts the contents of the data to an long. Exactly one argument accepted. 

```graql-template
match $x isa thing has val @long(<value>);
```

**`double`** converts the contents of the data to a double. Exactly one argument accepted. 

```graql-template
match $x isa thing has val @double(<value>);
```

**`boolean`** converts the contents of the data to a boolean. Exactly one argument accepted. 

```graql-template
match $x isa thing has val @boolean(<value>);
```

**`equals`** returns a boolean with a value specified by the gievn string. The boolean represents a `true` value if the string argumetn is not null and is equal, ignoring case, to the string "true". Can be used in conditional statements. Requires at least two arguments. 

```graql-template
insert $x isa thing val @equals(<this>, <that>, <other>)
```

```graql-template
if (@equals(<this>, <that>)) do { equals }"
```

**`date`** `(<value>, fromFormat, toFormat)` converts a date string from the given format to another format. If the third argument is missing, converts to epoch time. Date format specifications can be found [here](https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html). Returns a string. 

```graql-template
insert $x val @date(<date>, "mm/dd/yyyy", "dd/mm/yyyy");
```

```graql-template
insert $x val @date(<date>, "mm/dd/yyyy");
```

**`lower`** converts the contents of the data to lower case. 

```graql-template
match $x isa thing has val @lower(<value>);
```

**`upper`** converts the contents of the data to upper case.

```graql-template
match $x isa thing has val @upper(<value>);
```

**`split`** splits the given string around the matches of the given regular expression. More information about regular expressions can be found [here](https://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html#sum). Returns a list of strings.

```graql-template
insert $x 
    for (val in @split(<list>, ",")) do { 
        has description <val>
    };
```

**`concat`** concatenates all of the given arguments into a single string. If the arguments are not strings, it converts them to strings before concatenating. Returns a string.

```graql-template
insert $x has val @concat(<forname>, " ", <surname>);
```

#### Nesting macros

When writing a template, you can nest macros inside other macros. When doing so, you're usnig the results of the nested macros as arguments to the enclosing ones. 

For example, the `date` macro returns a string, but many people will want to convert to epoch time and store the value as a long. If that is the case you can:

```graql-template
insert $x val @long(@date(<date> "mm/dd/yyyy"));
```

#### User-defined Macros

The described macros are built-in to Graql templating language. Grakn provides an interface that a user should extend in java to implement custom macros. The user should then register the created macro with the `QueryBuilder` class. 

### Scopes

During iteration Graql variables will be automatically suffixed with an index.  

For example, the following loop:

```graql-template
insert
	$p isa person has identifier <pid>
		has firstname <name1>,
		
		if (<surname> != "") do 
		{
		has surname <surname>,
		}

		if (<name2> != "") do 
		{
		has middlename <name2>,
		}
		
		if (<age> != "") do 
		{
		has age @long(<age>),
		}

		if (<born> != "") do 
		{
		has birth-date <born>,
		}

		if (<dead> != "") do 
		{
		has death-date <dead>,
		}

		has gender <gender>;
```

would result in the expanded Graql queries:

```graql
...
insert $p0 isa person has firstname "Barbara" has identifier "Barbara Newman" has surname "Newman" has gender "female";
insert $p0 has birth-date "1811-03-06" isa person has surname "Newman" has gender "male" has death-date "1898-09-10" has identifier "Henry Newman" has age 87 has firstname "Henry";
... 
```


## Comments
Want to leave a comment? Visit <a href="https://github.com/graknlabs/docs/issues/42" target="_blank">the issues on Github for this page</a> (you'll need a GitHub account). You are also welcome to contribute to our documentation directly via the "Edit me" button at the top of the page.

{% include links.html %}

