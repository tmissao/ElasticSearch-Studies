# Nesting Queries & Joining Queries

## Nesting Queries
---

As known elasticsearch flattens the inner objects of a documents, as a result the relationship is lost and is impossible to query the inner object. 

A way to resolving this problem is create the mapping with inner object with nest type.

```bash
PUT /department
{
  "mappings": {  
    "properties": {
      "name": {
        "type": "text"
      },
      "employees": {
        "type": "nested" # Inner Object declared with Nest type
      }
    }
  }
}

PUT /department/_doc/1
{
  "name": "Development",
  "employees": [
    {
      "name": "Eric Green",
      "age": 39,
      "gender": "M",
      "position": "Big Data Specialist"
    },
    {
      "name": "Julie Powell",
      "age": 26,
      "gender": "F",
      "position": "Intern"
    }
  ]
}

PUT /department/_doc/2
{
  "name": "HR & Marketing",
  "employees": [
    {
      "name": "Jacqueline Hill",
      "age": 28,
      "gender": "F",
      "position": "Junior Marketing Manager"
    },
    {
      "name": "Evelyn Henderson",
      "age": 24,
      "gender": "F",
      "position": "Intern"
    }
  ]
}
```

- `Querying Nested Fields`
```bash
# GET /<index>/_search
# { query: { "nested": { "path": "<inner.path>", "query": { <query-operators> }}}}

GET /departament/_search
{
  "query" {
    "nested": {
      "path": "employees",
      "query": {
        "bool": {
          "must": [
            {
              "match": {
                "employees.position": "intern"
              }
            },
            {
              "term": {
                "employees.gender.keyword": "F"
              }
            }
          ]
        }
      }
    }
  }
}
```

## Troubleshooting Nested Queries
---

A good way to troubleshooting the nested queries ( discovery which elements makes the hit ) is enabling the property `inner_hits` in the query. In this way, elasticsearch will display information about the nested element that was hit by the query.

```bash
GET /department/_search
{
  "query": {
    "nested": {
      "path": "employees",
      "inner_hits": {}, # Activating Inner Hits
      "query": {
        "bool": {
          "must": [
            {
              "match": {
                "employees.position": "intern"
              }
            },
            {
              "term": {
                "employees.gender.keyword": "F"
              }
            }
          ]
        }
      }
    }
  }
}

# Result

{
  "took": 3,
  ...
  "hits" : [
    {
      ...
      "inner_hits" : {
        "employees" : {
          "hits" : {
            "total" : {
              "value" : 1,
              "relation" : "eq"
            },
            "max_score" : 2.3905568,
            "hits" : [
              {
                "_index" : "department",
                "_type" : "_doc",
                "_id" : "1",
                "_nested" : {
                  "field" : "employees",
                  "offset" : 3
                },
                "_score" : 2.3905568,
                "_source" : {
                  "gender" : "F",
                  "name" : "Julie Powell",
                  "position" : "Intern",
                  "age" : 26
                }
              }
            ]
          }
        }
      }
    }
  ]
}

```

## Joining Queries (Slow Queries!)
---

Another way to structure data on elasticsearch is defining relationships between the documents, like a relational database (that is not usual, and it is expensive).

- `Mapping Relationships`

```bash
# PUT /<index>
# { "mapping": { "properties": { "<join-field>": { "type": "join", "relations": { "<parent>": "child" }}}}}

PUT /department
{
  "mappings": {
    "properties": {
      "join_field": { # Custom Join Field
        "type": "join",
        "relations": {
          # deparment is the parent
          # employee is the child
          "department": "employee"
        }
      }
    }
  }
}
```

- `Adding Parents Documents`
```bash
PUT /index/_doc/<index>
#  { ...values, "join_field": "<relationship>"}

PUT /department/_doc/1
{
  "name": "Development",
  # Custom Join Field
  # Parent Relationship
  "join_field": "department"
}

PUT /department/_doc/2
{
  "name": "Marketing",
  "join_field": "department"
}
```

- `Adding Children Documents`

When adding children documents it is necessary to ensure that parent and children documents are stored in same shard, for that the `routing` property should be used in the query string passing the parent`s id.
```bash
PUT /index/_doc/<index>?routing=<parentId>
#  { ...values, "join_field": { "name": "<relationship>", "parent": "<parentId>" }}

PUT /department/_doc/3?routing=1
{
  "name": "Bo Andersen",
  "age": 28,
  "gender": "M",
  # Custom Join Field
  "join_field": {
    # Children Relationship
    "name": "employee",
    # Parent ID
    "parent": 1
  }
}

PUT /department/_doc/4?routing=2
{
  "name": "John Doe",
  "age": 44,
  "gender": "M",
  "join_field": {
    "name": "employee",
    "parent": 2
  }
}

PUT /department/_doc/5?routing=1
{
  "name": "James Evans",
  "age": 32,
  "gender": "M",
  "join_field": {
    "name": "employee",
    "parent": 1
  }
}
```

- `Querying by Parent Id`
```bash
# GET /department/_search
# { "query": { "parent_id": { "type": "<relationship>", "id": "<parentId>" }}}

GET /department/_search
{
  "query": {
    "parent_id": {
      "type": "employee",
      "id": 1
    }
  }
}
```

- `Querying Child Documents By Parent`
```bash
# GET /index/_search
# {"query": { "has_parent": { "parent_type": "<relationship>", "query": { ... } }}}

# Gets Children querying Parents
GET /department/_search
{
  "query": {
    "has_parent": {
      "parent_type": "department",
      "query": {
        "term": {
          "name.keyword": "Development"
        }
      }
    }
  }
}
```

- `Querying Parent Documents By Children`
```bash
# GET /index/_search
# {"query": { "has_clild": { "type": "<relationship>", "query": { ... } }}}

# Gets Parents querying children
GET /department/_search
{
  "query": {
    "has_child": {
      "type": "employee",
      "query": {
        "bool": {
          "must": [
            {
              "range": {
                "age": {
                  "gte": 50
                }
              }
            }
          ],
          "should": [
            {
              "term": {
                "gender.keyword": "M"
              }
            }
          ]
        }
      }
    }
  }
}
```

## Utils
---

- [Nested Queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-nested-query.html)

- [Join Field type](https://www.elastic.co/guide/en/elasticsearch/reference/current/parent-join.html)

- [Parent ID Query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-parent-id-query.html)

- [Has Child Query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-has-child-query.html)
