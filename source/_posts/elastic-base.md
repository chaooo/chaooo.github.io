---
title: 「ElasticStack」ElasticSearch入门
date: 2018-11-12 20:42:59
tags: [ElasticSearch, Kibana, ElasticStack, LogStash]
categories: ElasticStack
---


### 1. 概述
#### 1.1 ElasticStack特点
1. 使用门槛低，开发周期短，上线快
2. 性能好，查询快，实时展示结果
3. 扩容方便，快速支撑增长迅猛的数据
<!-- more -->

#### 1.2 ElasticStack各组件作用
1. **`Beats`**：数据采集
2. **`LogStash`**: 数据处理
3. **`ElasticSearch`**(核心引擎): 数据存储、查询和分析
4. **`Kibana`**: 数据探索与可视化分析<!-- more -->

#### 1.3 ElasticStack使用场景
+ 搜索引擎、日志分析、指标分析

#### 1.4 ElasticStack安装启动
1. `ElasticSearch`启动：解压到安装目录，启动`bin/elasticsearch`（默认端口:`http:\\localhost:9200`, 加参数`-d`后台启动）
2. `ElasticSearch`集群：

``` bash
bin/elasticsearch -d 
bin/elasticsearch -Ehttp.port=8200 -Epath.data=node2 -d
bin/elasticsearch -Ehttp.port=7200 -Epath.data=node3 -d
```

3. Kibana启动：解压到安装目录，启动`bin/kibana`（默认端口:`http:\\localhost:5601`）

#### 1.5 ElasticSearch常见术语
1. `Document`(文档)：用户存储在ES中的数据文档
2. `Index`(索引)：由具有相同字段的文档列表组成
3. `field`(字段)：包含具体数据
4. `Node`(节点)：一个ES的实例，构成clister的单元
5. `Cluster`(集群)：对外服务的一个/多个节点

#### 1.6 Document介绍
1. 常用数据类型：字符串、数值型、布尔型、日期型、二进制、范围类型
2. 每个文档都有一个唯一`ID`标识。（可以自行指定，也可由ES自动生成）
3. 元数据，用于标注文档的相关信息：
    + `_index`：文档所在的索引名
    + `_type`：文档所在的类型名(后续的版本中type这个概念将会被移除，也不允许一个索引中有多个类型)
    + `_id`：文档唯一标识
    + `_source`：文档的原始JSON数据，可从这获取每个字段的内容
    + `_all`：整合所有字段内容到该字段。（默认禁用）
    + `_version`：文档字段版本号，标识被操作了几次
4. `Index`介绍：
    + 索引中存储相同结构的文档，且每个index都有自己的Mapping定义，用于定义字段名和类型；
    + 一个集群中可以有多个inex，类似于可以有多个table。
5. `RESTful API`两种交互方式：
    1. CURL命令行：curl -XPUT xxx
    2. Kibana DevTools————PUT xxx{ }
6. `Index API`： 用户创建、删除、获取索引配置等。
    1. 创建索引：
        + `PUT /test_index` #创建一个名为`test_index`的索引
    2. 查看索引：
        + `GET _cat/indices` #查看所有的索引
    3. 删除索引：
        + `DELETE /test_index` #删除名为`test_index`的索引

#### 1.7 CRUD操作（交互基于Kibana DevTools）
1. 创建文档

``` bash
# 创建ID为1的Document
PUT /test_index/doc/1
{
 "username":"alfred",
 "age":"24"
}
#
# 不指定ID创建Document(ID会自动生成)
POST /test_index/doc
{
 "username":"buzhiding",
 "age":"1"
}
```

2. 查询文档：

``` bash
# 查看名为test_index的索引中id为1的文档
GET /test_index/doc/1
#
# 查询所有文档：
# 查询名为test_index的索引中所有文档,用到endpoint：_search，默认返回符合的前10条
# term和match的区别：term完全匹配，不进行分词器分析；match模糊匹配，进行分词器分析，包含即返回
GET /test_index/doc/_search
{
 "query":{
  "term":{
   "_id":"1"
  }
 }
}
```

3. 批量操作文档：

