---
title: 【ElasticStack】ElasticSearch分布式特性 与 Search机制
date: 2019-11-20 20:44:13
tags: [ElasticSearch, Kibana, ElasticStack, LogStash]
categories: ElasticStack
---


### 1. ElasticSearch的分布式特性
#### 1.1 分布式介绍
1. `ES`支持**集群模式**，即一个分布式系统。其好处主要有以下2个:
    1. **可增大系统容量**。比如：内存、磁盘的增加使得`ES`能够支持`PB`级别的数据；
    2. **提高了系统可用性**。即使一部分节点停止服务，集群依然可以正常对外服务。
2. `ES`集群由多个`ES实例`构成。
    + 不同集群通过**集群名字**来区分，通过配置文件`elasticsearch.yml`中的`cluster.name`可以修改，默认为`elasticsearch`
    + 每个`ES实例`的本质，其实是一个`JVM进程`，且有自己的名字，通过配置文件中的`node.name`可以修改。
<!-- more -->

#### 1.2 构建ES集群
``` bash
# 创建一个本地化集群my_cluster
bin/elasticsearch -Epath.data=node1 -Ecluster.name=my_cluster -Enode.name=node1 -d
bin/elasticsearch -Ehttp.port=8200 -Epath.data=node2 -Ecluster.name=my_cluster -Enode.name=node2 -d
bin/elasticsearch -Ehttp.port=7200 -Epath.data=node3 -Ecluster.name=my_cluster -Enode.name=node3 -d
```
> 可以通过cerebro插件可以看到，集群`my_cluster`中存在三个节点，分别为：`node1`、`node2`、`node3`

1. **`Cluster State`**：ES集群相关的数据，主要记录如下信息：
    + 节点信息：如节点名称、连接地址等
    + 索引信息：如索引名称、配置等
2. **`Master Node`**：**主节点**，可修改`cluster state`的节点。一个集群**只能有一个**。
    + `cluster state`存储于每个节点上，`master`维护最新版本并向其他从节点同步。
    + master节点是通过集群中所有节点**选举**产生的，可被选举的节点称为**`master-eligible节点`**
    + 通过配置`node.master:true`设置节点为可被选举节点(默认为true)
3. **`Cordinating Node`**：处理请求的节点。是所有节点的默认角色，且不能取消。
    + 路由请求到正确的节点处理，如：创建索引的请求到master节点。
4. **`Data Node`**：存储数据的节点，默认节点都是`data`类型。配置`node.data:true`。

#### 1.3 副本与分片
- 提高系统可用性：
    1. 服务可用性：集群
    2. 数据可用性：**副本**(Replication)
- 增大系统容量：**分片**(Shard)
- **分片**是`ES`能支持`PB级别数`据的基石：可在创建索引时指定
    1. 分片存储部分数据，可以分布于任意节点；
    2. 分片数在索引创建时指定，且后续不能更改，默认为5个；
    3. 有主分片和副本分片之分，以实现数据的高可用；
    4. 副本分片由主分片同步数据，可以有多个，从而提高数据吞吐量。
- **分片数的设定**很重要，需要提前规划好
    + **过小**会导致后续无法通过增加节点实现水平扩容
    + **过大**会导致一个节点分片过多，造成资源浪费，同时会影响查询性能
- 例如：在3个节点的集群中配置索引指定3个分片和1个副本（`index.number_of_shards:3`,
`index.number_of_replicas:1`），分布如下：
![](http:\\cdn.chaooo.top/Java/es_shard.jpg)

- 怎样增加节点或副本提高索引的吞吐量
    + **同时**增加新的节点**和**加新的副本，这样把新的副本放在新的节点上，进行索引数据读取的时候，并且读取，就会提升索引数据读取的吞吐量。

#### 1.4 ES集群状态 与 故障转移
- ES的**健康状态(`Cluster Health`)**分为三种：
    1. `Greed`，绿色。表示所有主分片和副本分片都正常分配；
    2. `Yellow`，黄色。表示所有主分片都正常分配，但有副本分片未分配；
    3. `Red`，红色。表示有主分片未分配。
- 可通过`GET _cluster/health`查看集群状态
    + 返回**集群名称**，**集群状态**，**节点数**，**活跃分片数**等信息。
    + 如果此时磁盘空间不够，name在创建新的索引的时候，主副分片都不会再分配，此时的集群状态会直接飙红，但此时依然可以访问集群和索引，也可以正常进行搜索。
    + 所以：**ES的集群状态为红色，不一定就不能正常服务**。

