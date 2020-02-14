---
title: 【ElasticStack】ElasticSearch聚合分析与数据建模
date: 2019-11-21 22:30:47
tags: [ElasticSearch, Kibana, ElasticStack, LogStash]
categories: ElasticStack
---


### 1. ElasticSearch中的聚合分析
聚合分析，英文`Aggregation`，是ES除了搜索功能之外提供的针对ES数据进行统计分析的功能。
- 特点：
    - ①功能丰富，可满足大部分分析需求；
    - ②实时性高，所有计算结果实时返回。
- 基于分析规则的不同，ES将聚合分析主要划分为以下4种：
    1. **`Metric`**: 指标分析类型，如：计算最值，平均值等；
    2. **`Bucket`**: 分桶类型，类似于`group by`语法，根据一定规则划分为若干个桶分类；
    3. **`Pipeline`**: 管道分析类型，基于上一级的聚合分析结果进行再分析；
    4. `Matrix`: 矩阵分析类型。<!-- more -->

``` bash
# 聚合分析格式：
GET my_index/_search
{
 "size":0,
 "aggs":{ # 关键词
  "<aggregation_name>":{ # 自定义聚合分析名称，一般起的有意义
   "<aggregation_type>":{ # 聚合分析类型
    "<aggregation_body>" # 聚合分析主体
   }
  }
  [,"aggs":{[<svb_aggregation>]+}] # 可包含多个子聚合分析
 }
}
```

#### 1.1 Metric聚合分析
主要分为两类：单值分析（输出单个结果）和多值分析（输出多个结果）。

##### 1.1.1 单值分析
1. `min`：返回数值类型字段的最小值
2. `max`：返回数值类型字段的最大值
3. `avg`：返回数值类型字段的平均值
4. `sum`：返回数值类型字段值的总和
5. `cardinality`：返回字段的基数
6. 使用多个单值分析关键词，返回多个结果

``` bash
GET my_index/_search
{
 "size": 0,
 "aggs":{
  "min_age":{
   "min":{ # 关键字min/max/avg/sum/cardinality
    "field":"age"    
   }
  }
 }
}
#
# 使用多个单值分析关键词，返回多个分析结果
GET my_index/_search
{
 "size": 0,
 "aggs": {
  "min_age":{
   "min":{  # 求最小年龄
    "field":"age"
   }
  },
  "max_age":{
   "max":{  # 求最大年龄
    "field":"age"
   }
  },
  "avg_age":{
   "avg":{  # 求平均年龄
    "field":"age"
   }
  },
  "sum_age":{
   "sum":{  # 求年龄总和
    "field":"age"
   }
  }
 }
}
```

##### 1.1.2 多值分析
1. `stats`：返回所有单值结果
2. `extended_stats`：对`stats`进行扩展，包含更多，如：方差，标准差，标准差范围等
3. `Percentile`：百分位数统计
4. `Top hits`：一般用于分桶之后获取该桶内最匹配的定不稳当列表，即详情数据

``` bash
GET my_index/_search
{
 "size": 0,
 "aggs":{
  "stats_age":{
   "stats":{ # 关键字stats/extended_stats/percentiles
    "field":"age"    
   }
  }
 }
}
#
# 使用percentiles关键词进行百分位数预测。
GET my_index/_search
{
 "size": 0,
 "aggs":{
  "per_age":{
   "percentiles":{    # 关键字
    "field":"age",
    "values":[20, 25] # 判断20和25分别在之前的年轻区间的什么位置，以百分数显示
   }
  }
 }
}
#
# 使用top_hits关键词
GET my_index/_search
{
 "size":0,
 "aggs":{
  "jobs":{
   "terms":{
    "match":{
     "field":"job.keyword", # 按job.keyword进行分桶聚合
     "size":10
    },
    "aggs":{
     "top_employee":{
      "top_hits":{
       "size":10,    # 返回文档数量
       "sort":[
        {
         "age":{
          "order":"desc"  # 按年龄倒叙排列
         } 
        }
       ]
      }
     }
    }
   }
  }
 }
}
```