``` bash
# 批量创建文档，用到endpoint：_bulk
# index和create的区别，如果文档存在时，使用create会报错，而index会覆盖
POST _bulk
{"index":{"_index":"test_index","_type":"doc","_id":"3"}}
{"username":"alfred","age":"20"}
{"delete":{"_index":"test_index","_type":"doc","_id":"1"}}
{"update":{"_id":"2","_index":"test_index","_type":"doc"}}
{"doc":{"age":"30"}}
#
# 批量查询文档，使用endpoint:_mget
GET _mget
{
 "doc":[
  {
   "_index":"test_index",
   "_type":"doc",
   "_id":"1"
  },
  {
   "_index":"test_index",
   "_type":"doc",
   "_id":"2"
  }
 ]
}
```

4. 删除文档：

``` bash
# 根据搜索内容删除文档,使用endpoint:_delete_by_query
POST /test_index/doc/_delete_by_query
{
 "query":{
  "match":{
   "username":"buzhiding"
  }
 }
}
# 
# 删除整个test_index的索引中的文档,依然使用endpoint:_delete_by_query
POST /test_index/doc/_delete_by_query
{
 "query":{
  "match_all":{}
 }
}
```



### 2. ElasticSearch倒排索引与分词
#### 2.1 倒排索引
1. 正排索引和倒排索引
    + 正排索引：文档ID ---> 文档内容
    + 倒排索引：单词---> 文档ID列表
2. 倒排索引组成：（单词词典，倒排列表）
    1. 单词词典（`Term Dictionary`）
        + 记录所有文档的单词，记录了单词到倒排列表的关联信息，一般使用`B+Tree`实现。
    2. 倒排列表（`Posting List`）
        + 记录单词对应的文档集合，由倒排索引项`Posting List`组成。
        + 倒排索引项：
            1. 文档`ID`：用于获取原始信息。
            2. 词频`TF`：记录该单词在该文档中的出现次数，用于计算相关性得分。
            3. 位置`Position`：记录单词在文档中的分词位置(多个)，用于词语搜索。
            4. 偏移`Offset`：记录单词在文档的开始和结束位置，用于高亮显示。

#### 2.2 分词Analysis
分词：将文本转换成一系列单词`Term/Token`的过程，也可称作文本分析，ES中叫作：Analysis。
+ 一些概念：
    1. `Token`(词元)：全文搜索引擎会用某种算法对要建索引的文档进行分析， 从文档中提取出若干Token(词元)。
    2. `Tokenizer`(分词器)：这些算法叫做Tokenizer(分词器)
    3. `Token Filter`(词元处理器)：这些Token会被进一步处理， 比如转成小写等， 这些处理算法被称为TokenFilter(词元处理器)
    4. `Term`(词)：被处理后的结果被称为Term(词)
    5. `Character Filter`(字符过滤器)：文本被Tokenizer处理前可能要做一些预处理， 比如去掉里面的HTML标记， 这些处理的算法被称为Character Filter(字符过滤器)
    6. `Analyzer`(分析器)：这整个的分析算法被称为Analyzer(分析器)，由Tokenizer(分词器)和Filter(过滤器)组成
+ ES有很多**内置`Analyzer`**,比如：
    1. `standard`：按单词边界划分、支持多语言、小写处理、移除大部分标点符号，支持停用词
    2. `whitespace`：空格为分隔符
    3. `simple`：按非字母划分、小写处理
    4. `stop`：类似简单分词器，同时支持移除停用词(the、an、的、这等)
    5. `keyword`：不分词
    6. `pattern`：通过正则表达式自定义分隔符，默认\w+，即：非字词的符号作为分隔符
+ 第三方analyzer插件：常用的**中文分词器**有：
    1. IK：实现中英文分词，支持多模式，可自定义词库，支持热更新分词词典。
    2. jieba。python中流行，支持繁体分词、并行分词，可自定义词典、词性标记等。
+ ES提供了一个测试分词的API接口，使用`endpoint：_analyze`，不指定分词时，会使用默认的`standard`

``` bash
# 指定分词器进行分词测试
POST _analyze
{
 "analyzer":"standard",
 "text":"hello world!"
}
# 
# 直接指定索引中字段：使用username字段的分词方式对text进行分词。
POST test_index/_analyze
{
 "field":"username",
 "text":"hello world!"
}
# 
# 自定义分词器，自定义Tokenizer、filter、等进行分词：
POST _analyze
{
 "tokenizer":"standard",
 "filter":["lowercase"],
 "text":"Hello World!"
}
```



### 3. ElasticSearch的Mapping
#### 3.1 Mapping简介
Mapping：类似于数据库中的表结构
1. 主要作用如下：
    1. 定义`Index`下的`Field Name`；
    2. 定义`Field`的类型，如：数值型、字符串型、布尔型等；
    3. 定义倒排索引的相关配置，如：是否有索引，记录position等。
