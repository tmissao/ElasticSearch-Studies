# Managing Documents

## Creating and Deleting Indices
---

- `Creating`
```bash
# PUT /<index>
PUT /pages
```

- `Creating with Options`
```bash
# PUT /<index>
# {
#   "settings": {
#     "setting1" : "value1"
#     "setting2" : "value2"
#   }
# }

PUT /products
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 1
  }
}
```

- `Deleting`
```bash
# DELETE /<index>
DELETE /pages
```

## Indexing Documents
---

- `Adding`
```bash
#POST /<index>/_doc
# {} jsonDoc
POST /products/_doc
{
  "name": "Coffee Maker",
  "price": 64,
  "in_stock": 10
}
```

- `Updating`
```bash
#POST /<index>/_update/<id>
# { "doc": { <fields to update or add > } }
POST /products/_update/100
{
  "doc": {
    "in_stock": 2 
  }
}
```

- `Updating with Scripting`
```bash
#POST /<index>/_update/<id>
# { "script": { "source": "<logic>" } }
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock -= params.quantity",
    "params" : {
      "quantity" : 4
    }
  }
}
```

- `Upsert`
```bash
#POST /<index>/_update/<id>
# { "script": { "source": "<logic>" }, "upsert": {jsondoc} }
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock--"
  },
  "upsert": {
    "name": "Coffee Maker",
    "price": 64,
    "in_stock": 10
  }
}
```

- `Replacing`
```bash
#PUT /<index>/_doc/<id>
# {} jsonDoc
PUT /products/_doc/100
{
  "name": "Toast",
  "price": 49,
  "in_stock": 4
}
```

- `Deleting`
```bash
#DELETE /<index>/_doc/<id>
DELETE /products/_doc/100
```

- `Getting by Id`
```bash
#GET /<index>/_doc/<id>
GET /products/_doc/100
```

- `Updating Multiple Documents`
```bash
#POST /<index>/_update_by_query
# { "script" { "source" : "<logic>" }, "query" { <logic> } }
POST /products/_update_by_query
{
  "script": {
    "source": "ctx._source.in_stock--"
  },
  "query": {
    "match_all": {}
  }
}
```

- `Deleting Multiple Documents`
```bash
#POST /<index>/_delete_by_query
# { "query" { <logic> } }
POST /products/_delete_by_query
{
  "query": {
    "match_all": {}
  }
}
```

## Bulk API
---

Allow to perform multiple operations on many documents with a single query. This API expects data to be formatted using `NDJSON` specification.

```bash
# POST /_bulk
# { <action> }
# { <document> }

# In this way it is possible to modify multiple indeces
POST /_bulk
{ "index" : { "_index": "products", "_id": 200 } }
{ "name": "Expresso Machine","price": 199,"in_stock": 5 }
{ "create" : { "_index": "products", "_id" 201} }
{ "name": "Milk Frother","price": 149,"in_stock": 14 }

# In this way it is possible to set the index on request Path, however it is impossible to modify other index
POST /products/_bulk
{ "update": { "_id": 201 } }
{ "doc": { "price": 129 }  }
{ "delete": { "_id": 200 } }
```

## Utils
---

- [Rest APis](https://www.elastic.co/guide/en/elasticsearch/reference/current/rest-apis.html)

- [Index APis](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices.html)

- [Document APis](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs.html)

- [Bulk Apis](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html)

- [Update By Query Apis](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update-by-query.html)

- [Delete By Query Apis](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete-by-query.html)