#### 1.2 Bucket聚合分析
`Bucket`，意为桶。即：按照一定规则，将文档分配到不同的桶中，达分类的目的。常见的有以下五类：
1. `Terms`: 直接按`term`进行分桶，如果是`text`类型，按分词后的结果分桶
2. `Range`: 按指定数值范围进行分桶
3. `Date Range`: 按指定日期范围进行分桶
4. `Histogram`: 直方图，按固定数值间隔策略进行数据分割
5. `Date Histogram`: 日期直方图，按固定时间间隔进行数据分割

##### 1.2.1 Terms
`Terms`: 直接按`term`进行分桶，如果是`text`类型，按分词后的结果分桶
``` bash
# 使用terms关键词
GET my_index/_search
{
 "size": 0,
 "aggs":{
  "terms_job":{
   "terms":{    # 关键字
    "field":"job.keyword", # 按job.keyword进行分桶
    "size":5               # 返回五个文档
   }
  }
 }
}
```

##### 1.2.2 Range
`Range`: 按指定数值范围进行分桶：
``` bash
# 使用range关键词
GET my_index/_search
{
 "size": 0,
 "aggs":{
  "number_ranges":{
   "range":{    # 关键字
    "field":"age",    # 按age进行分桶
    "ranges":[
     {
      "key":">=19 && < 25",  # 第一个桶：  19<=年龄<25
      "from":19,
      "to":25
     },
     {
      "key":"< 19",    # 第二个桶：  年龄<19
      "to":19
     },
     {
      "key":">= 25",    # 第三个桶：  年龄>=25
      "from":25
     }
    ]
   }
  }
 }
}
```

##### 1.2.3 Date Range
`Date Range`: 按指定日期范围进行分桶
``` bash
# 使用date_range关键词
GET my_index/_search
{
 "size": 0,
 "aggs":{
  "date_ranges":{
   "date_range":{    # 关键字
    "field":"birth",    # 按age进行分桶
    "format":"yyyy",
    "ranges":[
     {
      "key":">=1980 && < 1990",  # 第一个桶：  1980<=出生日期<1990
      "from":"1980",
      "to":"1990"
     },
     {
      "key":"< 1980",    # 第二个桶：  出生日期<1980
      "to":1980
     },
     {
      "key":">= 1990",    # 第三个桶：  出生日期>=1990
      "from":1990
     }
    ]
   }
  }
 }
}
```

##### 1.2.4 Histogram
`Histogram`: 直方图，按固定数值间隔策略进行数据分割
``` bash
# 使用histogram关键词
GET my_index/_search
{
 "size": 0,
 "aggs":{
  "age_hist":{
   "histogram":{     # 关键词
    "field":"age",
    "interval":3,    # 设定间隔大小为2
    "extended_bounds":{    # 设定数据范围
     "min":0,
     "max":30
    }
   }
  }
 }
}
```

##### 1.2.5 Date Histogram
`Date Histogram`: 日期直方图，按固定时间间隔进行数据分割
``` bash
# 使用date_histogram关键词
GET my_index/_search
{
 "size": 0,
 "aggs":{
  "birth_hist":{
   "date_histogram":{     # 关键词
    "field":"birth",
    "interval":"year",    # 设定间隔大小为年year
    "format":"yyyy",
    "extended_bounds":{    # 设定数据范围
     "min":"1980",
     "max":"1990"
    }
   }
  }
 }
}
```



#### 1.3 Bucket+Metric聚合分析
Bucket聚合分析允许通过添加子分析来进一步进行分析，该子分析可以是Bucket，也可以是Metric。
1. 分桶之后再分桶（Bucket+Bucket），在数据可视化中一般使用千层饼图进行显示。
2. 分桶之后再数据分析（Bucket+Metric）

``` bash
# 分桶之后再分桶——Bucket+Bucket
GET my_index/_search
{
 "size":0,
 "aggs":{
  "jobs":{
   "terms":{             # 第一层Bucket
    "match":{
     "field":"job.keyword",
     "size":10
    },
    "aggs":{
     "age_range":{
      "range":{             # 第二层Bucket
       "field":"age",
       "ranges":[
        {"to":20},
        {"from":20,"to":30},
        {"from":30}
       ]
      }
     }
    }
   }
  }
 }
}
```