+ **故障转移 `Failover`**
    1. 当其余节点发现定时ping主节点master无响应的时候，集群状态转为Red。此时会发起master选举。
    2. 新master节点发现若有主分片未分配，会将副本分片提升为主分片，此时集群状态转为Yellow。
    3. 新master节点会将提升后的主分片生成新的副本，此时集群状态转为Green。整个故障转移过程结束。

#### 1.5 文档分布式存储
通过文档到分片的**映射算法**，使文档**均匀分布**到所有分片上，以充分利用资源。
- 文档对应分片计算公式：`shard = hash(routing)%number_of_primary_shards`
    + `hash`保证数据均匀分布在分片中
    + `routing`作为关键参数，默认为文档ID，也可自行指定
    + `number_of_primary_shards`为主分片数
- **主分片数一旦设定，不能更改**：`为了保证文档对应的分片不会发生改变`。
- 文档**创建**流程:
![](http:\\cdn.chaooo.top/Java/elastic1.jpg)

- 文档**读取**流程
![](http:\\cdn.chaooo.top/Java/elastic2.jpg)

- 文档**批量创建**流程
![](http:\\cdn.chaooo.top/Java/elastic3.jpg)

- 文档**批量读取**流程
![](http:\\cdn.chaooo.top/Java/elastic4.jpg)

#### 1.6 脑裂问题
- 在分布式系统中一个经典的网络问题
    + 当一个集群在运行时，作为`master`节点的`node1`的网络突然出现问题，无法和其他节点通信，出现网络隔离情况。那么`node1`自己会组成一个单节点集群，并更新`cluster state`；同时作为`data`节点的`node2`和`node3`因为无法和`node1`通信，则通过选举产生了一个新的`master`节点`node2`，也更新了`cluster state`。那么当`node1`的网络通信恢复之后，集群无法选择正确的`master`。
- 解决方案也很简单：
    + 仅在可选举的`master-eligible`节点数`>=quorum`的时候才进行`master`选举。
    + `quorum(至少为2)=master-eligible数量/2 + 1`。
    + 通过`discovery.zen.minimum_master_nodes`为`quorum`即可避免脑裂。

#### 1.7 Shards分片详解
1. 倒排索引一旦生成，不能更改。
    + 优点：
        1. 不用考虑并发写文件的问题，杜绝了锁机制带来的性能问题
        2. 文件不在更改，则可以利用文件系统缓存，只需载入一次，只要内存足够，直接从内存中读取该文件，性能高；
        3. 利于生成缓存数据(且不需更改)；
        4. 利于对文件进行压缩存储，节省磁盘和内存存储空间。
    + 缺点：在写入新的文档时，必须重构倒排索引文件，然后替换掉老倒排索引文件后，新文档才能被检索到，导致实时性差。
2. 解决文档搜索的实时性问题的方案：
    + 新文档直接生成新待排索引文件，查询时同时查询所有倒排索引文件，然后做结果的汇总即可，从而提升了实时性。
3. `Segment`
    + `Lucene`就采用了上述方案，构建的单个倒排索引称为`Segment`，多个`Segment`合在一起称为`Index`(`Lucene`中的`Index`)。在`ES`中的一个`shard`分片，对应一个`Lucene`中的`Index`。且`Lucene`有一个专门记录所有`Segment`信息的文件叫做`Commit Point`。
    + `Segment`写入磁盘的过程依然很耗时，可以借助文件系统缓存的特性。【先将`Segment`在内存中创建并开放查询，来进一步提升实时性】，这个过程在`ES`中被称为：`refresh`。
    + 在`refresh`之前，文档会先存储到一个缓冲队列`buffer`中，`refresh`发生时，将`buffer`中的所有文档清空，并生成`Segment`。
    + `ES`默认每`1s`执行一次`refresh`操作，因此实时性提升到了`1s`。这也是`ES`被称为近实时的原因（`Near Real Time`）。
4. `translog`文件
    + `translog`机制：当文档写入`buffer`时，同时会将该操作写入到`translog`中，这个文件会即时将数据写入磁盘，在6.0版本之后默认每个要求都必须落盘，这个操作叫做`fsync`操作。这个时间也是可以通过配置：`index.translog.*`进行修改的。比如每五秒进行一次`fdync`操作，那么风险就是丢失这`5s`内的数据。