2. 获取一个`mapping`，使用`endpoint：_mapping`，例如：
    + `GET /test_index/_mapping`

#### 3.2 自定义Mapping
1. 使用`mappings`进行自定义`mapping`。
2. `Mapping`中的字段类型一旦设定之后，**禁止直接修改**。
    + 因为`Luence`事先的倒排索引生成后不能修改。
    + 如果一定要改，可以重新建立新的索引，然后对应修改`mapping`，之后将之前的数据进行`reindex`操作，导入新的文档。
3. 自定义`mapping`时允许新增字段。通过`dynamic`参数进行控制字段的新增，`dynamic`有三种配置：
    + `true`：默认配置，允许自动新增字段；
    + `false`：不允许自动新增字段，文档可以正常写入，但不能进行查询等操作；
    + `strict`：严格模式。文档不能写入，写入会报错。

``` bash
# 创建名为my_index的索引，并自定义mapping
# 使用dynamic参数控制字段的新增
PUT my_index
{
 "mappings":{        # 关键字
  "doc":{            # 类型名
   "dynamic":false,  # 设置为false，索引不允许新增字段
   "properties":{    # 字段名称及类型定义
    "title":{
     "type":"text"   # 字段类型
    },
    "name":{
     "type":"keyword"
    },
    "age":{
     "type":"integer"
    } 
   }
  }
 }
}
```

#### 3.3 copy_to的使用
将该字段的值复制到目标字段，类似于6.0版本之前的`_all`的作用。且不会出现在`_source`，一般只用来进行搜索。
``` bash
# copy_to的使用
PUT my_index
{
 "mappings":{
  "doc":{
   "properties":{
    "first_name":{
     "type":"text",
     "copy_to":"full_name"
    },
    "last_name":{
     "type":"text",
     "copy_to":"full_name"
    },
    "full_name":{
     "type":"text"
    }
   }
  }
 }
}
# 
# 向索引写入数据
PUT my_index/doc/1
{
 "first_name":"John",
 "last_name":"Smith"
}
# 
# 查询索引my_index中full_name同时包含John 和 Smith的数据
GET my_index/_search
{
 "query":{
  "match":{
   "full_name":{
    "query":"John Smith",
    "operator":"and"
   }
  }
 }
}
```

#### 3.4 index参数的使用
控制当前字段是否为索引，默认`true`，当设置为`false`的时候，不进行记录，此时该字段不能被搜索
``` bash
# index参数的使用
PUT my_index
{
 "mappings":{
  "doc":{
   "properties":{
    "cookie":{
     "type":"text",
     "index":false    # 设置为false，该字段不能被搜索
    }
   }
  }
 }
}
```

此时在进行数据写入和查询，不能进行该字段搜索。一般用来进行不想被查询的私密信息设置，如身份证号，电话号码等：
``` bash
# 向使用了index参数的字段写入信息
PUT my_index/doc/1
{
 "cookie":"name=alfred"
}
```

#### 3.5 index_options参数的使用：
控制倒排索引记录的内容，有如下四种配置：
1. `docs`：只记录文档ID
2. `freqs`：记录文档ID和词频TF
3. `positions`：记录文档ID、词频TF和分词位置
4. `offsets`：记录文档ID、词频TF、分词位置和偏移
> 其中：text类型默认的配置是positions，其他的比如integer等类型默认为docs，目的是为了节省空间。

``` bash
# index_options参数的使用
PUT my_index
{
 "mappings":{
  "doc":{
   "properties":{
    "cookie":{
     "type":"text",
     "index_options":"offsets"  # 记录文档ID、词频TF、分词位置和偏移
    }
   }
  }
 }
}
```

#### 3.6 null_value参数的使用：
当字段遇到空值`null`时的处理策略。默认为`null`，即跳过。此时ES会忽略该值，可通过修改进行默认值的修改：
``` bash
# 使用null_value修改ES遇到null值时的默认返回值
PUT my_index
{
 "mappings":{
  "doc":{
   "properties":{
    "cookie":{
     "type":"keyword",
     "null_value":"NULL"    # 当遇到空值null的时候，返回一个字符串形式的NULL
    }
   }
  }
 }
}
```

