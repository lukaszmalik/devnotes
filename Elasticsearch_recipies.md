# PARR-1542
## Agencies
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
