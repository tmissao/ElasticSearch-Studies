# Controlling Query Results

- `Change the Result Format`
```bash
# GET /<index>/_search?format=<format>
# { "query": {...} }

GET /recipe/_search?format=yaml # Sets yaml result format
{
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
```

- `Pretty Json Response`
```bash
# GET /<index>/_search?pretty
# { "query": {...} }

GET /recipe/_search?pretty # Configures ES to return a pretty json
{
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
```

- `Filtering _Source`
```bash
# GET /<index>/_search
# { "_source": ["<fields>"] , "query": {...} }

# filtering source elements
GET /recipe/_search
{
  "_source": ["title"], 
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}

# filtering source elements with wildcard
GET /recipe/_search
{
  "_source": "ingredients.*", 
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
# Including and Excluding Fields
GET /recipe/_search
{
  "_source": {
    "includes": "ingredients.*",
    "excludes": "ingredients.name"
  }, 
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}

# disabling all source elements
GET /recipe/_search
{
  "_source": false, 
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
```

- `Result Size`
```bash
# GET /<index>/_search
# { "size": <number-of-result>, "query": {...} }

GET /recipe/_search
{
  "size": 2, # Sets the maximum result set size to 2. Defaults 10.
  "_source": false,
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
```

- `Offset`
```bash
# GET /<index>/_search
# { "from": <number-of-first-results-to-skip>", size": <number-of-result>, "query": {...} }

GET /recipe/_search
{
  "from": 4 # Ignores the first four results
  "size": 2,
  "_source": false,
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
```

- `Sorting`
```bash
# GET /<index>/_search
# { "query": {...}, "sort": [ { "<sort-field>" : "asc|desc"}, { "<sort-field2>" : "asc|desc"}  ]}

GET /recipe/_search
{
  "_source": [ "preparation_time_minutes", "created" ]  
  "query": {
    "match": {
      "title": "pasta"
    }
  }
  "sort": [
    { "preparation_time_minutes": "asc" },
    { "created": "desc" }
  ]
}
```

- `Sorting Multi Values Fields`
```bash
# GET /<index>/_search
# { "query": {...}, "sort": [ { "<field>": { "order": "asc|desc", "mode": "max|min|avg|sum"  }}]}

GET /recipe/_search
{
  "_source": [ "ratings" ]  
  "query": {
    "match": {
      "title": "pasta"
    }
  }
  "sort": [
    {
      # Calculates the AVG value from all values inside `ratings` field.
      "ratings": {
        "order": "desc",
        "mode": "avg"
      }
    }
  ]
}
```

## Utils
---

- [Response Data Format](https://www.elastic.co/guide/en/elasticsearch/reference/current/sql-rest-format.html)

- [ES Common Options](https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html)