``` bash
# 分桶之后再数据分析——Bucket+Metric
GET my_index/_search
{
 "size":0,
 "aggs":{
  "jobs":{
   "terms":{             # 第一层Bucket
    "match":{
     "field":"job.keyword",
     "size":10
    },
    "aggs":{                 
     "stats_age":{
      "stats":{            # 第二层Metric
       "field":"age"
      }
     }
    }
   }
  }
 }
}
```



#### 1.4 Pipeline聚合分析
针对聚合分析的结果进行再分析，且支持链式调用：
``` bash
# 使用pipeline聚合分析,计算订单月平均销售额。
GET my_index/_search
{
 "size": 0,
 "aggs":{
  "sales_per_month":{
   "date_histogram":{
    "field":"date",
    "interval":"month"
   },
   "aggs":{
    "sales":{
     "sum":{
      "field":"price"
     }
    }
   }
  },
  "avg_monthly_sales":{
   "avg_bucket":{    # bucket类型
    "buckets_path":"sales_per_month>sales"    # 使用buckets_path参数，表明是pipeline
   }
  }
 }
}
```

`pipeline`的分析结果会输出到原结果中，由输出位置不同，分为两类：`Parent`和`Sibling`。
1. `Sibling`。结果与现有聚合分析结果同级，如：Max/Min/Sum/Avg Bucket、Stats/Extended Stats Bucket、Percentiles Bucket
2. `Parent`。结果内嵌到现有聚合分析结果中，如：Derivate、Moving Average、Cumulative Sum

``` bash
# Sibling聚合分析(min_bucket)
GET my_index/_search
{
 "size": 0,
 "aggs":{
  "jobs":{
   "terms":{    # 根据job.keyword进行分桶
    "field":"job.keyword",    
    "size":10
   },
   "aggs":{
    "avg_salary":{
     "avg":{    # 之后Metric中求工资的平均数
      "field":"salary"
     }
    }
   }
  },
  "min_salary_by_job":{
   "min_bucket":{    # 关键词
    "buckets_path":"jobs>avg_salary"    # 按工资平均数，排列每个桶中的job
   }
  }
}
```

``` bash
# Parent聚合分析(Derivate)
GET my_index/_search
{
 "size":0,
 "aggs":{
  "bitrh":{
   "date_histogram":{
    "field":"birth",
    "interval":"year",
    "min_doc_count":0
   },
   "aggs":{
    "avg_salary":{
     "avg":{
      "field":"salary"
     }
    },
    "derivative_avg_salary":{
     "derivative":{    # 关键词
      "buckets_path":"avg_salary"
     }
    }
   }
  }
 }
}
```


#### 1.5 聚合分析的作用范围
ES聚合分析默认作用范围是`query的结果集`
``` bash
# ES中聚合分析的默认作用范围是query的结果集
GET my_index/_search
{
 "size":0,
 "query":{
  "match":{
   "username":"alfred"
  }
 },
 "aggs":{
  "jobs":{
   "terms":{
    "match":{    # 此时，只在username字段中包含alfred的文档中进行分桶
     "field":"job.keyword",    
     "size":10
    }
   }
  }
 }
}
```

可通过以下方式修改：`filter`、`post_filter`、`global`
1. filter: 为某个结合分析设定过滤条件，从而在不改变整体query语句的情况下修改范围
2. post_filter，作用于文档过滤，但在聚合分析之后才生效
3. global，无视query条件，基于所有文档进行分析

``` bash
# 使用filter进行过滤
GET my_index/_search
{
 "size":0,
 "aggs":{
  "jobs_salary_small":{
   "filter":{
    "range":{
     "salary":{
      "to":10000
     }
    }
   },
   "aggs":{
    "jobs":{
     "terms":{    # 在salary小于10000的文档中对工作进行分桶
      "field":"job.keyword"
     }
    }
   }
  }
 }
}
```

