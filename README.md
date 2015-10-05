# Hardening Elasticsearch
Building a elasticsearch cluster and hardening it for production use.

Edit /etc/elasticsearch/elasticsearch.yml and add the following line. Now this line you can configure to suit your environment but 75% is a safe value.

Basically this will limit the cache when used for sorting or aggregating data. By default it is unbounded...the reason this is important because if you have massive queries you can overload your heap and cause data nodes to become unbalanced and your cluster to be out of sync.
```
indices.fielddata.cache.size: 75%
```