5. 文档搜索实时性——`flush`(十分重要)
    + `flush`的功能，就是：将内存中的`Segment`写入磁盘，主要做如下工作：
        1. 将`translog`写入磁盘；
        2. 将`index bufffer`清空，其中的文档生成一个新的`Segment`，相当于触发一次`refresh`；
        3. 更新`Commit Point`文件并写入磁盘；
        4. 执行`fsync`落盘操作，将内存中的`Segment`写入磁盘；
        5. 删除旧的`translog`文件。
6. `refresh`与`flush`的发生时机
    + `refresh`：发生时机主要有以下几种情况：
        1. 间隔时间达到。
            + 通过`index.settings.refresh_interval`设置，默认为`1s`。
        2. `index.buffer`占满时。
            + 通过`indices.memory.index_buffer_size`设置，默认`JVM heap`的`10%`，且所有`shard`共享。
        3. `flush`发生时。会触发一次`refresh`。
    + `flush`：发生时机主要有以下几种情况：
        1. 间隔时间达到。
            + 5.x版本之前，通过`index.translog.flush_threshold_period`设置，默认30min。
            + 5.x版本之后，**ES强制每30min执行一次flush，不能再进行更改**。
        2. `translog`占满时。
            + 通过`index.translog.flush_threshold_size`设置，默认`512m`。且每个`Index`有自己的`translog`。
7. 删除和更新文档：
    + 删除：
        + `Segment`一旦生成，就不能更改，删除的时候，`Lucene`专门维护一个`.del`文件，记录所有已删除的文档。
        + `.del`文件上记录的是文档在`Lucene`中的`ID`，在查询结果返回之前，会过滤掉`.del`文件中的所有文档。
    + 更新：
        + 先删除老文档，再创建新文档，两个文档的`ID`在`Lucene`中的`ID`不同，但是在`ElasticSearch`中`ID`相同。
8. `Segment Merging`(合并)
    1. 随着`Segment`的增多，由于每次查询的`Segment`数量也增多，导致查询速度变慢；
    2. `ES`会定时在后台进行`Segment merge`的操作，减少`Segment`数量；
    3. 通过`force_merge api`可以手动强制做`Segment`的合并操作。





### 2. ElasticSearch的集群优化
#### 2.1 生产环境部署
1. 遵照官方建议设置所有系统参数。
    + 在ES的配置文件中elasticsearch.yml中，尽量只写必备的参数，其他可通过api进行动态设置，随着ES版本的不断升级，很多网上流传的参数，现在已经不再适用，所以不要胡乱复制。
    + 建议设置的基本参数有：
        1. `cluster.name`
        2. `node.name`
        3. `node.master/node.data/node.ingest`
        4. `network.host`: 建议显示指定为服务器的内网`ip`，切勿直接指定`0.0.0.0`，很容易直接从外部被修改`ES`数据。
        5. `discovery.zen.ping.unicast.hosts`: 设置集群其他节点地址，一般设置选举节点即可
        6. `discovery.zen.minimum_master_nodes`: 一般设置为`2`，有`3`个即可。
        7. `path.data/path.log`
        8. 除上述参数外，再根据需要增加其他的静态配置参数，如：`refresh`优化参数，`indices.memory.index_buffer_size`。
    + 动态设定的参数有transient(短暂的)和persistent(持续的)两种，前者在集群重启后会丢失，后者在集群重启后依然# 生效。二者都覆盖了yml中的配置，举例：

``` bash
# 使用transient和persistent动态设置ES集群参数
PUT /_cluster/Settings
{
 "persistent":{    # 永久
  "discovery.zen.minimum_master_nodes:2
 },
 "transient":{   # 临时
  "indices.store.throttle.max_bytes_per_sec":"50mb"
 }
}
```

2. 关于JVM内存设定
    + 每个节点尽量不要超多`31GB`。
    + 预留一半内存给操作系统，用来做文件缓存。ES的具体内存大小根据node要存储的数据量来估算，为了保证性能
        + 搜索类项目中：内存：数据量   ===>   1：16；
        + 日志类项目中：内存：数据量   ===>   1：48/96。