#### 3.7 Field字段的数据类型：
1. 核心数据类型
    1. 字符串型：`text`(分词)，`keyword`(不分词)
    2. 数值型：`long,integer,short,byte,double,float,half_float,scaled_float`
    3. 日期类型：`date`
    4. 布尔类型：`boolean`
    5. 二进制类型：binary
    6. 范围类型：`integer_range,float_range,long_range,double_range,date_range`
2. 复杂数据类型
    1. 数组类型：`array`
    2. 对象类型：`object`
    3. 嵌套类型：`nested object`
3. 地理位置数据类型
    1. 点：`geo-point`
    2. 形状：`geo-shape`
4. 专用类型
    1. 记录ip地址：`ip`
    2. 实现自动补全：`completion`
    3. 记录分词数：`token_count`
    4. 记录字符串hash值：`murmur3`
    5. `perclator`
    6. `join`
5. 多字段特性：
    + ES允许对同一个字段采用不同的配置，如：分词。举例：对一个人名实现拼音搜索，只需要在人名字段中新增一个子字段pinyin即可。

#### 3.8 ES的自动类型识别：
1. Dynamic Mapping：
    + ES可以自动识别文档字段类型，从而降低用户使用成本。

``` bash
# ES的自动类型识别
PUT my_index/doc/1
{
 "username":"alfred",    # username字段自动识别为text类型
 "age":20                # age字段自动识别为long类型
}
```

2. ES依靠JSON文档的字段类型实现自动识别字段类型：

| JSON类型 | ElasticSearch类型 |
|---------|-----------------|
|null     |忽略             |
|boolean  |boolean          |
|浮点类型  |float            |
|整数类型  |long            |
|object   |object            |
|array    |由第一个非null的值的类型决定 |
|String   |匹配为日期，则为date类型(默认开启)<br>匹配为数字，则为long类型/float类型(默认关闭)<br>都未匹配，则设为text类型，并附带keyword子字段 |

3. 验证ES的字段类型自动识别：

``` bash
# 验证ES的字段类型自动识别
PUT my_index/doc/1
{
 "username":"alfred",    # 字符串类型text
 "age":20,               # 整数long
 "bitrh":"1998-10-10",   # 默认识别日期date
 "married":false,        # 布尔类型boolean
 "year":"18"             # 默认不识别数字text
 "tags":["boy","fashion"],# 数组中第一个不为null的元素为字符串类型，所以为text
 "money":100.1           # 浮点类型float
}
#  再对my_index进行mapping查询，就会获得每个字段的类型：
```

#### 3.9 ES中日期类型和数字的自动识别：
ES中可自行配置日期的格式，默认：["`strict_date_optional_time`","`yyyy/MM/dd HH:mm:ss Z`|| `yyyy/MM/dd z`"]
``` bash
# 1. 使用dynamic_date_formats自定义日期格式
PUT my_index
{
 "mappings":{
  "doc":{
   "dynamic_date_formats":["MM/dd/yyyy"]
  }
 }
}
# 写入符合自定义格式的日期数据，可识别为date类型
PUT my_index/doc/1
{
 "create_time":"01/01/2019"    # create_time字段识别为date类型
}
# 
#  2. 使用date_detection可以关闭自动识别日期格式：
PUT my_index
{
 "mappings":{
  "doc":{
   "date_detection":false 
  }
 }
}
# 
PUT my_index/doc/1
{
 "create_time":"01/01/2019"    # create_time字段是text类型
}
# 
# ES中可配置数字是否识别，默认关闭：
PUT my_index
{
 "mappings":{
  "doc":{
   "numeric_detection":true    # 开启数字自动识别
  }
 }
}
# 写入数字数据，ES可以自动识别其类型
PUT mu_index/doc/1
{
 "year":"18",    # year字段自动识别为long类型
 "money":"100.1"    # money字段自动识别为float类型
}
```

#### 3.10 ES中根据自动识别的数据类型，动态生成字符类型
例: 
1. 字符串类型都设为keyword类型（不分词）
2. 以message开头的字段都设为text类型（分词）
3. 以long_开头的字段都设为long类型
4. 自动匹配为double的类型都设为float类型。（为了节省空间）

``` bash
# ES根据自动识别的数据类型、字段名等动态设定字符类型
PUT test_index
{
 "mappings":{
  "doc":{
   "dynamic_template":[
    {
     "strings":{
      # 匹配到所有的字符串类型，全部设为keyword类型
      "match_mapping_type":"string",
      "mapping":{
       "type":"keyword"
      }
     }
    }
   ]
  }
 }
}
```

