# Reindexing

To change or update a mapping of an index, it is necessary to reindexing all documents. So :

Suppose you created the following index **reviews** 

```bash
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
```

And you want to update the field **product_id** from *integer* to *keyword* type. For that it is necessary to reindex. For that:

1. Create a new index with the correct mapping
```bash
PUT /reviews_new
{
  "mappings": {
    "properties": {
      "rating": { "type" : "float" },
      "content": { "type": "text" },
      "product_id": { "type": "keyword" },
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
```

2. Move all documents from old index to new index using **_reindex API**, also it is possible to modify the source data throught a script
```bash
POST /_reindex
{
  "source": {
    "index": "reviews"  
  },
  "dest": {
    "index": "reviews_new"
  },
  "script": {
    "source": """
      if (ctx._source.product_id != null) {
        ctx._source.product_id = ctx._source.product_id.toString();
      }
    """
  }
}
```

3. Delete the old index
```bash
DELETE /reviews
```

4. Create the index with the correct mapping
```bash
PUT /reviews
{
  "mappings": {
    "properties": {
      "rating": { "type" : "float" },
      "content": { "type": "text" },
      "product_id": { "type": "keyword" },
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
```

5. Reindex Again but now from the **reviews_new** to **reviews**
```bash
POST /_reindex
{
  "source": {
    "index": "reviews_new"
  },
  "dest": {
    "index": "reviews"
  }
}
```

## References
---

- [Reindex APis](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html)