``` bash
假设现有数据1TB，3个node，1个副本，那么：
每个node存储(1+1)*1024 / 3 = 666GB,即700GB左右，做20%预留空间，每个node约存850GB数据。
此时：
如果是搜索类项目，每个node内存约为850/16=53GB，已经超过31GB最大限制；
而：31*16 = 496，意味着每个node最大只能存496GB的数据，则：2024/496=4.08...即至少需要5个节点。
如果是日志类项目，每个node最大能存:31*48=1488GB,则：2024/1488=1.36...，则三个节点已经够了。
```


#### 2.2 写性能优化
在写上面的优化，主要是增大写的吞吐量——`EPS(Event Per Second)`
- 优化方案：
    1. `Client`：多线程写，批量写`bulk`；
    2. `ES`：在高质量数据建模的前提下，主要在`refresh`、`translig`和`flush`之间做文章。

1. 降低`refresh`写入内存的频率：
    1. 增大`refresh_interval`，降低实时性，增大每次`refresh`处理的文件数，默认1s。可以设为-1s，禁止自动`refresh`。
    2. 增大`index` `buffer`大小，参数为：`indices.memory.index_buffer_size`。此为静态参数，需设定在`elasticsea.yml`中，默认`10%`
2. 降低translog写入磁盘频率，同时会降低容灾能力：
    1. `index.translog.durability`：设为`async`；
    2. `index.translog.sync_interval`。设置需要的大小如：120s  =>  每120s才写一次磁盘。
    3. `index.translog.flush_threshold_size`。默认512m。即当`translog`大小超过此值，会触发一次`flush`，可以调大避免`flush`过早触发。
3. 在`flush`方面，从6.x开始，ES固定每30min执行一次，所以优化点不多，一般都是ES自动完成。
4. 其他：
    1. 将副本数设置为0，在文档全部写完之后再加副本；
    2. 合理设计`shard`数，保证`shard`均匀地分布在所有`node`上，充分利用`node`资源：
        + `index.routing.allocation.total_shards_per_node`：限定每个索引在每个`node`上可分配的主副分片数，
        + 如：有`5`个`node`，某索引有`10`个主分片，`1`个副本(`10`个副分片)，则：`20/5=45`,但是实际要设置为`5`，预防某个`node`下线后分片迁移失败。

> 写性能优化，主要还是index级别的设置优化。
> 一般在refresh、translog、flush三个方面进行优化；



#### 2.3 读性能优化
- 主要受以下几方面影响：
    1. 数据模型是否符合业务模型？
    2. 数据规模是否过大？
    3. 索引配置是否优化？
    4. 查询运距是否优化？
1. 高质量的数据建模
    1. 将需通过`cripte`脚本动态计算的值，提前计算好作为字段存入文档中；
    2. 尽量使数据模型贴近业务模型
2. 根据不同数据规模设定不同的`SLA`(服务等级协议)，万级数据和千万级数据和亿万级数据性能上肯定有差异；
3. 索引配置优化
    1. 根据数据规模设置合理的分片数，可通过测试得到最适合的分片数；
    2. 分片数并不是越多越好
4. 查询语句优化
    1. 尽量使用`Filter`上下文，减少算分场景(`Filter`有缓存机制，能极大地提升查询性能)；
    2. 尽量不用`cript`进行字段计算或算分排序等；
    3. 结合`profile`、`explain API`分析慢查询语句的症结所在，再去优化数据模型。

#### 2.4 其他优化点
1. 如何设定`shard`数？
    + `ES`的性能基本是线性扩展的，因此，只需测出一个`shard`的性能指标，然后根据实际的性能需求就可算出所需的`shard`数。
    + 测试一个`shard`的流程如下：
        1. 搭建与生产环境相同配置的单节点集群；
        2. 设定一个单分片`0`副本的索引；
        3. 写入实际生产数据进行测试，获取（写性能指标）；
        4. 针对数据进行查询操作，获取（读性能指标）。
2. 压力测试工具，可以采用`ES`自带的`esrally`，从经验上讲：
    + 如果是搜索引擎场景，单`shard`大小不超过`15GB`；
    + 如果是日志分析场景，单`shard`大小不超过`50GB`；
    + 估算索引的总数据大小，除以上述单`shard`大小，也可得到经验上的分片数。

