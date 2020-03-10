## PARR-1542
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

### Sector picker
#### Query
```json
````
#### Curl command
```sh
````
