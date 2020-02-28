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
### Commands
```sh
# Adding a node to a cluster (dev only)
bin/elasticsearch -Enode.name=node-3 -Epath.data=./node-3/data -Epath.logs=./node-3/logs
```

## Miscalanous
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