**匹配规则**的参数：
1. `match_mapping_type`：匹配ES自动识别的字段类型，如boolean、long、string等
2. `match`、`unmatch`：匹配字段名，比如"match":"message*" ===>以message开头的数据
3. `path_match`、`path_unmatch`：匹配路径

#### 3.11 自定义mapping的操作步骤
1. 写入一条文档到ES的临时索引中，获取(复制)ES自动生成的mapping
2. 修改获得的mapping，并在其中自定义相关配置
3. 使用修改后的mapping创建实际所需索引。



### 4. ElasticSearch的Search API
在ES中，为了实现对存储的数据进行查询分析，使用`endpoint`：**`_search`**。
1. 实现对所有索引的泛查询：`GET /_search`
2. 实现对一个索引的单独查询：`GET /my_index/_search`
3. 实现对多个索引的指定查询：`GET /my_index1,my_index2/_search`
4. 实现对符合指定要求的索引进行查询：`GET /my_*/_search`

在进行查询的时候，主要有两种方式：(`URI Search`，`Request Body Search`)
1. **`URI Search`**：操作简单，直接通过命令行方便测试，但仅包含部分查询语法；
    + 如：`GET /my_index/_search?q=username:alfred`
2. **`Request Body Search`**：ES提供的完备查询语法，使用`Query DSL(Domain Specific Language)`进行查询

``` bash
# 如：Request Body Search方式进行查询
GET /my_index/_search
{
 "query":{
  "match":{
   "username":"alfred"
  }
 }
}
```

#### 4.1 URI Search
1. 通过`url query`参数实现搜索，常用参数有：
    1. **`q`**：指定查询的语句，使用query string syntax语法
    2. **`df`**：q中不指定字段时默认查询的字段（在不指定的时候默认查询所有字段）
    3. **`sort`**：排序
    4. **`timeout`**：指定超时时间，默认不超时 
    5. **`from,size`**：用于分页
    + 举例：
        * `GET my_index/_search?q=alfred&df=username&sort=age:asc&from=4&size=10&timeout=1s`
        * 解释：查询索引`my_index`中`username`字段中包含`alfred`的文档，结果按`age`字段`升序排列`，返回第`5-14`个文档，若超过`1s`未结束，则以超时结束。
2. `query string syntax`语法
    + 前置内容：`term:单词`，`phrase:词语`。
    + 单词与词语语法：
        * 单词：`alfred way`等价于`alfred` OR `way`
        * 词语：`"alfred way"`语句查询，要求先后顺序
        * 泛查询：不指定字段，会在所有字段中去匹配其单词
        * 指定字段查询：指定字段，在指定字段中匹配单词
    + Group分组设定，使用括号指定匹配的规则
        * 举例：`GET my_index/_search?q=username:(alfred OR way)AND lee`

##### 4.1.1 URI Search API
1. 泛查询：

``` bash
GET my_index/_search?q=alfred
{
 "profile":true #使用profile参数，可以明确地看到ES如何执行的查询条件
}
```

2. 指定字段查询：

``` bash
# a.查询字段username中包含alfred的文档
GET my_index/_search?q=username:alfred
#
# b.查询字段username中包含alfred或way的文档
GET my_index/_search?q=username:alfred way
#
# c.查询字段username为"alfred way"的文档
GET my_index/_search?q=username:"alfred way"
#
# d.分组后，查询字段username中包含alfred，包含way的文档
GET my_index/_search?q=username:(alfred way)
# 这个和b的结果一样，但是区别在于使用分组之后，不进行泛查询。
```

3. 布尔操作符AND(&&)、OR(||)、NOT(!)、+(must)、-(must_not)

``` bash
# 查询索引my_index中username包含alfred但是不包含way的文档
GET my_index/_search?q=username:(alfred NOT way)
#
# 查询索引my_index中一定包含lee，一定不含alfred，可能有way的文档
GET my_index/_search?q=username:(way +lee -alfred)
# 或写成
GET my_index/_search?q=username:((lee && !alfred) || (way && lee && !alfred))
#
# 注意：url中，+(加号)会被解析成空格，所以要用 %2B ：
# 查询索引my_index中一定包含lee，一定不包含alfred，可能包含way的文档
GET my_index/_search?q=username:(way %2Blee -alfred)
```