#### 2.5 ES集群监控
使用官方免费插件`X-pack`。
1. 安装与启动：

``` bash
# X-pack的安装
cd ~/elasticsearch-6.1.1
bin/elasticsearch-plugin install x-pack
# 
cd ~/kibana-6.1.1
bin/kibana-plugin indtall x-pack
```
之后重启`ES`集群即可。
在`kibana`的界面可以看到新增了工具，使用`Monitoring`进行集群监控。





### 3. ElasticSearch中Search的运行机制
- `Search`执行的时候，实际分为两个步骤执行：
    1. `Query`阶段：搜索
    2. `Fetch`阶段：获取

#### 3.1 Query—Then—Fetch：
若集群`my_cluster`中存在三个节点node1、node2、node3，其中`master`为node1，其余的为`data`节点。
+ `Query`阶段:
![](http:\\cdn.chaooo.top/Java/elastic-q.jpg)

+ `Fetch`阶段: 
![](http:\\cdn.chaooo.top/Java/elastic-f.jpg)


#### 3.2 相关性算分：
**相关性算分在`shard`和`shard`之间是相互独立的**。也就意味着：同一个单词`term`在不同的`shard`上的`TDF`等值也可能是不同的。得分与`shard`有关。
当文档数量不多时，会导致相关性算分严重不准的情况发生。
- 解决方案：
    1. 设置分片数为`1`个，从根本上排除问题。（此方案只适用于百万/少千万级的少量数据）
    2. 使用`DFS Query-then-Fetch`查询方式。
- `DFS Query-then-Fecth`：
    + 在拿到所有文档后，再重新进行完整的计算一次相关性得分，耗费更多的CPU和内存，**执行性能也较低**。所以也不推荐。

``` bash
# 使用DLS Query-then-Fetch进行查询：
GET my_index/_search？search_type=dfs_query_then_fetch
{
 "query":{
  "match":{
   ...
  }
 }
}
```

#### 3.3 排序相关：
默认采用相关性算分结果进行排序。可通过`sort`参数自定义排序规则，如：
``` bash
# 使用sort关键词进行排序
GET my_index/_search
{
 "sort":{    # 关键词
  "birth":"desc"
 }
}
# 或使用数组形式定义多字段排序规则
GET my_index/_search
{
 "sort":[    # 使用数组
  {
   "birth":{
    "order":"asc"
   }
  },
  {
   "age":{
    "order":"desc"
   }
  }
 ]
}
```

1. 直接按数字/日期排序，如上例中`birth`
2. 按字符串进行排序：字符串排序较特殊，因为在`ES`中有`keyword`和`text`两种：

``` bash
# 直接对text类型进行排序
GET my_index/_search
{
 "sort":{
  "username":"desc"    # 针对username字段进行倒序排序
 }
}
#
# 针对keyword进行排序
GET my_index/_search
{
 "sort":{
  "username.keyword":"desc"    # 针对username的子类型keyword类型进行倒叙排序
 }
}
```

##### 3.3.1 关于fielddata和docvalues:
排序的实质是对字段的原始内容排序的过程，此过程中**倒排索引无法发挥作用**，需要用到**正排索引**。即：通过文档`ID`和字段得到原始内容。
- ES提供2中实现方式：
    - `Fielddata`。    默认禁用。
    - `DocValues`。    默认启用，除了text类型。

| 对比 | Fielddata | DocValues |
|------|----------|------------|
|创建时机|搜索时即时创建|创建索引时创建，和倒排索引创建时间一致|
|创建位置|JVM Heap|磁盘|
|优点|不占用额外磁盘空间|不占用Heap内存|
|缺点|文档较多时，同时创建会花费过多时间，占用过多Heap内存|减慢索引的速度，占用额外的磁盘空间|

##### 3.3.2 Fielddata的开启:
`Fielddata`**默认关闭**，可通过如下api进行开启，且在后续使用时随时可以开启/关闭：
- 使用场景：**一般在对分词做聚合分析的时候开启**。

``` bash
# 开启字段的fielddata设置
PUT my_index/_mapping/doc
{
 "properties":{
  "username":{
   "type":"text",
   "fielddata":true    # 关键词
  }
 }
}
```
            
