# ElasticSearch: Quick Start Guide
## Installation

via *docker* and *docker-compose*: `docker-compose.yml`:
```yaml
version: '3'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.0.0
    environment:
      - discovery.type=single-node
    ports:
      - 9200:9200
  kibana:
    image: docker.elastic.co/kibana/kibana:7.0.0
    ports:
      - 5601:5601
```
Save the file and run `docker-compose up`. After it finishises running the following addresses till be awailable:
* Elastic Search (mappings): `http://localhost:9200/_mappings`
* Kibana (development console): `http://localhost:5601/app/kibana#/dev_tools/console?_g=()` 
Configuration, plugins and module files are available in the `$ES_HOME` (`/usr/share/elasticsearch`) directory in the docker container.

## Queries and commands
### Queries 
#### Exporing a cluster
```bash
GET /_cluster/health
GET /_nodes
GET /_cat/nodes?v
GET /_cat/indices?v
GET /_cat/shards?v
```
#### Indexes
```bash
PUT /pages # create an index
DELETE /pages # delete an index
```
#### Documents 
##### Add a document
```bash
POST /pages/_doc # | POST /pages/_doc/ID
{
  "property": "value"
}
```
##### Fetch a document
```bash
GET /pages/_doc/12 # fetch a document
```
##### Edit a document
```bash
PUT /pages/_doc/12
{
  "property": "value"
}
POST /pages/_update/12
{
  "doc": {
   "name": "SomeName" 
  }
}
```
##### Scripting Updates
Add `word` to the `name` property if said property is not `word`. 
```bash
POST /pages/_update/12
{
  "script": {
   "source": """
		 if (ctx._source.name == params.word) {
			ctx.op = 'noop';
		 }
		 
		 ctx._source.name = params.word + ctx._source.name
		""",
   "params": {
     "word": "new"
   }
  }
}
```
##### Aliases
###### Add: 
```bash
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "deal.v2.2",
        "alias": "deal"
      }
    }
  ]
}
```
###### Remove
```bash
POST /_aliases
{
  "actions": [
    {
      "remove": {
        "index": "deal.v2.1",
        "alias": "deal"
      }
    }
  ]
}
```
### Recepies
#### Extract and search throu nested properties of a document
##### Index
```json
PUT deal
{
  "settings": {
    "analysis": {
      "normalizer": {
        "case_insensitive": {
          "filter": "lowercase"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "timetableEvents" : {
        "type": "nested",
        "properties" : {
          "agencyName" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "normalizer": "case_insensitive"
              }
            }
          }
        }
      }
    }
  }
}
```
##### Documents
```json
PUT deal/_doc/1
{
  "timetableEvents": [
    {
      "agencyName": "Brazilian National Telecommunications Agency (Agência Nacional de Telecomunicações (ANATEL))",
      "agencyId": 1,
      "agencyJurisdiction": "Brazil"
    },
    {
      "first": "Brazilian Secretariat for Economic Monitoring of the Ministry of Finance",
      "agencyId": 2,
      "agencyJurisdiction": "Brazil"
    },
    {
      "first": "Brazilian National Agency for Civil Aviation",
      "agencyId": 3,
      "agencyJurisdiction": "Brazil"
    },
    {
      "first": "Brazilian Electricity Regulatory Agency",
      "agencyId": 4,
      "agencyJurisdiction": "Brazil"
    }
  ]
}
PUT deal/_doc/2
{
  "timetableEvents": [
    {
      "first": "Brazilian Electricity Regulatory Agency",
      "agencyId": 5,
      "agencyJurisdiction": "Brazil"
    }
  ]
}
```
##### Search Query
```json
GET deal/_search
{
  "size": 0,
  "aggs": {
    "TTE": {
      "nested": {
        "path": "timetableEvents"
      },
      "aggs": {
        "agencies": {
          "composite": {
            "after": {
              "searchFields": "bank of albania",
              "agencyId": 73216
            },
            "size": 20,
            "sources": [
              {
                "searchFields": {
                  "terms": {
                    "script": {
                      "source": " if (doc['timetableEvents.agencyName.keyword'].size()==0) { return null; } if (doc['timetableEvents.agencyName.keyword'].value.indexOf('razilian') < 0) { return null; } return doc['timetableEvents.agencyName.keyword'].value; ",
                      "lang": "painless"
                    }
                  }
                }
              },
              {
                "agencyId": {
                  "terms": {
                    "field": "timetableEvents.agencyId"
                  }
                }
              }
            ]
          },
          "aggs": {
            "top": {
              "top_hits": {
                "size": 1,
                "_source": [
                  "timetableEvents.agencyDescription",
                  "timetableEvents.agencyJurisdiction",
                  "timetableEvents.agencyName",
                  "timetableEvents.agencyId"
                ]
              }
            }
          }
        }
      }
    }
  }
}
```
##### Result
```json
{
  "took" : 19,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "TTE" : {
      "doc_count" : 5,
      "agencies" : {
        "after_key" : {
          "searchFields" : "brazilian national telecommunications agency (agência nacional de telecomunicações (anatel))",
          "agencyId" : 1
        },
        "buckets" : [
          {
            "key" : {
              "searchFields" : "brazilian national telecommunications agency (agência nacional de telecomunicações (anatel))",
              "agencyId" : 1
            },
            "doc_count" : 1,
            "top" : {
              "hits" : {
                "total" : {
                  "value" : 1,
                  "relation" : "eq"
                },
                "max_score" : 1.0,
                "hits" : [
                  {
                    "_index" : "deal",
                    "_type" : "_doc",
                    "_id" : "1",
                    "_nested" : {
                      "field" : "timetableEvents",
                      "offset" : 0
                    },
                    "_score" : 1.0,
                    "_source" : {
                      "agencyId" : 1,
                      "agencyName" : "Brazilian National Telecommunications Agency (Agência Nacional de Telecomunicações (ANATEL))",
                      "agencyJurisdiction" : "Brazil"
                    }
                  }
                ]
              }
            }
          }
        ]
      }
    }
  }
}

```
#### Copy data between indexes:
```sh
POST /_reindex?wait_for_completion=true
{
  "source": {
    "index": "deal.v2.1"
  },
  "dest": {
    "index": "deal.v2.2"
  }
}
```
#### Case insensitive Sorting
Index:
```sh
PUT /deal
{
  "settings": {
    "analysis": {
      "normalizer": {
        "case_insensitive": {
          "filter": "lowercase"
        }
      }
  },
  "mappings": {
    "properties": {
      "targetCompanySortValue": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "normalizer": "case_insensitive"
          }
        }
      }
    }
  }
}
```
Query:
```sh
GET /deal/_search
{
  "_source": {
    "includes": [
      "targetCompanySortValue"
    ]
  },
  "size": 20,
  "sort": [
    {
      "targetCompanySortValue.keyword": "desc"
    }
  ]
}
```
### Commands
```sh
# Adding a node to a cluster (dev only)
bin/elasticsearch -Enode.name=node-3 -Epath.data=./node-3/data -Epath.logs=./node-3/logs
```