``` bash
# 使用post_filter进行过滤
GET my_index/_search
{
 "size":0,
 "aggs":{
  "jobs":{
   "terms":{    # 在salary小于10000的文档中对工作进行分桶
    "field":"job.keyword"
   }
  }
 },
 "post_filter":{    # 在集合分析之后才生效
  "match":{
   "job.keyword":"java engineer"   
  }
 }
}
```

``` bash
# 使用global进行过滤
GET my_index/_search
{
 "query":{
  "match":{
   "job.keyword":"java engineer"
  }
 },
 "aggs":{
  "java_avg_salary":{
   "avg":{
    "field":"salary"
   }
  },
  "all":{
   "global":{    # 关键词
    "aggs":{
     "avg_salary":{
      "avg":{
       "field":"salary"    # 依然是对所有的文档进行查询，而不会去管query   
      }
     }
    }
   }
  }
 }
}
```



#### 1.6 聚合分析中的排序
1. 可使用自带的关键数据排序，如：`_count`文档数、`_key`按key值
2. 也可使用聚合结果进行排序

``` bash
# 使用自带的数据进行排序
GET my_index/_search
{
 "size":0,
 "aggs":{
  "jobs":{
   "terms":{
    "field":"job.keyword",
    "size":10,
    "order":[
    {
     "_count":"asc"    # 默认按_count倒叙排列
    },
    {
     "_key":"desc"    使用多个排序值，从上往下的顺序进行排列
    }
    ]
   }
  }
 }
} 
```

``` bash
# 使用聚合结果进行排序
GET my_index/_search
{
 "size":0,
 "aggs":{
  "salary_hist":{
   "histogram":{
   },
   "aggs":{
    "age":{
     "filter":{
      "range":{
       "age":{
        "gte":10
       }
      }
     },
     "aggs":{
      "avg_age":{
       "field":"age"
      }
     } 
    }
   }
  }
 }
}
```



#### 1.7 计算精准度问题
ES聚合的执行流程：每个`Shard`上分别计算，由`coordinating Node`做聚合。
- `Terms`计算不准确原因：数据分散在多个`Shard`上，`coordinating Node`无法得悉数据全貌，那么在取数据的时候，造成精准度不准确。
- 如下图：正确结果应该为`a,b,c`,而返回的是a,b,d

![](http:\\cdn.chaooo.top/Java/elastic-hits.jpg)

- 解决办法有两种：
    1. 直接设置`shard`数量为1；消除数据分散问题，但无法承载大数据量。
    2. 设置`shard_size`大小，即每次从`shard`上额外多获取数据，从而提升精准度

- terms聚合返回结果中有两个统计值：
    1. `doc_count_error_upper_bound`：被遗漏的term可能的最大值；
    2. `sum_other_doc_count`：返回结果bucket的term外其他term的文档总数。

- 设定`show_term_doc_count_error`可以查看每个bucket误算的最大值(`doc_count_error_upper_bound`,为`0`表示计算准确)
- Shard_Size默认大小：`(size*1.5)+10`
    + 通过调整Shard_Size的大小降低`doc_count_error_upper_bound`来提升准确度
    + 增大了整体的计算量，从而降低了响应时间

- 权衡 **`海量数据`、`精准度`、`实时性`** 三者只能取其二。

- Elasticsearch目前支持两种近似算法：cardinality(度量) 和 percentiles(百分位数度量)
    + 结果近似准确，但不一定精准
    + 可通过参数的调整使其结果精准，但同时消耗更多时间和性能





### 2. ElasticSearch的数据建模
数据建模(Data Modeling)大致分为三个阶段：概念建模、逻辑建模、物理建模
1. 概念模型：时间占比`10%`
    + 基础。确定系统的核心需求和范围边界，实际实体与实体之间的关系。
2. 逻辑模型：时间占比`60-70%`
    + 核心。确定系统的核心需求和范围边界，实际实体与实体之间的关系。
3. 物理模型：时间占比`20-30%`
    + 落地实现。结合具体的数据库产品，在满足业务读写性能等需求的前提下确定最终的定义。

