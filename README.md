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

4.) API layer to protect data inside elasticsearch.

It is very common for people to use elasticsearch in a public facing production environment. Companies like yelp and wikimedia are big users of elasticsearch. So the one thing you NEVER want to give access to is a direct contact into your elasticsearch cluster. For this reason you will want to build an API layer. This can be as simple as a NodeJS or PHP application but always make sure that your data is protected. You can take this API and point it to an internal load balancer so that way the public never has access to your data directly.

5.) Avoiding split brain!

Split-brain is a catastrophic event for ElasticSearch clusters. Once a cluster is split into two and and each side has elected a new master, they will diverge and cannot be rejoined without killing one half of the cluster. 

```
discovery.zen.minimum_master_nodes: set this to at least N/2+1 on clusters with N > 2, where N is the number of master nodes in the cluster.

In the tutorial created for this I have 3 master nodes so I will set the following.

discovery.zen.minimum_master_nodes: 2
```

minimum_master_nodes ensures that a node has to see that number of master eligible nodes (nodes that could potentially take over as masters) to be operational. Otherwise, the node goes into an error state and refuses to accept requests, as it considers itself to be split off from the cluster.

6.) Install these plugins and always monitor!

```
./bin/plugin --install mobz/elasticsearch-head
./bin/plugin --install lukas-vlcek/bigdesk
./bin/plugin --install lmenezes/elasticsearch-kopf/1.5.7
```

7.) Tag your nodes properly.

Always give your nodes a good name and a good unique cluster name. This way you can at a glance know exactly what a node does and which cluster it's in. Typically i'll give them names like client-node1,2,3 etc.

* Citation : http://asquera.de/opensource/2012/11/25/elasticsearch-pre-flight-checklist/
