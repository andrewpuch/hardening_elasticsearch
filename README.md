# Hardening Elasticsearch
Building a elasticsearch cluster and hardening it for production use.

1.) Setting field data cache.

Edit /etc/elasticsearch/elasticsearch.yml and add the following line. Now this line you can configure to suit your environment but 75% is a safe value.

Basically this will limit the cache when used for sorting or aggregating data. By default it is unbounded...the reason this is important because if you have massive queries you can overload your heap and cause data nodes to become unbalanced and your cluster to be out of sync.
```
indices.fielddata.cache.size: 75%
```

2.) Updating heap size.

Edit /etc/sysconfig/elasticsearch and go to the line with ES_HEAP_SIZE and you will want to set this value to be 50% of the available RAM in your system and NO MORE than 31g. This is important because you will be able to allocate more cache to the JVM and is especially useful in high volume applications. The default is 250mb.

3.) Seperating out your nodes.
3a.) Data nodes

Edit /etc/elasticsearch/elasticsearch.yml and set these values.

```
node.master: false
node.data: true
```

3b.) Client nodes - The client node acts as sort of a load balancer to the cluster. It performs fetching data from nodes, aggregating results, etc.

Edit /etc/elasticsearch/elasticsearch.yml and set these values.

```
node.master: false
node.data: false
```

3c.) Master nodes

Edit /etc/elasticsearch/elasticsearch.yml and set these values. The master node is the coordinator of your cluster. It is recommended to have 3 nodes who's job is to only be master.

```
node.master: true
node.data: false
```

4.) Install these plugins and always monitor!

```
./bin/plugin -install mobz/elasticsearch-head
./bin/plugin -install lukas-vlcek/bigdesk
./bin/plugin --install lmenezes/elasticsearch-kopf/1.5.7
```
