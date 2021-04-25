# Mapping & Analysis

## Analysis
---
Before the document is saved on Elastic, it is analysed to defined the best way to persist the data and how to search it.

The process of analysis of document is called `Analyzer` and can divided into 3 parts:

- `Character Filters` - Adds, removes or changes characters ( Can have zero or more).
```
# *html_strip* - removes html elements
Input: "I&apos;m in a <em>good</em> mood&nbsp; and I <strong>love</strong> açaí!"

Output: "I'm in a good mood - and I love açaí"
```
- `Tokenizer` - Splits a String into tokens. (Just 1 is allowed)
```
Input: "I REALLY like beer!"

Output: ["I", "REALLY", "like", "beer"]
```
- `Token Filters` - Receives tokens from tokenizer then add, remove or modify tokens (Can have zero or more token filters)
```
# LowerCase Token Filter
Input: ["I", "REALLY", "like", "beer"]

Output: ["i", "really", "like", "beer"]
```

### Using the API
```bash
POST /_analyze
{
  "analyzer": "standard",
  "text": "2 guys walk into    a bar, but the third... DUCKS! :--)"  
}
```

## Mapping
---
Mapping defines the structure of documents, fields and their data types (likewise a database schema). Used to configure how values are been indexed.

On Elastic Search there are 2 types of mapping:

- `Explicit` - Define by ourselves
- `Dynamic`  - Generated automatically

`Important !!! `

- `Optional Fields` - All fields in ElasticSearch are **optional**, so **you can leave out** a field when indexing documents. Adding a field mapping does not make a field required. Also, searches automatically handle missing fields.

- `Cannot Update Mapping` - Once a mapping is defined it is impossible to change its type. Since will be necessary to reindex and reanalyses all data.

- `Cannot Delete Mapping` - Once a mapping is defined it is impossible to delete it. Since will be necessary to reindex and reanalyses all data.


### Mapping API`s

- `Creating Index with Mapping`
```bash
# Creates an Index defining its mapping
# PUT /<index>
# { "mappings" : { "properties" : <fields-values> } }
PUT /reviews
{
  "mappings": {
    "properties": {
      "rating": { "type" : "float" },
      "content": { "type": "text" },
      "product_id": { "type": "integer" },
      "author": {
        "properties": {
          "first_name": { "type": "text" },
          "last_name": { "type": "text" },
          "email": { "type": "keyword"  }
        }
      }
    }
  }
}

# Or better way:
PUT /reviews_better
{
  "mappings": {
    "properties": {
      "rating": { "type" : "float" },
      "content": { "type": "text" },
      "product_id": { "type": "integer" },
      "author.first_name": {"type": "text"},
      "author.last_name": {"type": "text"},
      "author.email": {"type": "keyword"}
    }
  }
}
```

- `Getting Index Mapping`
```bash
# GET /<index>/_mapping
GET /reviews/_mapping
```

- `Adding field to Existing Mapping`
```bash
# Adds a new field to existing mapping
# PUT /<index>/_mapping
# { "properties: { <fields-values> }" }
PUT /reviews/_mapping
{
  "properties": {
    "created_at": { "type": "date" }
  }
}
```

## Index Template
---

Allows Elasticsearch to reutilize or detect how configure an index when it is created. Usefull for metrics and logs indeces that have thousands of documents.

Suppose you have an access-logs indeces for every day. So instead of every day create a new index with the same mapping, it is possible to create a template that will be applied when the `index_patterns` matches.

```bash
PUT /_template/access-logs
{
  "index_patterns": ["access-logs-*"],
  "settings": {
    "number_of_shards": 2,
    "index.mapping.coerce": false
  }, 
  "mappings": {
     "properties": {
       "@timestamp": { "type": "date" },
       "url.original": { "type": "keyword"},
       "http.request.referrer": { "type": "keyword" },
       "http.response.status_code": { "type": "integer"}
     }
  }
}
```
Then when creating a new index that matches the pattern all the configuration will be applied

```bash
PUT /access-logs-2020-01-01

# It is possible check the configuration by using the following command
GET /access-logs-2020-01-01
```

## Dynamic Templates
---

Allows to customize how Elasticsearch maps data beyond the default `dynamic field mapping rules`.

```bash
PUT /dynamic_template_test
{
  "mappings": {
    # Changes the default role to map a number to long, used integer intead
    "dynamic_templates": [
      {
        "integers": {
          "match_mapping_type": "long",
          "mapping": {
            "type": "integer"
          }
        }
      }  
    ]
  }
}

POST /dynamic_template_test/_doc
{
   "value": 5
}
```


## DataTypes
---

- `Object` - Json Object that supports nested, but have its properties **flattened** (nested relationship is lost).
``` bash
{
  "group" : "fans",
  "user" : [ 
    {
      "first" : "John",
      "last" :  "Smith"
    },
    {
      "first" : "Alice",
      "last" :  "White"
    }
  ]
}

# is stored as:
{
  "group" :        "fans",
  "user.first" : [ "alice", "john" ],
  "user.last" :  [ "smith", "white" ]
}
```
- `nest` - JsonObject nested in other JsonObject preserving the relationship. As a result the nested document is stored as an entire new document (hidden document), and needs a special operator for searching
```bash
{
  "mappings": {
    "properties": {
      "user": {
        "type": "nested" 
      }
    }
  }
}
```
- `keyword` - Used for exact matching fields, like **IDs**, **zip codes**, **status codes** and **hostnames** (Internally used for filtering, sorting and aggregations).

- `date` - Used to store date. The date data should be in long miliseconds (UnixTime * 1000) or in a string format (ISO 8601).

**There is not an array datataype !** - Arrays values are merged together and just analyzed if they are **strings**. Also all array values should be of the same type.


## Utils
---

- [Analyzer APis](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-analyze.html)

- [Mapping APis](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices.html#mapping-management)

- [Mapping Parameters](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-params.html)

- [Index Template API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-templates-v1.html)

- [Dynamic Template](https://www.elastic.co/guide/en/elasticsearch/reference/current/dynamic-templates.html)

- [Elastic Commmon Schema](https://www.elastic.co/guide/en/ecs/current/ecs-reference.html)

- [Data Types](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html)