4. 范围查询（支持数值和日期）
    + 区间写法：闭区间使用`[]`，开区间使用`{}`
        1. `age:[1 TO 10]`  # 1<= age <=10
        2. `age:[1 TO 10}`  # 1<= age <10
        3. `age:[1 TO ]`    # age >=1
        4. `age:[* TO 10]`  # age <=10
    + 算数符号写法：
        1. `age:>=1 `
        2. `age:(>=1 && <= 10) / age:(+ >= 1 + <= 10)`
    + 还可以对日期进行范围查询，注意：年/月是从1月1号/1号开始算的：

``` bash
# a.查询索引my_index中username字段包含alfred_或_年龄大于20的文档
GET my_index/_search?q=username:alfred age>20
#  
# b.查询索引my_index中username字段包含alfred_且_年龄大于20的文档
GET my_index/_search?q=username:alfred AND age>20
# 
# 查询索引my_index中birth字段在1985和1990之间的文档
GET my_index/_search?q=birth:(>1985 AND < 1990)
```

5. 通配符查询
    + `?`代表一个字符，`*`代表0个或多个字符，如：`name:a?lfred`或`name:a*d`或`name:alfred*`
    + 注意：通配符匹配的执行效率较低，且占用内存较多，不建议使用，如果没有特殊要求，也不要将?或者*放在最前面，因为意味着要匹配所有文档，可能会造成OOM。

6. 正则表达式/模糊匹配/近似度查询
    * 正则表达式：举例：`/[a]?l.*/`
    * 模糊匹配：`fuzzy query`
    * 近似度查询：`proximity search`

``` bash
# 模糊匹配。匹配与alfred差一个字符的词，比如：alfreds、alfret等
GET my_index/_search?q=username:alfred~1
#
# 近似度查询，查询字段username和"alfred way"差n个单词的文档
GET my_index/_search?q=username:"alfred way" ~5
```
> 使用场景常见于用户输入词的纠错中。



#### 4.2 Request Body Search
ES自带的完备查询语句，将查询语句通过`http request body`发送到ES，主要参数有：
1. `query`：符合`Query DSL`语法的查询条件
2. `from，size`
3. `timeout`
4. `sort`

+ `Query DSL`语法：
    + 基于`JSON`定义的查询语言，主要包含两个类型：
        1. 字段类查询————如：`term`，`match`，`range`等。只针对一个字段进行查询
        2. 复合查询————如：`bool`查询等。包含一个/多个字段类查询/符合查询语句

##### 4.2.1 字段类查询-全文匹配
针对`text`类型的字段进行全文检索，会对查询语句进行“先分词再查询”处理，如：`match`、`match_phrase`等
###### 4.2.1.1 match query
1. 对字段进行全文检索(最基本和最常用的查询类型)，举例：

``` bash
GET my_index/_search
{
  "query":{  
   "match":{                 # 关键词
    "username":"alfred way"  # 字段名和查询语句
   }
  }
}
```
> 从结果，可以返回匹配文件总数，返回文档列表，_score相关性得分等。
> 一般的执行流程为： 1.对查询语句分词==>2.根据字段的倒排索引列表，进行匹配算分==>3.汇总得分==>4.根据得分排序，返回匹配文档

2. 使用`operator`参数，可以控制单词间关系，有`and/or`：

``` bash
# 使用operator参数控制单词间关系
GET my_index/_search
{
 "query":{
  "match":{
   "username":"alfred way",
   "operator":"and"    # and，同时包含alfred和way
  }
 }
}
```

3. 使用`minimum_should_match`参数控制需匹配的单词数

``` bash
# 使用minimum_should_match参数控制需匹配的单词数
GET my_index/_search
{
 "query":{
  "match":{
   "username":"alfred way",
   "minimum_should_match":"2"
  }
 }
}
```


###### 4.2.1.2 相关性算分，其本质就是一个排序问题
- 计算文档与待查询语句之间的相关度，一般有四个重要概念：
    1. `Term Frequency` 词频(正相关)
    2. `Document Frequency` 文档频率(负相关)
    3. `Inverse Term Frequency` 逆文本频率(正相关)
    4. `Field-length Norm` 文档长度(负相关)
- 目前ES有两个相关性算分的模型：
    1. `TF/IDF`模型：经典模型。
    2. `BM25`模型：5.x版本后的默认模型，是对TF/IDF的优化模型。

1. `TF/IDF`模型：在使用kibana进行查询时，使用explain参数，可以查看具体的计算方法。