#### 2.1 ES中的数据建模
ES是基于Luence以倒排索引为基础实现的存储体系，不遵循关系型数据库中的范式约定。
![](http:\\cdn.chaooo.top/Java/elastic-md.jpg)

#### 2.2 Mapping字段相关设置
1. **`enabled`**:`true/false`。`false`表示 仅存储，不做搜索或聚合分析。
2. **`index`:`true/false`。是否构建倒排索引。不需进行字段的检索的时候设为false。
3. **`index_options`**:`docs/freqs/positions/offsets`。确定存储倒排索引的哪些信息。
4. **`norms`**:`true/false`。是否存储归一化相关系数，若字段仅用于过滤和聚合分析，则可关闭。
5. **`doc_values`**:`true/false`。是否启用doc_values，用于排序和聚类分析。默认开启。
6. **`field_data`**:`true/false`。是否设text类型为fielddata，实现排序和聚合分析。默认关闭。
7. **`store`**:`true/false`。是否存储该字段。
8. **`coerce`**:`true/false`。 是否开启数值类型转换功能，如：字符串转数字等。
9. **`multifields`**:`多字段`。灵活使用多字段特性来解决多样业务需求。
10. **`dynamic`**:`true/false/strict`。控制mapping自动更新。
11. **`date_detection`**:`true/false`。是否启用自定识别日期类型，一般设为false，避免不必要的识别字符串中的日期。

#### 2.3 Mapping字段属性设定流程
`判断类型`--->`是否需要检索`--->`是否需要排序和聚合分析`--->`是否需要另行存储`
1. 判断类型
    + 字符串类型：需要分词，则设为text，否则设为keyword。
    + 枚举类型：基于性能考虑，设为keyword，即便该数据为整型。
    + 数值类型：尽量选择贴近的类型，如byte即可表示所有数值时，即用byte，而不是所有都用long。
    + 其他类型：布尔型，日期类型，地理位置类型等。
2. 是否需要检索
    + 完全不需要检索、排序、聚合分析的字段`enabled设为false`。
    + 不需检索的字段`index设为false`。
    + 需检索的字段，可通过如下配置设定需要的存储粒度:
        - `index_options` 结合需要设定。
        - `norms` 不需归一化数据时可关闭。
3. 是否需要排序和聚合分析
    + 当不需要排序和聚合分析功能时：
        - `doc_values设为false`。
        - `field_data设为false`。
4. 是否需要另行存储
    + `store设为true`即可存储该字段的原始内容(且与`_source`无关)，一般结合`_source`的`enabled设为false`时使用。

#### 2.4 ES建模实例
- 针对博客文章设定索引blog_index，包含字段：
    + 标题：title
    + 发布日期：publish_data
    + 作者：author
    + 摘要：abstract
    + 网址：url
- **简易的数据模型**：

``` bash
# 简易模型blog_index
PUT blog_index
{
 "mappings":{
  "doc":{
    "properties":{
      "title":{
          #title设为text，包含自字段keyword。支持检索、排序、聚合分析
          "type":"text",
          "fields":{
            "keyword":{"type":"keyword"}
          }
      },#publish_data设为date，支持检索、排序、聚合分析
      "publish_data":{"type":"date"},
      # author设为keyword，支持检索、排序、聚合分析
      "author":{"type":"keyword"},
      # abstract设为text，支持检索、排序、聚合分析
      "abstract":{"type":"text"},
      # url设为date，不需进行检索
      "url":{"enabled":false}
     }
   }
 }
}
```

- **如果在`blog_index`中加入一个内容字段`content`**

