TODO - replace examples and snippets with those specific to SDG, MOJ etc

### SDG: 5 Star Linked Open Data

This document is a walkthrough of how we can create five star linked open data using SDG (sustainable development goals) data.

To do this we're going to create a csvw representation of the dataset (in compliance with the specification for CSV on the web https://w3c.github.io/csvw/](https://w3c.github.io/csvw/). This CSVW will then be used to generate the RDF.

---
- TODO: linkify headings
---
- Info: Prefixes
- Example 1: Data Validation
- Info: Observation File Changes
- Example 2. Codelists as external files
- Info: Urls as CSVW
- Example 3: Transformation
- Example 4: The DSD
- Example 5: Additional Metadata
- Info: Further Information


### Info: Prefixes

For ease of use, csvw uses a system of known prefixes to define fields without cluttering files with endless url references.

For example the field `dc:title` represents `title` as defined by the dc (**d**ublin **c**ore) vocabulary, which can be viewed here: https://www.dublincore.org/specifications/dublin-core/dcmi-terms/

Csvw supports a range of predefined prefixes as it's "initial context", a full list of which can be found here:
https://www.w3.org/2011/rdfa-context/rdfa-1.1. In our example it's the included `@context` key that imports
these prefixes for use.

This is principally just a thing to be aware of, as (providing you include the @context key) the system works without user setup or intervention.

### Example 1. Data Validation

Consider the included example 1.

- Each csv defined by the schema (one at this point) has an entry in `tables`.
- Each `table` in `tables` has a `tableSchema`
- Each `tableSchema` has a number of `columns`
- Each `column` has a `datatype`.

You can validate using simple types within the datatype filed (i.e "number", "string") as well as use an additional `format` field for basic pattern matching (as per the following example: where we are confirming all codes in the Sex column to are either M, F or T).

```json
"tableSchema": {
  "columns": [
    {
      "titles": "Value",
      "required": false,
      "name": "value",
      "datatype": "number"
    },
    {
      "titles": "Sex",
      "required": true,
      "name": "sex",
      "dc:description": "",
      "datatype": {
        "base": "string",
        "format": "^(M|F|T)$"
      }
    }]
  }
```

