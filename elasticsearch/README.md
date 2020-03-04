## ElasticSearch instance & Kibana URLs

| Service | Url Live | Url Aslive |
| :---- | :---- | :---- |
| Elasticsearch | https://vpc-live-parrelasticsearch-uzvrnmaslcmmcfubjdfv6e76ay.eu-west-1.es.amazonaws.com | https://vpc-aslive-parrelasticsearch-bjhtyuy57emg6iswxqqocbweui.eu-west-1.es.amazonaws.com |
| Kibana | https://vpc-live-parrelasticsearch-uzvrnmaslcmmcfubjdfv6e76ay.eu-west-1.es.amazonaws.com/_plugin/kibana/ | https://vpc-aslive-parrelasticsearch-bjhtyuy57emg6iswxqqocbweui.eu-west-1.es.amazonaws.com/_plugin/kibana |

To connect to ASLIVE environment use links given above. Instruction how to connect to LIVE environment is given below.


## ElasticSearch & Kibana LIVE access

This is a proxy that will sign requests to Elasticsearch (required for connecting to Elasticsearch instances hosted on AWS).
**Before entering any commands contained within this documents make sure that you are in the *elasticsearch* directory.**

``` bash
docker build -t es .
```
You can run it in interactive mode. It will ask you to authorize. When you stop container (ctrl+C) it will be removed automatically.
``` bash
docker run --rm -it -p 9200:9200 -e ACCOUNT=mmgprod -e TARGET_ES=https://vpc-live-parrelasticsearch-uzvrnmaslcmmcfubjdfv6e76ay.eu-west-1.es.amazonaws.com es
```

Your session is valid for 1h and afterwards you have to terminate and run it again.

| Service | Url |
| :---- | :---- |
| elasticsearch | <http://localhost:9200> |
| kibana | <http://localhost:9200/_plugin/kibana> |

# Advanced
To connect to Elasticsearch query in non-interactive mode can be used. Rerun it each time you need without passing all the parameters (including credentials) each time. In the command below OneLogin Protect is chosen as MFA device.

```
docker build -t es .
docker run -p 9200:9200 --name esa -e EMAIL=john.smith@acuris.com -e MMG_PASSWORD=password -e MMGAWS_MFA_INDEX=1 -e ACCOUNT=mmgprod -e TARGET_ES=https://vpc-live-parrelasticsearch-uzvrnmaslcmmcfubjdfv6e76ay.eu-west-1.es.amazonaws.com es
```
You can use following commands to manage your container.
```
docker stop esa
docker start esa
docker restart esa
```
