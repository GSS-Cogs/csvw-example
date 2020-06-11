# CSVW Walkthrough

The following document is walkthrough of using csvw but without direct reference to a specific example. The intention is to delve more into the _why_ than the _how_. 

- [CSVW Walkthrough](#csvw-walkthrough)
  - [Fundementals](#fundementals)
    - [Prefixes](#prefixes)
    - [CSVW Data Validation](#csvw-data-validation)
    - [External File References](#external-file-references)
  - [The Relationship Between Csvw and Linked Data](#the-relationship-between-csvw-and-linked-data)
      - [A brief note on urls](#a-brief-note-on-urls)
  - [Extension 1](#extension-1)
    - [Extending A Schema With Concept Definfitions](#extending-a-schema-with-concept-definfitions)
  - [Extension 2](#extension-2)
    - [Defining Properties and Values](#defining-properties-and-values)
      - [propertyUrl](#propertyurl)
  - [Extension 3](#extension-3)
    - [Adding Full Sementic Meaning](#adding-full-sementic-meaning)
  - [Putting it all together](#putting-it-all-together)
  
  
## Fundementals

### Prefixes

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

This idea of context is inderited from json-ld where it is defined as follows: `In some cases, when describing an attribute of an entity, it is tempting to using string values which have no independent meaning. Such values often are used for well known things. A JSON-LD context can define a term for such values, which allow them to appear as strings within the message, but be associated with specific identifiers.`


-----
### CSVW Data Validation

By using the csvw context we get access to defined classes that can be used to fully describe a csv file. You can view these classes at [https://www.w3.org/ns/csvw#class-definitions](https://www.w3.org/ns/csvw#class-definitions).

Example: For a given csvw file

- The schema references 1 or more `tables` as an array of type `table`.
- Each `table` has a `tableSchema`
- Each `tableSchema` has `columns` as an array of `columns`
- Each `column` is represented by a hashmap of one or more fields (themselves defined classes used to describe the properties of individual columns), eg `titles`, `required`, `name` etc

The following is a snipped from a csvw file that holds the necessary information to validate the files content.

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

The above pattern matching (i.e `"^(M|F)$"`) is done using regular expressions (more information on these can be found at https://blog.usejournal.com/regular-expressions-a-complete-beginners-tutorial-c7327b9fd8eb).

-----

### External File References

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
    }
}
```

You'll notice that I've include one of the `url` references as `my-example.csv`, the lack of a full path is because **that is the csv file that this csvw schema is accompanying**. It's also worth noting that the tableSchema entries can themsevles be remote resources and can be freely reused where appropriate.

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

If you're unsure where `notation` came from, it's in the tableSchema we provided when we defined the codelist tables, see here: [https://gss-cogs.github.io/ref_common/codelist-schema.json](https://gss-cogs.github.io/ref_common/codelist-schema.j)

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
"foreignKeys": [
  {
  "columnReference": "age",
  "reference": {
    "resource": "http://gss-data.org.uk/codelists/age.csv",
    "columnReference": "notation"
  },
  {
  "columnReference": "sex",
  "reference": {
    "resource": "http://gss-data.org.uk/codelists/sex.csv",
    "columnReference": "notation"
  }
  }
]

```

You get an english language schema fully describing the columns of a csv (with basic data validation) as well as linking to both the data and schema of associated csv files representing the codes and codelists in use.

-----

## The Relationship Between Csvw and Linked Data

The guidance up to this point has been deliberately general information, so applies to **all** csvw files, regardless of the eventual goals of their creation.

####  A brief note on urls

We can't really get into url representations of data without touching on linked data and there's an important nuance to understand here. 

Linked data is really all about **name spaces**. Every resource in the world of linked data should be **uniquely identifiable** and transparently available to everyone.

Since every url on the world wide web **is already** uniquely identifiable and available to everyone, then the use of csvw (which is really just a user friendly way of defining things via urls) moves you firmly in the direction of fully linked data.

To be clear, you *should* also use a url that tells you a little something about the thing being defined, or at very least you have a place you *could* provide that information, but it's this principle of "definition by public namepace" that underpins what we're doing here.

In other words, `first and foremost make everything identifiable, you can always expand on the definition (and even harmonise your references) later`. 

The following sections details how you can expand on the fundementals to start bringing in these additional elements of semantic definition. 

-----

## Extension 1

### Extending A Schema With Concept Definfitions

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

Note - you'll see we're using a common SDMX dimension for the age concept and switching to our own specific resource for the ages with it. This is a typical approach for extending the linkage between datasets (not many datasets will use our exact age ranges and definitions - but most people can agree on what the general concept of "age" means). 

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


-----

## Extension 2

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


## Extension 3

### Adding Full Sementic Meaning

So at this point we have a csvw schema that fully describes the concepts and codelists it's referencing, so that last thing to accomplish is to provide the missing details of a full dataset strcuture defintion.

## Putting it all together

So with the work so far we have a fully semantically described schema for a dataset as represented by a csv file. This means that we have everything required to convert the data into fully declared RDF triples.

`TODO - some guidence on tooling, csv2rdf etc`