The above pattern matching is done using regular expressions (more information on these can be found at [https://blog.usejournal.com/regular-expressions-a-complete-beginners-tutorial-c7327b9fd8eb](https://blog.usejournal.com/regular-expressions-a-complete-beginners-tutorial-c7327b9fd8eb)).

There are a few tools you can use to validate your csvw using these datatype entries (and the csvw on the whole). For the COGS project we're using csvlint via a docker image.

To run this locally you need to have [docker](https://docs.docker.com/install/) installed, then:

```
docker run -v /:/workspace -w /workspace gsscogs/csvlint csvlint <PATH_TO_YOUR_SCHEMA_JSON>
```

### Info: Observation File Changes

To make our data transformable we need to make a few basic formatting changes to the data on the observation file (we'll explain the rationale as we go), as follows:

- Sex Column: we're switching Male, Female, "" () entries to M, F, T respectively (to match the common SDMX definition: http://purl.org/linked-data/sdmx/2009/code)

- Age Column: we'll replace the blank entries (denoting all ages) with 'Total'.

- I'm going to change the column name `Time` to the more common (for linked data) `Reference Period`.


### Example 2. Codelists as external files

All columns in a flattened dataset represent items from a list of concepts (or a subset of), these are typically represented by a codelist mapping a code or machine readable name to a human readable label.

One advantage of csvw is you can use it represent more than one csv - allowing you to link directly to a csvs codelists by including them as additional tables in the datasets metadata.

Example snippet (please note, as the example are all local files in the same directory our "urls" are just a filename, usually these would be full urls pointing to real web resources):

```json
"tables": [
{
  "url": "age.csv",
  "tableSchema": "codelist-schema.json",
  "suppressOutput": true
},
{
  "url": "sex.csv",
  "tableSchema": "codelist-schema.json",
  "suppressOutput": true
}]
```
The above snippet shows a csvw entry holding metadata for two csv files. You'll notice the `tableSchema` field is just pointing to another schema (the same schema for both in fact). This is common practice where mutiple csvs have an identical or repeating structure. In the case of our actual observation csv, we'll still be defining the tableSchema in-line, as the metadata structures for datasets very rarely repeated exactly (as individual datasets are unique by definition).

So to include this information for our example there's three tasks:
- define a simple schema for the additional codelist csvs.
- create the additional codelist csvs.
- create the link between the principle csvw schema, and the additional codelist csvs using 'Foreign Keys'

The definition of `Foreign Keys` as provided by [https://www.w3.org/ns/csvw](https://www.w3.org/ns/csvw) is "*Describes relationships between Columns in one or more Tables*."

Consider the following example:

```json
"foreignKeys": [
{
  "columnReference": "Age",
  "reference": {
    "resource": "age.csv",
    "columnReference": "Notation"
  }
}]
```

This is actually telling us the following:
- we're importing the `resource` age.csv (again this would a full url in practice)
- the `Age` column of our observation file (denoted by the first `columnReference` field) ...
- maps to the `Notation` column of the imported resource (as denoted by the **indented** `columnReference` field).


### Info: Urls as CSVW

In this section we're going to de-mystify the semantic "all resources should appear on the web" part of linked open data.

The first thing to note - linked data isn't really about the web, not really. It's all about **name spaces**. Every resource in the world of linked data should be **uniquely identifiable** and transparently available to everyone.

Thankfully, every url on the public internet **is already** uniquely identifiable and available to everyone, so by defining things via csvw (which is really just a user friendly way of defining things via urls) you get both the sharing of definitions and the unique name spacing for free.

To be clear, you *should* also use a url that should tell you a little something about the thing being defined, or at very least you have a place you *could* provide that information, but it's the principle of "definition by public namespace" that underpins linked open data.

**So...how does this work for our csvw example?**

If you think back to example one, we had the field `dc:title`, representing `https://www.dublincore.org/specifications/dublin-core/dcmi-terms#title`, so that field ...*already has* a fully semantically described definition.

**What about those fields without a prefix?**

They don't need one, as the [https://www.w3.org/ns/csvw](https://www.w3.org/ns/csvw) standard is _itself_ a namespace. So where a field doesn't have an prefix (for example `columns`), that just indicates it's defined by eg `https://www.w3.org/ns/csvw#columns`

**Ok, so all that's basically done for me - what about urls for any new resources I need to add? (a column unique to your dataset perhaps?)**

We'll cover this in the next section.


### Example 3: Transformation

Please note: Linked open data needs to inlcude a base url (so, where you're linked data is going to live). For the sake of these example I'm using http://gss-data.org.uk and our standard patterns. You can freely substitute for your own domain and
patterns (or better yet get in contact and use ours!).

Let's look at a `column` entry (as touched on in example 1: validation) that's been extended with this additional information:

```json
{
    "titles": "Observation Status",
    "required": true,
    "name": "observation_status",
    "datatype": "string",
    "propertyUrl": "http://gss-data.org.uk/def/dimension/observation-status",
    "valueUrl": "http://gss-data.org.uk/def/concept/observation-status/{observation_status}"
    },
```

You can see we've added two additional fields:

- propertyUrl: the "thing" that we're describing.
- valueUrl: the location of the individual codes & labels (collectively, the concepts) that sit within that "thing".

If you consider the `valueUrl`, it's literally just information from a codelist.csv we would just make available on the web. Ideally we would reuse them, but the principle point of definition is the `propertyUrl` field.

The `propertyUrl` is broader, in that it's the mechanism to bring in shared definitions. In this example it's an ad-hoc definition (ideally, where these are created they are also curated and reused across multiple resources).

As an alternate approach, let's see about bringing in an pre-existing property definition.

```json
{
    "titles": "Age",
    "required": true,
    "name": "age",
    "datatype": "string",
    "propertyUrl": "http://purl.org/linked-data/sdmx/2009/dimension#age",
    "valueUrl": "http://gss-data.org.uk/def/concept/age/{age}"
    },
```

So even though I need to provide custom age codes via the `valueUrl` I'm still able to use a common SDMX propertyUrl for the age dimension (http://purl.org/linked-data/sdmx/2009/dimension#age) - which will create better linkage between my dataset and other other data sets using this property.

In the case of the attached example 3, I've used a mixture of both these approaches.

### Example 4: The DSD

The DSD (Data Structure Definition) is probably the most confusing looking part of the csvw - but - also probably the simplest to do, consisting as it does of simple repeating patterns.

As such it's a relatively simple thing to automate as (other than explicitly stating a few things a human could just infer from the earlier example) we're only actually adding a small amount of additional information.

There are effectively three types of component that make up your linked data cube (`Dimension`, `Measure`, `Attribute`), so our main goal is to differentiate which of those the columns of our csv actually are.

Lets's look an example `Dimension` component as it appears in the dsd:

```json
{
  "qb:dimension": {
    "@id": "http://purl.org/linked-data/sdmx/2009/dimension#refPeriod",
    "@type": "qb:DimensionProperty",
    "rdfs:label": "Reference Period",
    "rdfs:range": {"@id": "http://www.w3.org/2006/time#Interval"},
    "qb:codeList": {"@id": "http://gss-data.org.uk/def/concept-scheme/refPeriod" }
    }
},
```

### Example 5: Additional Metadata

Anyone that's worked with datasets a lot will already know that their value is often defined by the accompanying metadata (doesn't matter how great your data is if no one can find it).

With linked data cubes there are numerous fields you can use to enhance the metadata of a single RDF data cube, some notable examples include:

```
dct:creator
dct:description
dct:issued
dct:modified
dct:publisher
dct:title
dcat:keyword
dcat:landingPage
```

One of the advantages of linked data is that it's almost infinitely extendable (you want to describe further aspects of something? add more triples). So in the case of linked data cubes, our current best practice is to create supplementary metadata fields of the sort listed above as a `.trig` file (a trig file is just a simple text file for representing graph/rdf data, see [https://en.wikipedia.org/wiki/TriG_(syntax)](https://en.wikipedia.org/wiki/TriG_(syntax)) for additional details).

The writing of RDF is a little beyond the scope of these examples, but we'll touch briefly on two main principles.

#### Understanding Prefixing

Consider this snippet:

```rdf
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
@prefix ex: <http://www.example.org/vocabulary#> .

Joe ex:name "Joe"
Joe rdf:Type ex:Person
```

The main thing to understand is the prefixes that are (always) included at the top of the file. If you substritute them in (for rdf and ex repsectively) you get two basic RDF triples.

```RDF
Mike <http://www.example.org/vocabulary#name> "Mike" .
Mike <http://www.w3.org/1999/02/22-rdf-syntax-ns#Type> <http://www.example.org/vocabulary#Person> .
```

#### Understanding Continuations (TODO - not the right word, whats the real one?)

Consider this alternate way of writing the same thing:

```

@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
@prefix ex: <http://www.example.org/vocabulary#> .

Joe <http://www.example.org/vocabulary#name> "Joe";
      <http://www.w3.org/1999/02/22-rdf-syntax-ns#> <http://www.example.org/vocabulary#Person> .
```

You'll notice that the second triple has only two statements. That's because the `;` character is indicating that an element of the previous line will be reused (in this case, the variable defined by `Joe`)

These techniques should provide the basics to understand the .trig file included in example 5.

### Info: Further Information

---
- TODO: useful or waffle? confusing either way, cannibalise or remove.
---


#### The Golden Rule

Everything in linked data world will eventually becomes a url - so **make your column values url friendly**. This principally means don't use special characters or spaces.

..but....my label has spaces in it?!

Remember the codelists we've previously defined? (see example 2). Those codelists have two main columns:

- 1.) The Code - the url friendly, no spaces, no special characters representation of a thing.

- 2.) The Label - where you can do whatever you want to make a nice readable plain english label for us humans.

So just remember: **the code goes in the observation file, then you add each codes user friendly label via the codelist** and you can't go far wrong.

#### Common problems and things that are good to known

There's quite a few solved problems (and ongoing discussions) that'll inevitably crop up for everyone, so we'll finish with a bit of a knowledge share:


**Time.** Everyone has to include details on time, and traditionally nobody knows quite how to express it. Thankfully in five star linked open data world we have [https://github.com/epimorphics/IntervalServer/blob/master/interval-uris.md](https://github.com/epimorphics/IntervalServer/blob/master/interval-uris.md) as a common reference point in regards to this.

By way of some quick examples:

| Time as plain text | The same time as a url  |
|-----|-----------|
| 2005   | http://reference.data.gov.uk/id/year/2005
| 2006 Q2 | http://reference.data.gov.uk/id/quarter/2006-Q2
| A three month interval, starting on 1st February 2004  | http://reference.data.gov.uk/id/gregorian-interval/2004-02-01T00:00:00/P3M


**Sex.** There is an old and very commonly used SDMX definition of sex at [http://purl.org/linked-data/sdmx/2009/code](http://purl.org/linked-data/sdmx/2009/code) that uses M, F, T (Male, Female, Total).

Given the commonality and wide spread use of this reference, we would recommend converting your observation file to use use the M,F,T codes wherever possible.

**Age.** is an ongoing discussion (as everyone seems to handle it slightly differently). At a minimum make sure there are no spaces in your codes for age, beyond that please do contact us@gsscogs.uk and get involved in that discussion (we genuinely want to hear from you!)

**Measure Type.** For the cogs project we’ve been laying things out in CSV with multiple rows for each measure, by including a `Measure Type` column in each observation csv.

So if (for example) you were measuring both "mass" and "volume" for a given combination of dimensions, you'd include each on it's own row and differentiate via literally recording either "mass" or "volume" in the `Measure Type` column.

**Unit Multiplier**

Where a unit multipler is required to express the data, a `Unit` column needs to be included in the observation csv.

This allows the creation of (again, SDMX based) URIs of the form of http://purl.org/linked-data/sdmx/2009/code#unitMult-0(where the last bit is replaced with the unit multiplier, i.e. 10 to the power of).

It's just that final number that you'd need to include in the observation csv.
