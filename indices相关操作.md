原文地址：https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-create-index.html

elasticsearch版本：6.5

# 1、查看所有indices
```
GET /_cat/indices?v
```
 得到的响应：
```
health status index uuid pri rep docs.count docs.deleted store.size pri.store.size
```
没有具体的值，表示现在elasticsearch中还没有索引。

# 2、创建索引

创建一个名为“customer”的索引，并且再列出来：
```
PUT /customer?pretty
GET /_cat/indices?v
```
第一行命令是用PUT方法创建一个名为“customer”的索引。我们只需在调用的末尾附加pretty命令它可视化地打印JSON响应（如果有）

响应为：
```
health status index     uuid   pri rep docs.count docs.deleted store.size pri.store.size
yellow  open   customer 95SQ4   5   1       0          0           260b        260b
```
第二个命令的结果告诉我们，我们现在有1个名为customer的索引，它有5个主分片和1个副本（默认值），其中包含0个文档。

您可能还会注意到客户索引中有一个黄色的健康标签。回想一下我们之前的讨论，黄色意味着一些副本尚未分配。此索引发生这种情况的原因是，默认情况下，ElasticSearch为此索引创建了一个副本。因为目前只有一个节点在运行，所以在另一个节点加入集群之前，还不能分配一个副本（为了高可用性）。一旦该副本分配到第二个节点上，该索引的运行状况将变为绿色。

# 3、索引和查询文档

现在我们把一些东西放到客户索引中。我们将在客户索引中索引一个简单的客户文档，其ID为1，如下所示：
```
PUT /customer/_doc/1?pretty
{
    "name":"John Doe"
}
```
得到的响应为：
```
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```
从上面，我们可以看到在客户索引中成功地创建了一个新的客户文档。文档还有一个内部ID 1，我们在索引时指定了它。

需要注意的是，ElasticSearch并不要求您在索引文档之前先显式创建索引。在上一个示例中，如果客户索引之前不存在，那么ElasticSearch将自动创建该索引。

现在让我们检索刚才索引的文档：
```
GET /customer/_doc/1?pretty
```
得到的响应为：
```
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "_source" : { "name": "John Doe" }
}
```
除了一个字段之外，这里没有发现任何异常的地方，说明我们找到了一个具有请求的ID 1的文档和另一个字段_source，它返回了我们从上一步索引的完整JSON文档。

# 4、删除索引

现在让我们删除刚才创建的索引：
```
DELETE /customer?pretty
GET /_cat/indices?v
```
得到的响应为：
```
health status index uuid pri rep docs.count docs.deleted store.size pri.store.size
```
这说明我们删除索引成功了，我们回到了没有索引的初始状态。

在我们继续之前，让我们再仔细看看我们迄今为止学到的一些API命令：
```
PUT /customer
PUT /customer/_doc/1
{
  "name": "John Doe"
}
GET /customer/_doc/1
DELETE /customer
```
如果我们仔细研究上述命令，我们实际上可以看到在ElasticSearch中如何访问数据的模式。这种模式可以概括如下：
```
<HTTP Verb> /<Index>/<Type>/<ID>
```
这种REST访问模式在所有API命令中都非常普遍，如果您能简单地记住它，那么您将在掌握ElasticSearch方面有一个很好的开端。