``` bash
# 为blog_index增加content字段
PUT blog_index
{
 "mappings":{
    "doc":{
     #关闭，不存原始内容到_source
     "_source":{"enabled":false},
     "properties":{
        #title设为text，包含自字段keyword。支持检索、排序、聚合分析
        "title":{
            "type":"text",
            "fields":{
              "keyword":{
               "type":"keyword"
              }
            },
            "store":true #对数据进行存储
        },#publish_data设为date，支持检索、排序、聚合分析
        "publish_data":{
            "type":"date",
            "store":true # 对数据进行存储
        },
        "author":{# author设为keyword，支持检索、排序、聚合分析
            "type":"keyword",
            "store":true    # 对数据进行存储
        },
        "abstract":{# abstract设为text，支持检索、排序、聚合分析
            "type":"text",
            "store":true    # 对数据进行存储
        },
        "content":{# content设为text，支持检索、排序、聚合分析
            "type":"text",
            "store":true    # 对数据进行存储
        },
        "url":{
            "type":"keyword",   # url设为keyword
            "doc_values":false, # url不支持排序和聚合分析
            "norms":false,      # url也不需要归一化数据
            "ignore_above":100, # 预设内容长度为100
            "store":true        # 对数据进行存储
        }
      }
    }
  }
}
```

- **在搜索时增加高亮**: 在此时，`content`里面的数据会存储大量的内容数据，数据量可能达到上千、上万，甚至几十万。那么在搜索的时候，根据`search`机制，如果还是像之前一样进行`_search`搜索，并只显示其他字段的话，其实依然还是每次获取了`content`字段的内容，影响性能，所以，使用`stored_fields`参数，控制返回的字段。节省了大量资源：

``` bash
# 使用stored_fields返回指定的存储后的字段
GET blog_index/_search
{
 "stored_fields":["title","publish_data","author","Abstract","url"],
 "query":{
  "match":{
   "content":"world"#依然进行content搜索，但是不返回所有的content字段
  }
 },
 "highlight":{ #针对content字段进行高亮显示
  "fields":{
     "content":{}
  }
 }
}
```
> 注意：`GET blog_index/_search?_source=title` 虽然只显示了`title`，但是`search`机制决定了，会把所有`_source`内容获取到，但只是显示`title`。

#### 2.5 ES中关联关系处理
`ES`不擅长处理关系型数据库中的关联关系，因为底层使用的倒排索引，如：文章表`blog`和评论表`comment`之间通过`blog_id`关联。
目前ES主要有以下4种常用的方法来处理关联关系：
1. **`Nested Object`**:嵌套文档
2. **`Parent/Child`**:父子文档
3. `Data denormalization`:数据的非规范化
4. `Application-side joins`:服务端Join或客户端Join

##### 2.5.1 Application-side joins（服务端Join或客户端Join）
索引之间完全独立（利于对数据进行标准化处理，如便于上述两种增量同步的实现），由应用端的多次查询来实现近似关联关系查询。
- 适用于第一个实体只有少量的文档记录的情况（使用`ES`的`terms`查询具有上限，默认`1024`，具体可在`elasticsearch.yml`中修改），并且最好它们很少改变。这将允许应用程序对结果进行缓存，并避免经常运行第一次查询。

##### 2.5.2 Data denormalization（数据的非规范化）
通俗点就是通过字段冗余，以一张大宽表来实现粗粒度的`index`，这样可以充分发挥扁平化的优势。但是这是以牺牲索引性能及灵活度为代价的。
- 使用的前提：冗余的字段应该是很少改变的；比较适合与一对少量关系的处理。当业务数据库并非采用非规范化设计时，这时要将数据同步到作为二级索引库的ES中，就很难使用上述增量同步方案，必须进行定制化开发，基于特定业务进行应用开发来处理`join`关联和实体拼接。
> 宽表处理在处理一对多、多对多关系时，会有字段冗余问题，适合“一对少量”且这个“一”更新不频繁的应用场景。

##### 2.5.3 Nested objects（嵌套文档）
索引性能和查询性能二者不可兼得，必须进行取舍。
嵌套文档将实体关系嵌套组合在单文档内部（类似与json的一对多层级结构），这种方式牺牲索引性能（文档内任一属性变化都需要重新索引该文档）来换取查询性能，可以同时返回关系实体，比较适合于一对少量的关系处理。
- 当使用嵌套文档时，使用通用的查询方式是无法访问到的，必须使用合适的查询方式（nested query、nested filter、nested facet等），很多场景下，使用嵌套文档的复杂度在于索引阶段对关联关系的组织拼装。

