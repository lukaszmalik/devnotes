## PARR-1542
### Mapping
```json
PUT deal
{
  "settings": {
    "analysis": {
      "filter": {
        "autocomplete_filter": {
          "type": "edge_ngram",
          "min_gram": 1,
          "max_gram": 10
        }
      },
      "normalizer": {
        "case_insensitive": {
          "filter": "lowercase"
        }
      },
      "analyzer": {
        "autocomplete_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "autocomplete_filter"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "dealValue": {
        "type": "nested",
        "properties": {
          "currencyCode": {
            "type": "keyword"
          },
          "isDominant": {
            "type": "boolean"
          },
          "value": {
            "type": "float"
          }
        }
      },
      "companies": {
        "properties": {
          "bidders": {
            "type": "nested",
            "properties": {
              "name": {
                "type": "text",
                "fields": {
                  "keyword": {
                    "type": "keyword",
                    "normalizer": "case_insensitive"
                  }
                }
              },
              "sectors" : {
                "type": "nested",
                "properties" : {
                  "code" : {
                    "type" : "text",
                    "fields" : {
                      "keyword" : {
                        "type" : "keyword",
                        "normalizer": "case_insensitive"
                      }
                    }
                  },
                  "name" : {
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
          },
          "targets": {
          "type": "nested",
            "properties": {
              "name": {
                "type": "text",
                "fields": {
                  "keyword": {
                    "type": "keyword",
                    "normalizer": "case_insensitive"
                  }
                }
              },
              "sectors" : {
                "type": "nested",
                "properties" : {
                  "code" : {
                    "type" : "text",
                    "fields" : {
                      "keyword" : {
                        "type" : "keyword",
                        "normalizer": "case_insensitive"
                      }
                    }
                  },
                  "name" : {
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
      },
      "dominantTargetGeography": {
        "properties": {
          "name": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "normalizer": "case_insensitive"
              }
            }
          }
        }
      },
      "dominantSector": {
        "properties": {
          "name": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "normalizer": "case_insensitive"
              }
            }
          }
        }
      },
      "timetableEvents": {
        "type": "nested",
        "properties": {
          "agencyName": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "normalizer": "case_insensitive"
              }
            }
          },
          "agencyJurisdiction": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "normalizer": "case_insensitive"
              }
            }
          },
          "agencyJurisdictionGroup": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "normalizer": "case_insensitive"
              }
            }
          }
        }
      },
      "targetCompanySortValue": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "normalizer": "case_insensitive"
          }
        }
      },
      "bidderCompanySortValue": {
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

### Agency picker
#### Query
```json
GET deal.v2.3/_search
{
  "size": 0,
  "aggs": {
    "TTE": {
      "nested": {
        "path": "timetableEvents"
      },
      "aggs": {
        "filtered_timetableEvents": {
          "filter": {
            "bool": {
              "must": [
                {
                  "wildcard": {
                    "timetableEvents.agencyName.keyword": "*bank*"
                  }
                }
              ]
            }
          },
          "aggs": {
            "group_by_company": {
              "terms": {
                "order": {
                  "_key": "asc"
                },
                "size": 20,
                "field": "timetableEvents.agencyName.keyword"
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
  }
}
```
#### Curl command
```sh
curl  -s http://web:9200/deal/_search -H "Content-Type: application/json" -d "$(cat query.json)" | jq ".aggregations.TTE.filtered_timetableEvents.group_by_company.buckets[].top.hits.hits[]._source.agencyName"
````

### Company:target picker
#### Query
```json
GET deal.v2.3/_search
{
  "size": 0,
  "aggs": {
    "companies": {
      "nested": {
        "path": "companies.targets"
      },
      "aggs": {
        "filtered_companies": {
          "filter": {
            "bool": {
              "must": [
                {
                  "wildcard": {
                    "companies.targets.name.keyword": "*IBM*"
                  }
                }
              ]
            }
          },
          "aggs": {
            "group_by_company": {
              "terms": {
                "order": {
                  "_key": "asc"
                },
                "size": 20,
                "field": "companies.targets.name.keyword"
              },
              "aggs": {
                "additional_data": {
                  "top_hits": {
                    "size": 1,
                    "_source": [
                      "companies.targets.id",
                      "companies.targets.mmgid",
                      "companies.targets.name",
                      "companies.targets.description"
                    ]
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
```
#### Curl command
```sh
curl  -s http://web:9200/deal/_search -H "Content-Type: application/json" -d "$(cat query.json)" | jq ".aggregations.companies.filtered_companies.group_by_company.buckets[].additional_data.hits.hits[]._source.name"
````

### Company:bidder picker
#### Query
```json
GET deal.v2.3/_search
{
  "size": 0,
  "aggs": {
    "companies": {
      "nested": {
        "path": "companies.bidders"
      },
      "aggs": {
        "filtered_companies": {
          "filter": {
            "bool": {
              "must": [
                {
                  "wildcard": {
                    "companies.bidders.name.keyword": "*IBM*"
                  }
                }
              ]
            }
          },
          "aggs": {
            "group_by_company": {
              "terms": {
                "order": {
                  "_key": "asc"
                },
                "size": 20,
                "field": "companies.bidders.name.keyword"
              },
              "aggs": {
                "additional_data": {
                  "top_hits": {
                    "size": 1,
                    "_source": [
                      "companies.bidders.id",
                      "companies.bidders.mmgid",
                      "companies.bidders.name",
                      "companies.bidders.description"
                    ]
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
```
#### Curl command
```sh
curl  -s http://web:9200/deal/_search -H "Content-Type: application/json" -d "$(cat query.json)" | jq ".aggregations.companies.filtered_companies.group_by_company.buckets[].additional_data.hits.hits[]._source.name"
````

### Jurisdiction picker
#### Query
```json
{
  "size": 0,
  "aggs": {
    "TTE": {
      "nested": {
        "path": "timetableEvents"
      },
      "aggs": {
        "filtered_timetableEvents": {
          "filter": {
            "bool": {
              "should": [
                {
                  "wildcard": {
                    "timetableEvents.agencyJurisdiction.keyword": "a*"
                  }
                },
                {
                  "wildcard": {
                    "timetableEvents.agencyJurisdictionGroup.keyword": "a*"
                  }
                }
              ]
            }
          },
          "aggs": {
            "group_by_company": {
              "terms": {
                "order": {
                  "_key": "asc"
                },
                "size": 20,
                "field": "timetableEvents.agencyJurisdiction.keyword"
              },
              "aggs": {
                "top": {
                  "top_hits": {
                    "size": 1,
                    "_source": [
                      "timetableEvents.agencyJurisdiction",
                      "timetableEvents.agencyJurisdictionGroup"
                    ]
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
````
#### Curl command
```sh
curl  -s http://web:9200/deal/_search -H "Content-Type: application/json" -d "$(cat query.json)" | jq ".aggregations.TTE.filtered_timetableEvents.group_by_company.buckets[].top.hits.hits[]._source" 
````

### Target Sector picker
#### Query
```json
{
  "size": 0,
  "aggs": {
    "targets": {
      "nested": {
        "path": "companies.targets.sectors"
      },
      "aggs": {
        "filtered_targets": {
          "filter": {
            "bool": {
              "should": [
                {
                  "wildcard": {
                    "companies.targets.sectors.name.keyword": "a*"
                  }
                }
              ]
            }
          },
          "aggs": {
            "group_by_company": {
              "terms": {
                "order": {
                  "_key": "asc"
                },
                "size": 20,
                "field": "companies.targets.sectors.name.keyword"
              },
              "aggs": {
                "additional_data": {
                  "top_hits": {
                    "size": 1,
                    "_source": [
                      "companies.targets.sectors.code",
                      "companies.targets.sectors.name"
                    ]
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
````
#### Curl command
```sh
curl  -s http://web:9200/deal/_search -H "Content-Type: application/json" -d "$(cat query.json)" | jq ".aggregations.targets.filtered_targets.group_by_company.buckets[].additional_data.hits.hits[]._source"
````
### Bidder Sector picker
#### Query
```json
{
  "size": 0,
  "aggs": {
    "bidders": {
      "nested": {
        "path": "companies.bidders.sectors"
      },
      "aggs": {
        "filtered_bidders": {
          "filter": {
            "bool": {
              "should": [
                {
                  "wildcard": {
                    "companies.bidders.sectors.name.keyword": "a*"
                  }
                }
              ]
            }
          },
          "aggs": {
            "group_by_company": {
              "terms": {
                "order": {
                  "_key": "asc"
                },
                "size": 20,
                "field": "companies.bidders.sectors.name.keyword"
              },
              "aggs": {
                "additional_data": {
                  "top_hits": {
                    "size": 1,
                    "_source": [
                      "companies.bidders.sectors.code",
                      "companies.bidders.sectors.name"
                    ]
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
````
#### Curl command
```sh
curl  -s http://web:9200/deal/_search -H "Content-Type: application/json" -d "$(cat query.json)" | jq ".aggregations.bidders.filtered_bidders.group_by_company.buckets[].additional_data.hits.hits[]._source"
````