``` bash
# 使用explain参数，可以查看具体的相关性的得分是如何计算的
GET my_index/_search
{
 "explain":true,    # 设置为true
 "query":{
  "match":{
   "username":"alfred"
  }
 }
}
```
> 注意：ES计算相关性得分是根据`shard`进行的，即分片的分数计算相互独立，所以在使用的时候要注意分片数，可以通过设定分片数为1来避免这个问题，主要是为了观察，不代表之后所有的分片全都设为1。一般放在创建索引后，未加数据之前。

``` bash
# 设定shards数量为1
PUT my_index
{
 "settings":{
  "number_of_shards":"1"
 }
}
```

2. BM25模型。5.x版本后的默认模型，是对TF/IDF的优化模型。
    + `best match，25`指：迭代了25次才计算。BM25的使用，降低了TF/IDF中因为TF过大导致的负面影响，在BM25中，一个单词的TF一直增长，到一定程度就趋于0变化。


###### 4.2.1.3 match phrase query
对字段做全文检索，有顺序要求。
1. 使用`match——phrase`查询词语

``` bash
GET my_index/_search
{
 "query":{
  "match_phrase":{    # 关键词
   "job":"java engineer"
  }
 }
}
```

2. 通过使用`slop`参数，可以控制单词间间隔：

``` bash
GET my_index/_search
{
 "query":{
  "match_phrase":{
   "job":{
    "query":"java engineer",
    "slop":"1"    # 关键词，设定单词间隔
   }
  }
 }
}
```


###### 4.2.1.4 query string query
类似于`URI Search`中的q参数查询，举例：
1. 使用`query_string`查询

``` bash
GET my_index/_search
{
 "query":{
  "query_string":{
   "default_field":"username",
   "query":{alfred AND way"
  }
 }
}
#
#* 或 */
GET my_index/_search
{
 "query":{
  "query_string":{
   "fileds":["username","job"],
   "query":"alfred OR (java AND ruby)"
  }
 }
}
```


###### 4.2.1.5 simple query string query

类似于`query string`，但会忽略错误的查询语法，且仅支持部分查询语句。使用`+，|，-`分别代替`AND，OR，NOT`。
1. 使用simple query string query
``` bash
GET my_index/_search
{
 "query":{
  "simple_query_string":{
   "fields":[username],
   "query":"alfred +way"    #等价于 "query":"alfred AND way"
  }
 }
}
```


##### 4.2.2 字段类查询-单词匹配
###### 4.2.2.1 term/terms query
将待查询语句作为整个单词进行查询，不做分词处理，举例：
1. 使用term进行单查询

``` bash
GET my_index/_search
{
 "query":{
  "term":{
   "username":"alfred"
  }
 }
}
```

2. 使用terms进行多查询

``` bash
GET my_index/_search
{
 "query":{
  "terms":{
   "username":["alfred","way"]
  }
 }
}
```
> 此时如果直接使用`alfred way`作为`username`查询条件，是不会返回任何文档的。因为在`username`的倒排索引列表中，存在`"alfred"`和`"way"`的索引，但是不存在`"alfred way"`的索引。

###### 4.2.2.2 range query
- 范围查询，主要针对数值类型和日期类型。
    + **`gt`**: greater than 大于
    + **`gte`**: greate than or equal to 大于等于
    + **`lt`**: less than 小于
    + **`lte`**: less than or equal to 小于等于
1. 对数值的查询

``` bash
# range query对数值的查询
GET my_index/_search
{
 "query":{
  "range":{
   "age":{
    "gte":10,
    "lte":20
   }
  }
 }
}
```

2. 对日期的查询

``` bash
# range query对日期的查询
GET my_index/_search
{
 "query":{
  "range":{
   "birth":{
    "lte":"1988-01-01" 
    # 或者使用"lte":"now-30y",这种Date Math类型
   }
  }
 }
}
```
> **`Date Math`类型**：针对日期提供的一种更友好的计算方式。
> 当前时间用`now`代替，具体时间的引用，需要使用`||`间隔。年、月、日、时、分、秒跟`date`一致：`y、M、w、d、h、m、s`。
> 举例：

``` bash
# 假设当前时间为2019-01-02 12:00:00
now+1h   =>   2019-01-02 13:00:00
now-1h   =>   2019-01-02 11:00:00
now-1h/d =>   2019-01-02 00:00:00
2019-01-01||+1M/d  => 2019-02-01 00:00:00
```



##### 4.2.3 复合查询
包含一个/多个字段类查询/符合查询语句
###### 4.2.3.1 constant_score query
1. `constant_score query`: 将内部的查询结果文档得分全部设定为1或boost的值。返回的相关性得分全部为1或boost