#### 3.3.3 Docvalues的关闭
`Docvalues`**默认开启**，可在创建索引时关闭，且之后不能再打开，要打开只能做reindex操作。
- 使用场景：当明确知道，不会使用这个字段排序或者不做聚合分析的时候，可关闭doc_values，减少磁盘空间的占用。

``` bash
# 关闭字段的docvalues设置
PUT my_index
{
 "mappings":{
  "doc":{
   "properties":{
    "username":{
     "type":"keyword",
     "doc_values":false    # 关键词
    }
   }
  }
 }
}
```
            

#### 3.4 分页与遍历
ES提供了三种方式来解决分页和遍历的问题： `from/size`，`scroll`，`search_after`。

##### 3.4.1 from/size
- `from`：指明开始位置；
- `size`：指明获取总数

``` bash
# 使用from——size
GET my_index/_search
{
 "from":1,    # 从第2个开始搜索
 "size":2     # 获取2个长度
}
```

1. 经典问题：**深度分页**。
    - 问题：如何在数据分片存储的情况下， 获取前1000个文档？
    - 答案：
        + 先从每个分片上获取前1000个文档， 然后由处理节点聚合所有分片的结果之后，再排序获取前1000个文档。
        + 此时页数越深，处理的文档就越多，占用的内存就越大，耗时就越长。这就是深度分页问题。
        + 为了尽量避免深度分页为题，ES通过设定`index.max_result_window`限定最多到10000条数据。
2. 在设计分页系统时，有一个分页数十分重要：
    - `total_page=(total + page_size -1) / page_size`
    - 总分页数= (文档总数+认为设定的文档大小-1) / 人为设定的文档大小
    - 但是在搜索引擎中的意义并不大，因为如果排在前面的结果都不能让用户满意，那么越往后，越不能让用户满意。

##### 3.4.2 scroll
- 遍历文档集的`API`，以快照的方式来避免深度分页问题。
    1. 不能用来做实时搜索，因为数据不是实时的；
    2. 尽量不用复杂的`sort`条件，使用`_doc`最高效；
    3. 使用比较复杂。
- 步骤：
    1. 发起一个`scroll search`，会返回后续会用到的`_scroll_id`
    2. 调用`scroll search`的`api`，获取文档集合，不断迭代至返回`hits`数组为空时停止
    3. 之后不断返回新的`_scroll_id`，使用新的`_scroll_id`进行查询，直到返回数组为空。
    4. 当不断的进行迭代，会产生很多`scroll`，导致大量内存被占用，可以通过`clear api`进行删除

``` bash
# 发起一个scroll search
GET my_index/_search?scroll=5m # 该快照的有效时间为5min
{
 "size"1    # 指明每次scroll返回的文档数
}
#
# 调用scroll search 的api，获取文档集合
POST _search/scroll
{
 "scroll":"5m",    # 指明有效时间
 "scroll_id":"xxxxxx"    # 上一步返回的_scroll_id
}
#
# 使用clear api对scroll进行删除
DELETE /_search/scroll
{
 "scroll_id":[
   "xxxxxx",    # _scroll_id
   "xxxxxx",    # _scroll_id
   ......
 ]
}
#
# 删除所有的scroll
DELETE /_search/scroll/_all
```

##### 3.4.3 search_after
避免深度分页的性能问题，提供实时的下一页文档获取功能。
- 缺点：不能使用from参数，即：不能指定页数。且只能下一页，不能上一页。
- 使用步骤：
    1. 第一步：正常搜索，但是要指定sort值，并保证值唯一：
    2. 第二步：使用上一步最后一个文档的sort值进行查询：

``` bash
# 第一步，正常搜索
GET my_index/_search
{
 "size":1,
 "sort":{
  "age":"desc",
  "_id":"desc"
 }
}
#
# 第二步，使用sort值进行查询
GET my_index/_search
{
 "size":1,
 "search_after":[28,"2"],# 28,"2"，是上一次搜索返回的sort值
 "sort":{
  "age":"desc",
  "_id":"desc"
 }
}
```

##### 3.4.4 如何避免深度分页问题:
这个问题目前连google都没能解决，所以只能最大程度避免，通过唯一排序值定位每次要处理的文档数都控制在size内：
- 应用场景：
    1. from/size:需实时获取顶部的部分文档，且需自由翻页（实时）；
    2. scroll:需全部文档，如：导出所有数据的功能（非实时）；
    3. search_after:需全部文档，不需自由翻页（实时）。


