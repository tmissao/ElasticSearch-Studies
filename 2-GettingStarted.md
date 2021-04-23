# GettingStarted

## Installing ElasticSearch
---

- `Running as a Service` - https://info.elastic.co/elasticsearch-service-trial-course.html
- `Docker` - https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html

## Architecture
---

`Document` - Data stored in a index representing a unit of information, also contains metadata that is used internally by ES

![Documents](./artifacts/2-Documents.png)

`Index` -  Collection of documents, groups documents logically

![Documents](./artifacts/2-Index.png)

`Node` - Instance of ElasticSearch that stores data, and each node can store just a part of the data. A node will be always part of a cluster. So at starting the node will join a cluster or create a new one.

`Cluster` - Collection of related nodes, can have one or more nodes.

![Architecture](./artifacts/2-Architeture.png)

## ElasticSearch API
---
Allows to run commands on ElasticSearch using Rest API

### `Kibana`
[`VERB`] /[`API`]/[`COMMAND`]
```bash
# <VERB> /<API>/<COMMAND>
GET /_cluster/health
```

### `cURL`
curl -XGET -u authentication "`<endpoint>`/`<api>`/`<command>`"
```bash
curl -XGET -u elastic:xpto1325196 "https://missaostudies.es.us-west1.gcp.cloud.es.io:9243/_cluster/health"
```


## Inspect Cluster
---

- Cluster Health
```
GET /_cluster/health
```

- Nodes Informations
```
GET /_cat/nodes?v
```

- List Indeces
```
GET /_cat/indeces?v
```

## Sharding
---
A way to divide indices into smaller pieces, so each piece is a shard, allowing horizontal scalling volume. The default number os shardes in `1`.

Sharding is done at index level. 

A Shard can store up to 2 billion documents.

Shard allows queries to run in parallel, increasing the throughput of an index.

![Sharding](./artifacts/2-Sharding.png)

`Spli API` allows to increase the number of shards in a index (involves create a new index)

`Shrink API` allows to decrease the number of shards in a index (involves create a new index)

## Utils
---

- cat - https://www.elastic.co/guide/en/elasticsearch/reference/current/cat.html

- cat nodes API - https://www.elastic.co/guide/en/elasticsearch/reference/current/cat-nodes.html

- cat nodes info API - https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-nodes-info.html