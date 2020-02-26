## SDG CSVW Examples

The idea behind csvw is that all metadata fields (and all the columns of the csv) are fully described using web based vocabularies see [https://www.w3.org/standards/semanticweb/ontology](https://www.w3.org/standards/semanticweb/ontology) for an overview of linked data vocabularies).

For the cogs project, we've successfully explored taking this as a step further, using these fully described csvw files as a direct (and relatively straight forward) entry point to the world of five start linked open data.

The following is a walk through of how we've been doing this. For the sake of practicality, I've built towards this via five discreet steps/examples, with each example expanding on the previous.



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

You can use a `format` field within those `dataType` entry to create validation statements. Example follows:

```json
"tables": [
  {
    "url": "indicator_1-2-1.csv",
    "tableSchema": {
      "columns": [
        {
          "titles": "Year",
          "required": true,
          "name": "year",
          "datatype": "integer"
        }, {
          "titles": "Sex",
          "required": false,
          "name": "sex",
          "datatype": {
            "base": "string",
            "format": "^(Male|Female)$"
          }
        ]}
      }
    ]
```

The datatype can be a simple variables type (integer, string) and (in the case of strings) also include a `format` field, allowing you to pattern match using regular expressions (more information on these can be found at [https://blog.usejournal.com/regular-expressions-a-complete-beginners-tutorial-c7327b9fd8eb](https://blog.usejournal.com/regular-expressions-a-complete-beginners-tutorial-c7327b9fd8eb)).

There are a few tools you can use to validate your csvw using these datatype entries (and the csvw on the whole). For the COGS project we're using csvlint via a docker image.

To run this locally you need to have [docker](https://docs.docker.com/install/) installed, then:

```
docker run -v /:/workspace -w /workspace gsscogs/csvlint csvlint <PATH_TO_YOUR_SCHEMA_JSON>
```

### Example 2. Codelists as external files

All columns in a flattened dataset represent items from a list of concepts (or a subset of), these are typically represented by a codelist mapping a code or machine readable name to a human readable label.

One advantage of csvw is you can use it represent more than one csv - allowing you to link directly to a csvs codelists by including them as additional tables in the datasets metadata.

Example:

```json
tables: [
{
url: "https://gss-cogs.github.io/ref_alcohol/codelists/underlying-cause-of-death.csv",
tableSchema: "https://gss-cogs.github.io/ref_common/codelist-schema.json",
suppressOutput: true
},
{
url: "https://gss-cogs.github.io/ref_alcohol/codelists/health-social-care-trusts.csv",
tableSchema: "https://gss-cogs.github.io/ref_common/codelist-schema.json",
suppressOutput: true
},
```
The above snippet shows a csvw entry holding metadata for two csv files. You'll notice the `tableSchema` field is just pointing to another schema (the same schema for both in fact). This is common practice where mutiple csvs have an identical or repeating structure. In the case of our actual observation csv, we'll still be defining the tableSchema in-line, as the metadata structures for datasets very rarely repeated exactly (as individual datasets are unique by definition).

The above example is a small part of a COGS schema I've cut out illustrate the point, you can view this schema in full at [https://ci.floop.org.uk/job/GSS_data/job/Health/job/NISRA_alcohol_deaths/lastSuccessfulBuild/artifact/out/observations.csv-schema.json](https://ci.floop.org.uk/job/GSS_data/job/Health/job/NISRA_alcohol_deaths/lastSuccessfulBuild/artifact/out/observations.csv-schema.json).

*note - suppressOutput is a boolean switch that hides output when transforming the contents of said table (this will be relevant later). A description of this and other csvw namespaced terms can be found at [https://www.w3.org/ns/csvw](https://www.w3.org/ns/csvw).*

So to include this information for our example there's two tasks:
- define a simple schema for the additional codelist csvs.
- add the additional codelist csvs.

For the sake of this example I've included these in `/resources` within the `example2` repo, but in actuality you'd aim to host them somewhere directly accessible online (so you can easily reuse schemas etc across multiple datasets).

### -----
### ----- TODO: explain foreign key references ----
### -----  

### Info: Urls as CSVW

In this section we're going to de-mystify the semantic "all resources should appear on the web" part of linked open data. The goal here is simple - if you establish the relevant links via your csvw metadata, you've created a resource that can be used to convert csv data directly to RDF triples in a deterministic and repeatable fashion.

The first thing to note - linked data isn't really about the web, not really. It's all about **name spaces**. Every resource in the world of linked data should be **uniquely identifiable** and transparently available to everyone.

Thankfully, every url on the public internet **is already** uniquely identifiable and available to everyone, so by defining things via csvw (which is really just a user friendly way of defining things via urls) you get both the sharing of definitions and the unique name spacing for free.

To be clear, you *should* also use a url that should tell you a little something about the thing being defined, or at very least you have a place you *could* provide that information, but it's the principle of "definition by public namespace" that underpins linked open data.

**So...how does this work for our csvw example?**

If you think back to example one, we had the field `dc:title`, representing `https://www.dublincore.org/specifications/dublin-core/dcmi-terms#title`, so that field ...*already has* a fully semantically described definition.

**What about those fields without a prefix?**

They don't need one, as the [https://www.w3.org/ns/csvw](https://www.w3.org/ns/csvw) standard is _itself_ a namespace. So where a field doesn't have an prefix (for example `columns`), that just indicates it's defined by eg `https://www.w3.org/ns/csvw#columns`

**Ok, so all that's basically done for me - what about urls for any new resources I need to add? (a column unique to your dataset perhaps?)**

As part of the COGS project we're creating these resources using the codelists csvs (which we've already linked to via the csvw - see example 2) which gives us the information we need to generate a large part of the semantic resources (codes, labels and their definitions) needed for our dataset.

Think of what we'll have at this point:
- we have a csv represention of each codelist
- we have the `foreign key` entry linking each of these csv codelists to a column in our observation file.
- we have a the base url of whichever domain we intend to host these web resources on (if you don't and you're producing government statistical data then contact us and we'll gladly do that for you).

Taken together, the above is all the information required to generate any supporting reference data your datasets would require (for example `www.my-organisations-domain.gov.uk/data/my-codelist/code1`) directly from your csvw.

I won't go into the technical implementation here (but please do contact us at us@gsscogs.uk you want to discuss) but to repeat how we opened this section **if you establish the relevant links via your csvw metadata, you've created a resource that can be used to convert csv data directly to RDF triples in a deterministic and repeatable fashion**.

And if you're following along, you already have!


### Example 3: Conversion to Linked Data

So we've touched on codelists and how we can go from csvw to RDf, but what about the dataset itself? Where and how do we defined the "linked" in linked data? Thos shared definitions that lie behind each thing we're describing?

### -----
### ----- TODO: explain the mechanics of dimensional url references via csvw ----
### -----  


### Example 4: Additional Metadata

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

*By now, I suspect you'll recognise our old friend the prefix appearing again.*

One of the advantages of linked data is that it's almost infinitely extendable (you want to describe further aspects of something? add more triples). So in the case of linked data cubes, our current best practice is to create supplementary metadata fields of the sort listed above as a `.trig` file (a trig file is just a simple text file for representing graph/rdf data, see [https://en.wikipedia.org/wiki/TriG_(syntax)](https://en.wikipedia.org/wiki/TriG_(syntax)) for additional details).

Again, I'm not going to go into the full technical implementation (and again, happy to if anyone wants to get in contact) but you're basically aiming to write a text file and upload alongside your data cube - as long as you include relevant fields (if unsure, see the included example 3) you can generally pick whatever method works best with your current processes.

### Example 5: Formatting the Observation file

We've spoken previously about how we can convert the whole observation file to linked open data (which is true), but what we haven't spoken about yet, is how this goal will effect the required structure and formatting of that observation file (don't worry, it's pretty minimal).

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