``` bash
# 使用constant_score query
GET my_index/_Search
{
 "query":{
  "constant_score":{    #关键词
   "match":{
    "username":"alfred"
   }
  }
 }
}
```

###### 4.2.3.2 bool query
`bool query`: 由一个/多个布尔子句组成，主要包含以下四个：
1. `filter`: 只过滤符合条件的文档，不计算相关性得分，返回的相关性得分全部为0；
    + `ES`会对`filter`进行智能缓存，因此执行效率较高，在做简单匹配查询且不考虑得分的时候没推荐使用`filter`代替`query`

``` bash
# 使用filter查询
GET my_index/_search
{
 "query":{
  "bool":{    # 关键词
   "filter":[
    "term":{
     "username":"alfred"
    }
   ]
  }
 }
}
```

2. `must`: 文档必须符合`must`中的所有条件，影响相关性得分；

``` bash
# 使用must进行查询
GET my_index/_search
{
 "query":{
  "bool":{
   "must":[    
    {
     "match":{
      "username":"alfred"
     }
    },
    {
     "match":{
      "job":"specialist"
     }
    }
   ]
  }
 }
}
```

3. `must_not`: 文档必须排除must_not中的所有条件； 

``` bash
# 使用must_not进行查询
GET my_index/_search
{
 "query":{
  "bool":{
   "must":[
   {
    "match":{
     "job":"java"
    }
   }
   ],
   "must_not":[
   {
    "match":{
     "job":"ruby"
    }
   }
   ]
  }
 }
}
```

4.  `should`: 文档可以符合`should`中的条件，影响相关性得分，分为两种情况：同时配合`minimum_should_match`控制满足调价你的个数/百分比。
    1. `bool`查询中只有`should`，不包含`must`的情况
    2. bool查询中既有should，又包含must的情况，文档不必满足should中的条件，但是如果满足的话则会增加相关性得分。

``` bash
# bool查询中只有should的情况
GET my_index/_search
{
 "query":{
  "bool":{
   "should":[
    {
     "term":{"job":"java"}    # 条件1
    },
    {
     "term":{"job":"ruby"}    # 条件3
    }
    {
     "term":{"job":"specialist"}    # 条件3
    }
   ],
   "minimum_should_match":2    # 至少需要满足两个条件
  }
 }
}
# 
# bool查询中同时包含should和must
GET my_index/_search
{
 "query":{
  "bool":{
   "should":[    # 同时包含should
   {
    "term":{"job":"ruby"}
   }
   ],
   "must":[    # 同时包含must
   {
    "term":{"usernmae":"alfred"}
   }
   ]
  }
 }
}
```
> 当一个查询语句位于query或filter上下文的时候，ES的执行结果也不同。

|-|-|-|
|---------|-----------|---------|
| query   | 查找和查询语句最匹配的文档，<br>并对所有文档计算相关性得分 | query<br>bool中的：must/should |
| filter   | 查找和查询语句最匹配的文档 | bool中的：filter/must_not<br>constant_score中的：filter |

``` bash
# query和filter上下文
GET my_index/_search
{
 "query":{
  "bool":{
   "must":[    # query上下文
   {
    "term":{"title":"Search"}
   },
   {
    "term":{"content":"ElasticSearch"}
   }
   ],
   "filter":[    # filter上下文
   {
    "term":{"status":"published"}
   },
   {
    "range":{
     "publish_date":{
      "gte":"2015-01-01"
     }
    }
   }
   ]
  }
 }
}
```

###### 4.2.3.3 count API
`count API`: 获取符合条件的文档书，使用`endpoint：_count`。
``` bash
# 使用_count获取符合条件的文档数
GET my_index/_count    # 关键词
{
 "query":{
  "match":{
   "username":"alfred"
  }
 }
}
```

###### 4.2.3.4 Source Filtering
`Source Filtering`: 过滤返回结果中的`_source`中的字段，主要由以下两种方式：
1. GET my_index/_search?_source=username #url参数
2. 使用Request Body Search：

``` bash
# 不返回_source
GET my_index/_search
{
 "_source":false
}
# 返回_source部分字段
GET my_index/_search
{
 "_source":["username","age"]
}
# 通配符匹配返回_source部分字段
GET my_index/_search
{
 "_source":{
  "includes":"*I*",
  "encludes":"birth"
 }
}
```