##### 2.5.4 Parent/Child（父子文档）
父子文档牺牲了一定的查询性能来换取索引性能，适用于一对多的关系处理。其通过两种type的文档来表示父子实体，父子文档的索引是独立的。父-子文档ID映射存储在 Doc Values 中。当映射完全在内存中时， Doc Values 提供对映射的快速处理能力，另一方面当映射非常大时，可以通过溢出到磁盘提供足够的扩展能力。
- 在查询parent-child替代方案时，发现了一种filter-terms的语法，要求某一字段里有关联实体的ID列表。基本的原理是在terms的时候，对于多项取值，如果在另外的index或者type里已知主键id的情况下，某一字段有这些值，可以直接嵌套查询。具体可参考官方文档的示例：通过用户里的粉丝关系，微博和用户的关系，来查询某个用户的粉丝发表的微博列表。
> 父子文档相比嵌套文档较灵活，但只适用于“一对大量”且这个“一”不是海量的应用场景，该方式比较耗内存和CPU，这种方式查询比嵌套方式慢5~10倍，且需要使用特定的has_parent和has_child过滤器查询语法，查询结果不能同时返回父子文档（一次join查询只能返回一种类型的文档）。

- 而受限于父子文档必须在同一分片上，ES父子文档在滚动索引、多索引场景下对父子关系存储和联合查询支持得不好，而且子文档type删除比较麻烦（子文档删除必须提供父文档ID）。
- 如果业务端对查询性能要求很高的话，还是建议使用宽表化处理的方式，这样也可以比较好地应对聚合的需求。在索引阶段需要做join处理，查询阶段可能需要做去重处理，分页方式可能也得权衡考虑下。


#### 2.6 ES中的reindex
`reindex`：指重建所有数据的过程，一般发生在一下情况：
1. `mapping`设置变更，如：字段类型变化，分词器字典更新等；
2. `index`设置变更，如：分片数变化；
3. 迁移数据。

- ES提供了线程的api用于完成数据重建：
    + `_update_by_query`：在现有索引上重建；
    + `_reindex`：在其他索引上重建。

``` bash
# 将blog_index中所有文档重建一遍：
# 如果遇到版本冲突，依然执行。
POST blog_index/_update_by_query?conflicts=proceed    
# 此时如果blog_index中没有store的数据，则会报错
```

##### 2.6.1 使用`_update_by_query`，更新文档的字段值和部分文档：
``` bash
# 更新文档的字段值及部分文档
POST blog_index/_update_by_query
{
 "script":{    # 更新文档的字段值
  "source":"ctx._source.likes++",    # 代码
  "lang":"painless"    # ES自带script语法
 },
 "query":{    # 更新部分文档
  "term":{
   "user":"tom"
  }
 }
}
```

在reindex发起后进入的文档，不会参与重建，类似于快照的机制。因此：一般在文档不再发生变更时，进行文档的reindex。

##### 2.6.2 使用`_reindex`，重建数据：
``` bash
# 使用_reindex：
POST _reindex
{
 "source":{    # 被重建索引
  "index":"blog_index"
 },
 "dest":{    # 目标索引
  "index":"blog_new_index"
 }
} 
```

- 数据重建时间，受到索引文档规模的影响，此时设定`url`参数`wait_for_completion`为`false`，来异步执行。
- `ES`通过`task`来描述此类执行任务，并提供了`task api`来查看任务的执行进度和相关数据：

``` bash
# 使用task api
POST blog_index/_update_by_query?comflicts=proceed&wait_for_completion=false
# 使用返回的taskid，查看任务的执行进度和相关数据
GET _tasks/<返回的task id>
```

#### 2.7 其他建议：
1. 对mapping进行版本管理：
    + 要么写文件/注释，加入到`Git`仓库，一眼可见；
    + 要么增加`metadata`字段，维护版本，并在每次更新`mapping`设置的时候加`1`。

``` bash
"metadata":{
 "version":1
}
```

2. 防止字段过多：
    + `index.mapping.total_fields_limit`，默认`1000`个。一般是因为没有高质量的数据建模导致，如：`dynamic`设为`true`。此时考虑查分多个索引来解决问题。