## Miscellaneous
### Concepts 
####  Sharding
**_Sharding_**: dividing indeces into smaller piecies (shards), it's done on an index level. Shards can be though of as independent indeces. Each shard is a seperate _Apache Luciene_ index. Disc space used grows as documents are added. The shard storage limit is 2 bilion documents. Queries can be run on multiple shards simultaneously. Running `GET /_cat/indices?v` gives us a `pri` column - this is the number of shards per index. For increasing/decreasing the number of shards see `Split API` / `Shrink API`. Create more than one shard only fox indexes with more than a milion documents.

#### Replication
**_Replication_**: Creating copies of shards, it's done on an index level. _Primary shard_ is the replicated shard, _replica shards_ are it's copies and there're called a _replication group_. 

### Settings
* Cluster
	*	`action.auto_create_index` - index fill be created when adding documents.
* Indexes
	* `auto_expand_replicas` - change the number of replicas based on the number of nodes.
* Nodes:
	* `node.master` - sets node as a master node. 
	* `node.data` - sets node as a data node. 
	* `node.voting_only` - a voting_only node cannot take the master role.
	* `node.ingest` - enable injest pipelines. An *ingest pipeline* is a series of steps that should be performed when ingesting a document It has _Processors_ for manipulating documents.
	* `node.ml` - identifies node as a machine learning node.
	* `xpack.ml.enabled` - enables machine learning API.

### Other stuff to search on the internet
* X-Pack (elastic plugin)
	* Graph
	* Elasticseach SQL
* Beats
  * Filebeat
  * Metricbeat
* Snapshots
