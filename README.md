# Elasticsearch Langdetect Ingest Processor

Uses the [langdetect](https://github.com/YouCruit/language-detection/) plugin to try to find out the language used in a field.

## Installation

| ES    | Command |
| ----- | ------- |
| 6.4.2 | `bin/elasticsearch-plugin install https://github.com/spinscale/elasticsearch-ingest-langdetect/releases/download/6.4.2.1/ingest-langdetect-6.4.2.1.zip` |
| 6.4.1 | `bin/elasticsearch-plugin install https://github.com/spinscale/elasticsearch-ingest-langdetect/releases/download/6.4.1.1/ingest-langdetect-6.4.1.1.zip` |
| 6.4.0 | `bin/elasticsearch-plugin install https://github.com/spinscale/elasticsearch-ingest-langdetect/releases/download/6.4.0.1/ingest-langdetect-6.4.0.1.zip` |
| 6.3.2 | `bin/elasticsearch-plugin install https://github.com/spinscale/elasticsearch-ingest-langdetect/releases/download/6.3.2.1/ingest-langdetect-6.3.2.1.zip` |
| 6.3.1 | `bin/elasticsearch-plugin install https://github.com/spinscale/elasticsearch-ingest-langdetect/releases/download/6.3.1.1/ingest-langdetect-6.3.1.1.zip` |
| 6.3.0 | `bin/elasticsearch-plugin install https://github.com/spinscale/elasticsearch-ingest-langdetect/releases/download/6.3.0.1/ingest-langdetect-6.3.0.1.zip` |
| 6.2.4 | `bin/elasticsearch-plugin install https://github.com/spinscale/elasticsearch-ingest-langdetect/releases/download/6.2.4.1/ingest-langdetect-6.2.4.1.zip` |
| 6.2.3 | `bin/elasticsearch-plugin install https://github.com/spinscale/elasticsearch-ingest-langdetect/releases/download/6.2.3.1/ingest-langdetect-6.2.3.1.zip` |
| 6.2.2 | `bin/elasticsearch-plugin install https://github.com/spinscale/elasticsearch-ingest-langdetect/releases/download/6.2.2.1/ingest-langdetect-6.2.2.1.zip` |
| 5.2.0 | `bin/elasticsearch-plugin install https://oss.sonatype.org/content/repositories/releases/de/spinscale/elasticsearch/plugin/ingest-langdetect/5.2.0.1/ingest-langdetect-5.2.0.1.zip` |
| 5.1.2 | `bin/elasticsearch-plugin install https://oss.sonatype.org/content/repositories/releases/de/spinscale/elasticsearch/plugin/ingest-langdetect/5.1.2.1/ingest-langdetect-5.1.2.1.zip` |
| 5.1.1 | `bin/elasticsearch-plugin install https://oss.sonatype.org/content/repositories/releases/de/spinscale/elasticsearch/plugin/ingest-langdetect/5.1.1.1/ingest-langdetect-5.1.1.1.zip` |

## Usage


```
PUT _ingest/pipeline/langdetect-pipeline
{
  "description": "A pipeline to do whatever",
  "processors": [
    {
      "langdetect" : {
        "field" : "my_field",
        "target_field" : "language"
      }
    }
  ]
}

PUT /my-index/my-type/1?pipeline=langdetect-pipeline
{
  "my_field" : "This is hopefully an english text, that will be detected."
}

GET /my-index/my-type/1

# Expected response
{
  "my_field" : "This is hopefully an english text, that will be detected.",
  "language": "en"
}
```

You could also set certain fields that use different analyzers for different languages

```
PUT _ingest/pipeline/langdetect-analyzer-pipeline
{
  "description": "A pipeline to index data into language specific analyzers",
  "processors": [
    {
      "langdetect": {
        "field": "my_field",
        "target_field": "lang"
      }
    },
    {
      "script": {
        "source": "ctx.language = [:];ctx.language[ctx.lang] = ctx.remove('my_field')"
      }
    }
  ]
}

PUT documents
{
  "mappings": {
    "doc" : {
      "properties" : {
        "language": {
          "properties": {
            "de" : {
              "type": "text",
              "analyzer": "german"
            },
            "en" : {
              "type": "text",
              "analyzer": "english"
            }
          }
        }
      }
    }
  }
}

PUT /my-index/doc/1?pipeline=langdetect-analyzer-pipeline
{
  "my_field" : "This is an english text"
}

PUT /my-index/doc/2?pipeline=langdetect-analyzer-pipeline
{
  "my_field" : "Das hier ist ein deutscher Text."
}

GET my-index/doc/1

GET my-index/doc/2
```

## Configuration

| Parameter | Use |
| --- | --- |
| field         | Field name of where to read the content from |
| target_field  | Field name to write the language to |
| max_length    | Max length of of characters to read, defaults to 10kb, requires a byte size value, like 1mb |

## Setup

In order to install this plugin, you need to create a zip distribution first by running

```bash
gradle clean check
```

This will produce a zip file in `build/distributions`.

After building the zip file, you can install it like this

```bash
bin/plugin install file:///path/to/ingest-langdetect/build/distribution/ingest-langdetect-0.0.1-SNAPSHOT.zip
```

## Side notes

In order to cope with the security manager, a special factory is used to load the languages from the classpath.
You can check out the `SecureDetectorFactory` class. This implementation also does not use jsonic to prevent the use of reflection when loading the languages.
