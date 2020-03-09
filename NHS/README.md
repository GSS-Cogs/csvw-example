### NHS Digital: 5 Star Linked Open Data

This document is a walkthrough of how we can represent NHS digital data with csvw, in such a way as it supports conversion to RDF.

To do this we're going to create a csvw representation of the dataset (in compliance with the specification for CSV on the web https://w3c.github.io/csvw/](https://w3c.github.io/csvw/). This CSVW can then be used to generate five star liked open data via any standard compliant tools (for example [https://github.com/Swirrl/csv2rdf](https://github.com/Swirrl/csv2rdf)).

We're approaching this on step by step basis, with the intention to create a learning resource as much as an example. As such, each example is included in it's own self contained directory, while this document will walk through how we've expanded our initial CSVW file with additional detail each time.

Please note: This is intended to be a walkthrough of "what" we're doing, with the technical implementation (i.e "the how") being covered in more detail elsewhere (as technology and solutions will vary from team to team). For further details or to get involved please [Join The Conversation](#join-the-conversation).

This document will also be broken up with additional information sections as needed.

- [Info: Prefixes](#info-prefixes)
- [Example 1: Data Validation](#example-1-data-validation)
- [Info: Observation File Changes](#info-observation-file-changes)
- [Example 2. Codelists as external files](#example-2-codelists-as-external-files)
- [Info: Urls as CSVW](#info-urls-as-csvw)
- [Example 3: Transformation](#example-3-transformation)
- [Example 4: The DSD](#example-4-the-dsd)
- [Example 5: Additional Metadata](#example-5-additional-metadata)
- [Join The Conversation](#join-the-conversation)


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

You can validate using simple types within the datatype field (i.e "number", "string") as well as use an additional `format` field for basic pattern matching (as per the following example: where we are confirming all codes in the Sex column are one of M, F).

```json
"tableSchema": {
  "columns": [
  {
    "titles": "Sex",
    "required": true,
    "name": "sex",
    "datatype": {
       "format": "^(M|F)$"
          }
   },
   {
     "titles": "Age",
     "required": true,
     "name": "age",
     "datatype": "string"
   }]
  }
```

The above pattern matching is done using regular expressions (more information on these can be found at [https://blog.usejournal.com/regular-expressions-a-complete-beginners-tutorial-c7327b9fd8eb](https://blog.usejournal.com/regular-expressions-a-complete-beginners-tutorial-c7327b9fd8eb)).

*Please note - in the provided example1 there's a particularly complicated regex for reference period, this is boiler plate for validating against common RDF expressions of time indicated here: https://github.com/epimorphics/IntervalServer/blob/master/interval-uris.md#uri-set-and-dataset-descriptions.*

There are a few tools you can use to validate your csvw using these datatype entries (and the csvw on the whole). For the COGS project we're using csvlint via a docker image.

To run this locally you need to have [docker](https://docs.docker.com/install/) installed, then:

```
docker run -v /:/workspace -w /workspace gsscogs/csvlint csvlint <PATH_TO_YOUR_SCHEMA_JSON>
```

### Info: Observation File Changes

For our observation file (which I'm calling "combined_source.csv" for this example) we've combined two data sources. 

- Patients Registered at a GP Practice - December 2019: Single year of age (GP practice-females)

- Patients Registered at a GP Practice - December 2019: Single year of age (GP practice-male)

This combined file is present in all examples, but has been compressed due to save space/time (just unzip when working with a given examples).

We've also made a few basic formatting changes to the data (we'll explain the rationale as we go), as follows:

- Sex Column: we're switching Male, Female entries to M, F respectively (to match the common SDMX definition: http://purl.org/linked-data/sdmx/2009/code)

- I've remove `POSTCODE` as this information is linked to the organisation,provided by the `Org Code`.

- I've added a `Measure Type` of `Count` as it's required.

- I've changed `EXTRACT_DATE` to `Period` and reformmatted to `day/2019-Dec-01` as per: https://github.com/epimorphics/IntervalServer/blob/master/interval-uris.md#uri-set-and-dataset-descriptions

- I've rename `NUMBER_OF_PATIENTS` to `Value` and `ONS_CCG_CODE` to `Area` to better match existing linked data resources.

- I've title cased remaining columns to keep the formatting consistent.


### Example 2. Codelists as external files

All columns in a flattened dataset represent items from a list of concepts (or a subset of), these are typically represented by a codelist mapping a code or machine readable name to a human readable label.

One advantage of csvw is you can use it represent more than one csv - allowing you to link directly to a csvs codelists by including them as additional tables in the datasets metadata.

Example snippet:

*Please note, I've generated the example using our base url eg `http://gss-data.org.uk/codelists/sex.csv` as the aim is for these resources to be located on the web, though technically the path to the csv is literally just `sex.csv` at the moment (i.e they're in the same folder as the schema right now)*


```json
"tables": [
{
  "url": "http://gss-data.org.uk/codelists/sex.csv",
  "tableSchema": "https://gss-cogs.github.io/ref_common/codelist-schema.json",
  "suppressOutput": true
},
{
  "url": "http://gss-data.org.uk/codelists/age.csv",
  "tableSchema": "https://gss-cogs.github.io/ref_common/codelist-schema.json",
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
  "columnReference": "age",
  "reference": {
    "resource": "http://gss-data.org.uk/codelists/age.csv",
    "columnReference": "notation"
  }
}]
```

This is actually telling us the following:
- we're importing the `resource` age.csv
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
   "titles": "Org Code",
   "required": true,
   "name": "org_code",
   "datatype": "string",
   "propertyUrl": "http://gss-data.org.uk/def/dimension/org_code",
   "valueUrl": "http://gss-data.org.uk/def/concept/org-code/{org_code}"
}
```

You can see we've added two additional fields:

- propertyUrl: the "thing" that we're describing (in this case an ad hoc dimension).
- valueUrl: the location of the individual codes & labels (collectively, the concepts) that sit within that "thing".

If you consider the `valueUrl`, it is (or can be) just information derived from a codelist.csv such as we've already created.

The `propertyUrl` is broader, in that it's the mechanism to linked shared concepts, even where exact codes do not match up across datasets. In this example it's an ad-hoc definition (ideally, where these are created they are also curated and reused across multiple resources).

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

So even though I'll still need to provide custom age codes via the `valueUrl` I'm able to use a common SDMX propertyUrl for the age dimension (http://purl.org/linked-data/sdmx/2009/dimension#age) - which will create better linkage between my dataset and other other data sets using this property.

In the case of the attached example 3, I've used a mixture of both these approaches.

### Example 4: The DSD

The DSD (Data Structure Definition) is probably the most confusing looking part of the csvw - but - significantly easier to grasp than if looks at first glance.

The principle thing to understand, other than explicitly stating a few things a human could just infer from the earlier example, we're only actually adding a very small amount of additional information.

There are effectively three types of component that make up your linked data cube (`Dimension`, `Measure`, `Attribute`), so *our main goal is to differentiate which of those components each column of our csv actually represents*.

For more information on the differences between these three component types, please see [https://www.w3.org/TR/vocab-data-cube/#cubes-model](https://www.w3.org/TR/vocab-data-cube/#cubes-model).

First, lets's look an example `Dimension` component as it appears in the dsd:

```json
{
  "@id": "http://gss-data.org.uk/sdg-example-dataset/component/age",
  "@type": "qb:ComponentSpecification",
  "qb:componentProperty": {
    "@id": "http://purl.org/linked-data/sdmx/2009/dimension#age"
  },
  "qb:dimension": {
    "@id": "http://purl.org/linked-data/sdmx/2009/dimension#age",
    "@type": "qb:DimensionProperty",
    "rdfs:label": "Age",
    "rdfs:range": {
      "@id": "http://gss-data.org.uk/def/classes/age/age"
    }
  }
}
```

In terms of what we're looking at here:

We need to declare that it is a component (the upper block).
We need to declare that it is a dimension, as well as confirm it's label and give it a range (the lower block).

For both `Attribute` and `Measure` we follow the same pattern switching in `qb:attribute, qb:AttributeProperty` and `qb:measure, qb:MeasureProperty` respectively.

The principle other key thing to be aware of is the RDF datacube specs handling of measure types. Consider the following example:

```json
{
  "@id": "http://gss-data.org.uk/sdg-example-dataset/component/measure_type",
  "@type": "qb:ComponentSpecification",
  "qb:componentProperty": {
    "@id": "http://purl.org/linked-data/cube#measureType"
  },
  "qb:dimension": {
    "@id": "http://purl.org/linked-data/cube#measureType",
    "@type": "qb:DimensionProperty",
    "rdfs:label": "Measure Type",
    "rdfs:range": {
      "@id": "qb:MeasureProperty"
    }
  }
}
```

This indicates that the dimension `Measure Type` has a range of `qb:MeasureProperty`, this is indicative of the specs special handling for measure types.

**If you specify a component dimension as http://purl.org/linked-data/cube#measureType, all measures within that cube are assumed to reside within that dimension.**

These component definitions, combined with some boiler plate gets you to a fully described DSD (please see the included example 4).

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
Joe <http://www.example.org/vocabulary#name> "Joe" .
Joe <http://www.w3.org/1999/02/22-rdf-syntax-ns#Type> <http://www.example.org/vocabulary#Person> .
```

#### Understanding RDF Continuations

Consider this alternate way of writing the same thing:

```

@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
@prefix ex: <http://www.example.org/vocabulary#> .

Joe <http://www.example.org/vocabulary#name> "Joe";
      <http://www.w3.org/1999/02/22-rdf-syntax-ns#> <http://www.example.org/vocabulary#Person> .
```

You'll notice that the second triple has only two statements. That's because the `;` character is indicating that an element of the previous line will be reused (in this case, the variable defined by `Joe`)

These techniques should provide the basics to understand the .trig file included with example 5.


### Join The Conversation

The stated goal of the COGS project is as follows:

```
Revolutionising how UK Government Statistical Data is made available on the Web by creating a repository of 5* Linked Open Data Statistics.
```

This will not be done easily, nor can it be done in isolation, as such the "**C**onnected" in **C**OGS is as much about connecting people and organisations as it is about connecting data (nobody is climbing this mountain alone).

As such we are always actively looking for partners in our joint mission, and new voices to join the conversation around linked open data statistics and the future of government statistics.

So please do get in touch.
