# CSVW Maturity Levels

The following document is walkthrough of using csvw and the maturity levels of it's adoption. This guide is broken down into two principle section:

1.) A walkthrough of the different levels of maturity you can reach while using csvw.

2.) Optional supplementary material that will be referenced from the main text at appropriate places. 



- [CSVW Maturity Levels](#csvw-maturity-levels)
  - [Level 0: Tidy CSV with no JSON](#level-0-tidy-csv-with-no-json)
  - [Level 1: The addition of dataset metadata as JSON](#level-1-the-addition-of-dataset-metadata-as-json)
  - [Level 2: The addition of a table schema](#level-2-the-addition-of-a-table-schema)
  - [Level 3: The addition of external tables, foreign keys and primary keys](#level-3-the-addition-of-external-tables-foreign-keys-and-primary-keys)
  - [Level 4: The addition of the transform definition](#level-4-the-addition-of-the-transform-definition)
    - [Defining Properties and Values](#defining-properties-and-values)
      - [propertyUrl](#propertyurl)
    - [Defining an 'about' url](#defining-an-about-url)
  - [Level 5: The addition of a dataset structure definition.](#level-5-the-addition-of-a-dataset-structure-definition)
- [Supplementary](#supplementary)
  - [REMOVE OR USE](#remove-or-use)
  - [A brief note on urls](#a-brief-note-on-urls)
  - [CSVW Validation](#csvw-validation)
  - [Output Tooling](#output-tooling)


## Level 0: Tidy CSV with no JSON

A "tidy" csv is a version of data that has been rendered flat and into a a-one-observation per line form.

So this untidy data...

![not tidy](images/nottidy.png)

Looks like this as a tidy data

![tidy](images/tidy.png)

While this doesn't provide supporting metadata, it does make the dataset easily digestible by machines and should be considered a crucial initial step.


## Level 1: The addition of dataset metadata as JSON

For ease of use, csvw uses a system of known prefixes to define fields without cluttering files with endless url references.

The csvw spec makes use of several pertinent vocabularies as shown below:


| Prefix | Namespace | Description |
| ------------- | ------------- | ------------- |
| csvw:  | [http://www.w3.org/ns/csvw#](http://www.w3.org/ns/csvw#) | The principle csvw vocabulary, provides the requried definitions to define a data table.
| rdf:  | [http://www.w3.org/1999/02/22-rdf-syntax-ns#](http://www.w3.org/1999/02/22-rdf-syntax-ns#) | The RDF Schema.
| rdfs: | [http://www.w3.org/2000/01/rdf-schema#](http://www.w3.org/2000/01/rdf-schema#) | The RDF vocabulary (which makes use of the schema - see above)
| xsd: | [http://www.w3.org/2001/XMLSchema#](http://www.w3.org/2001/XMLSchema#) | The XML Schema namespace
| dc: | [http://purl.org/dc/terms/](http://purl.org/dc/terms/) | Specification of all metadata terms maintained by the Dublin Core Metadata Initiative. Includes an extensive metadata vocabulary.
| dcat: | [http://www.w3.org/ns/dcat#](http://www.w3.org/ns/dcat#) | The Data Catalogue vocabulary.
| prov: | [http://www.w3.org/ns/prov#](http://www.w3.org/ns/prov#) | Supports the interchange of provenance on the web.

This use of these prefixes allows you to easily express semantically defined metadata via your json file, for example:

```json
{
  "dc:title": "Trade in goods",
  "dc:issued": {
      "@value": "2020-04-08",
      "@type": "http://www.w3.org/2001/XMLSchema#date"
    },
}
```

Allows you to express the title, the data issued (and the schema the date is using) in a relatively terse way.

eg `dc:title` resolves to [http://purl.org/dc/terms/title](http://purl.org/dc/terms/title) which will lead you to a specific (and public) definition of the exact meaning of the word `title` in this context.

The following is more extensive example of using csvw to capture metadata about a dataset (please note - the `url` field is just indicating the location of the csv table being defined - in this case "observations.csv" and in the same location).

```json

{
  "@context": [
    "http://www.w3.org/ns/csvw",
    {
      "@language": "en"
    }
  ],
  "tables": [
    {
      "url": "observations.csv"
    }
  ],
  "@id": "http://gss-data.org.uk/data/gss_data/trade/ons-trade-in-goods#tablegroup",
  "prov:hadDerivation": {
    "@id": "http://gss-data.org.uk/data/gss_data/trade/ons-trade-in-goods",
    "@type": "dcat:Dataset",
    "dc:publisher": {
      "@id": "https://www.gov.uk/government/organisations/office-for-national-statistics"
    },
    "dc:title": "Trade in goods: country-by-commodity imports and exports",
    "dcat:landingPage": {
      "@id": "https://www.ons.gov.uk/economy/nationalaccounts/balanceofpayments/datasets/uktradecountrybycommodityimports"
    },
    "rdfs:label": "Trade in goods: country-by-commodity imports and exports",
    "dc:creator": {
      "@id": "https://www.gov.uk/government/organisations/office-for-national-statistics"
    },
    "dc:issued": {
      "@value": "2020-04-08",
      "@type": "http://www.w3.org/2001/XMLSchema#date"
    },
    "dc:license": {
      "@id": "http://www.nationalarchives.gov.uk/doc/open-government-licence/version/3/"
    },
    "dc:modified": {
      "@value": "2020-04-17T15:09:09.835868+00:00",
      "@type": "http://www.w3.org/2001/XMLSchema#dateTime"
    },
    "dc:description": {
      "@value": "Monthly import country-by-commodity data on the UK's trade in goods, including trade by all countries and selected commodities, non-seasonally adjusted.",
      "@type": "https://www.w3.org/ns/iana/media-types/text/markdown#Resource"
    },
    "dcat:contactPoint": {
      "@id": "mailto:trade@ons.gov.uk"
    },
    "dcat:theme": {
      "@id": "http://gss-data.org.uk/def/concept/statistics-authority-themes/business-industry-trade-energy"
    },
    "rdfs:comment": "Monthly import and export country-by-commodity data on the UK's trade in goods, including trade by all countries and selected commodities, non-seasonally adjusted.",
    "void:sparqlEndpoint": {
      "@id": "http://gss-data.org.uk/sparql"
    }
  }
}
```

As a general point its worth remembering that _any_ semantically defined metadata is a step forward and that any json metadata schema can easily be expanded upon (i.e the above is an example goal, not necesserily where you'd need to start).

## Level 2: The addition of a table schema 

By using the csvw context we get access to defined classes that can be used to fully describe a csv file. You can view these classes at [https://www.w3.org/ns/csvw#class-definitions](https://www.w3.org/ns/csvw#class-definitions).

Example: For a given csvw file

- The schema references 1 or more `tables` as an array of type `table`.
- Each `table` has a `tableSchema`
- Each `tableSchema` has `columns` as an array of `column`
- Each `column` is represented by a hashmap of one or more fields (themselves defined classes used to describe the properties of individual columns), eg `titles`, `required`, `name` etc

It's important to understand a rule of csvw here - all fields in a csvw json file should be one of:

- a plain field defined within the csvw vocabulary (this examples)
- a prefixed field referencing one of the included supplementary vocabularies (as described for maturity level 1).

The following is a snippet that uses the former to describe a table schema.

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

You'll notice this snippet includes a datatype field, this is a simple mechanism for providing basic data validation.

This includes pattern matching (i.e `"^(M|F)$"`) using regular expressions (more information on these can be found at https://blog.usejournal.com/regular-expressions-a-complete-beginners-tutorial-c7327b9fd8eb) and a simple statment of the "age" columna as type `string`.

Both can be used to validate the provided contents of the columns in question (for more information on how you can implement said validation, please see the included supplementary section on csvw tooling).

Please note - in this example the distinction between the name and title fields seem somewhat redundant (which is fair - in a deliberately simple example) but be aware that they serve as effectively the label (title) and identifer (name) so it's a destinction that becomes increasingly important for more complex scenarios where they won't necessarily be the same.

-----

## Level 3: The addition of external tables, foreign keys and primary keys

Csvw allows for a single csvw.json file to define more than one data table. This is to permit clear linkage to external data sources.

For example, lets extend the previous example to hold information about codelists that define the columns within our principle csv.

```json
"tables": [
{
  "url": "http://gss-data.org.uk/codelists/sex.csv",
  "tableSchema": "https://gss-cogs.github.io/ref_common/codelist-schema.json",
},
{
  "url": "http://gss-data.org.uk/codelists/age.csv",
  "tableSchema": "https://gss-cogs.github.io/ref_common/codelist-schema.json",
},
{
  "url":"my-example.csv",
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
    },
  "primaryKey": [
      "age",
      "sex"
    ]
}
```

You'll notice that I've include one of the `url` references as `my-example.csv`, the lack of a full path is because **that is the csv file that this csvw schema is accompanying**. It's also worth noting that the tableSchema entries can themsevles be remote resources and can be freely reused where appropriate.

Then there is the `primaryKey` field. One crucial requirement of tidy data is that **each oberserable data point should be uniquely identifiable**. The job of the primary key is to list the dimensional options that make each observation unique. In other words - the `primaryKey` field in our example is specifiying that **a composite key of the values within the "age and "sex" columns will identify one (and only one) obervable data point**.


Lastly, we need to establish the link between the referenced codelst csvs and the principle csv (i.e "my-example.csv") that we're describing. We do this via the declaration of `Foreign Keys`.

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

This is telling us the following:
- we're importing the `resource` age.csv
- The `Age` column of our observation file (denoted by the above `columnReference` field referencing `age` - the `name` field we defined for that column earlier) ...
- Is mapped to the `notation` column of  abother resource (as denoted by the **indented** `columnReference` field).

If you're unsure where `notation` came from, it's in the tableSchema we provided when we defined the codelist tables, see here: [https://gss-cogs.github.io/ref_common/codelist-schema.json](https://gss-cogs.github.io/ref_common/codelist-schema.json)

So when you put it all together ...

```json
{
"@context": [
          "http://www.w3.org/ns/csvw",
          {
          "@language": "en"
          }
      ],
"tables": [
{
  "url": "http://gss-data.org.uk/codelists/sex.csv",
  "tableSchema": "https://gss-cogs.github.io/ref_common/codelist-schema.json",
},
{
  "url": "http://gss-data.org.uk/codelists/age.csv",
  "tableSchema": "https://gss-cogs.github.io/ref_common/codelist-schema.json",
},
{
  "url":"my-example.csv",
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
},
{
"foreignKeys": [
  {
  "columnReference": "age",
  "reference": {
    "resource": "http://gss-data.org.uk/codelists/age.csv",
    "columnReference": "notation"
    },
  }
  {
  "columnReference": "sex",
  "reference": {
    "resource": "http://gss-data.org.uk/codelists/sex.csv",
    "columnReference": "notation"
      }
  }
],
"primaryKey": [
      "age",
      "sex"
    ]
}
```

You get an english language schema fully describing the columns of a machine readable csv (with the potential for basic data validation) as well as links to both the data and schema of associated csv files representing the codes and codelists in use.

-----

## Level 4: The addition of the transform definition

We'll start with how to incorporate concept definitions in your csvw schemas.

If you think back to example one, we had the field `title` being imported via the csvw context, this property is representing `https://www.dublincore.org/specifications/dublin-core/dcmi-terms#title` and that definition is provided by the csvw context, so that field *already has* a fully semantically described definition (as do all class and properties taken from the csvw vocabulary).

The principle task therefore is to provide semantic meaning for any data you are including *that has not already described by the csvw vocabulary*. In practice this will just be the definitons of the data concepts and codes in use.

Let's look at a column definition from the earlier examples that's has been extended with additional concept information.

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

The `propertyUrl` can broadly be thought of as the concept list or dimension this column is describing.

The `valueUrl` provides definitions of the concepts or dimension-items that reside within these them.

The formatting of the `valueUrl`, eg `/age/{age}` is representing all the values within the age column. So for every given unique value in the age column its effectively referencing a url of `http://gss-data.org.uk/def/concept/age/{whichever value from the age column we're talking about}`.

Note - you'll see we're using a common SDMX dimension for the age concept and switching to our own specific resource for the ages within it. This is a typical approach for extending the linkage between datasets (not many datasets will use our exact age ranges and definitions - but most people can agree on what the general concept of "age" means).

Despite the example, you can also supply home grown `propertyUrls` as well should you be tackling a more niche concept, a concept not described elsewhere or where your using a different definition than peer organisations.

In other words ...

```json
{
  "titles": "Age",
  "required": true,
  "name": "age",
  "datatype": "string",
  "propertyUrl": "http://gss-data.org.uk/def/concept/age",
  "valueUrl": "http://gss-data.org.uk/def/concept/age/{age}"
},
```
Is also a perfectly valid definition, abeit one with less linkage (as we're using a home-grown property).

Note - it's both valid and fairly common to start with home-grown definitons of everything (much like the `gss-data.org.uk` namespace we're using in the examples) and work on harmonising your data references later, once you know what you want to use it's ultimately changing a value in a  single field.

### Defining Properties and Values

So the next question is, what exactly are the `propertyUrl` and `valueUrls` that we've started including in the previous stage?

#### propertyUrl

In the context of a column of data, the property (as identified bythe propertyURL) encapsulates several pieces of information:

- the concept being represented (e.g. time or geographic area),
- the nature of the component (dimension, attribute or measure) as represented by the type of the component property,
- the type or code list used to represent the value.

So the next step will be capturing this information and making it availible at the provided urls.

`Property Example 1`: the generic SDMX property definition for age we referenced previously can be seen here: [http://purl.org/linked-data/sdmx/2009/dimension#age](http://purl.org/linked-data/sdmx/2009/dimension#age) and is very much a simple dump of basic SDMX attriutes written in an RDF form (that nonetheless provides the three principle pieces of information listed above).

You'll notice this page also covers a number of concepts at a very basic level, this is perfectly valid and may even be a good starting point.

`Property Example 2`: a property definition taken from an RDF centric data platform can be seen here [](). This is more in depth and extends the base information in a way that adds value for users (i.e it allows more in the way of cross-querying datasets).

Either will work, but unless you have access to a RDF platform then it's probably the most practical to start with a variation of the former and build on it.

For a much more detailed look into properties and how they (will eventually) help inform the data structure definition of a semantically defined dataset please see: [https://www.w3.org/TR/vocab-data-cube/#dsd](https://www.w3.org/TR/vocab-data-cube/#dsd)

### Defining an 'about' url

## Level 5: The addition of a dataset structure definition.

So at this point we have a csvw schema that fully describes the concepts and codelists it's referencing, so that last thing to accomplish is to provide the missing details of a full dataset strcuture defintion.

# Supplementary

## REMOVE OR USE

For ease of use, csvw uses a system of known prefixes to define fields without cluttering files with endless url references.

For example the field `title` represents the property `title` as defined by the dc (**d**ublin **c**ore) vocabulary, which can be viewed here: https://www.dublincore.org/specifications/dublin-core/dcmi-terms/

Csvw supports a range of predefined properties imported as part of its context, a full list of
the csvw vocabulary can be found here: [https://www.w3.org/ns/csvw](https://www.w3.org/ns/csvw)

To make us of this pre-defined context you include the `@context` key in your csvw file, example:

```json
"@context": [
          "http://www.w3.org/ns/csvw",
          {
          "@language": "en"
          }
      ],
}
```


-----

##  A brief note on urls

We can't really get into url representations of data without touching on linked data and there's an important nuance to understand here.

Linked data is really all about **name spaces**. Every resource in the world of linked data should be **uniquely identifiable** and transparently available to everyone.

Since every url on the world wide web **is already** uniquely identifiable and available to everyone, then the use of csvw (which is really just a user friendly way of defining things via urls) moves you firmly in the direction of fully linked data.

To be clear, you *should* also use a url that tells you a little something about the thing being defined, or at very least you have a place you *could* provide that information, but it's this principle of "definition by public namepace" that underpins what we're doing here.

In other words, `first and foremost make everything identifiable, you can always expand on the definition (and even harmonise your references) later`.

-----
## CSVW Validation

## Output Tooling

So with the work so far we have a fully semantically described schema for a dataset as represented by a csv file. This means that we have everything required to convert the data into fully declared RDF triples.

`TODO - some guidence on tooling, csv2rdf etc`