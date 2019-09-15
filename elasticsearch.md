## elasticsearch

### 安装head 插件：

1. 打开elasticsearch.yml 中几处配置

   cluster.name: my-application

   node.name: node-1

   network.host: 0.0.0.0

   http.port: 9200

   cluster.initial_master_nodes: ["node-1"]  //除掉node-2

2. 追加

   http.cors.enabled: true 
   http.cors.allow-origin: "*"

### 常用命令

* 创建索引，创建索引是可以设定索引的结构化信息和初始的一些信息。如下创建pepple索引：

  post ip:port/peple

  ~~~json
  //**貌似新版本的如7之后的es 不支持通过这种方式设定并结构化类型，但支持settings
  {
      "settings":{ 
          "number_of_shards":3,  		//指定索引分片数
          "number_of_replicas":1 		//指定索引分片的备份数
      },
      "mappings":{ //索引类型的结构化映射
      	"man":{ //定义类型man
              "properties":{  //定义类型的属性
                  "name":{
                      "type":"text"
                  },
                  "country":{
                      "type":"keyword" //定义为keyword类型则不会分词
                  },
                  "age":{
                      "type":"integer"
                  },
                  "date":{
                      "type":"date",
                      "format":"yyyy-MM-DD HH:mm:ss||yyyy-MM-DD||epoch_millis" //格式可以是多个 也支持时间戳
                  }
              }
          },
          "woman":{}
  	}
  }
  //修改以存在索引的文档类型
  get ip：port/index
  {
  	"mappings":{ 
      	"man":{ 
              "properties":{  
                  "name":{
                      "type":"text"
                  },
                  "country":{
                      "type":"keyword" 
                  },
                  "age":{
                      "type":"integer"
                  },
                  "date":{
                      "type":"date",
                      "format":"yyyy-MM-DD HH:mm:ss||yyyy-MM-DD||epoch_millis" 
                  }
              }
          }
  	}
  }
  ~~~

* 插入数据

  1. 指定id, 用put 

     put ip:port/index/type/id

     {

     ​	“name”:“title”

     }

  2. 自动生成id

     post ip:port/index/type

     {

     ​	“name”:“title”

     }

* 修改数据

  1. 直接指定id,修改字段

     post ip:port/index/type/id/_update

     {

     ​	“doc”:{

     ​		“name”:“newname”

     ​	}

     }

  2. 通过脚本方式

     post ip:port/index/type/id/_update

     ~~~json
     {
     	“script”:{
     		“lang”:“painless” //painless是es内置脚本语言，其他还支持 nodejs,python
     		“inline”：“ctx._source.age += 10”  //ctx._source 表示上下文是当前文档
     	}
     }
     ~~~

     

  3. 删除文档和索引

     delete ip:port/index/type/id  //删除文档

     delete ip:port/index 			//删除索引

  4. 查询

     根据id查询：get ip:port/index/type/id

     范围查询：

     post ip:port/index/{type/}_search

     ~~~josn
     //查询全部
     {
     	"query":{
     		"match_all":{}
     	}
     	"from":1,   //可以有类似mysql limt风格 ，如果全部显示则不需要写后面两个参数
     	"size":1
     }
     //按条件查询
     {
     	"query":{
     		"match":{
     			"title":"hello"  //title中包含hello都会被检索
     		}
     	}，
     	"sort":[
     		{"public_date":{"order":"desc"}} //降序
     	]
     }
     //聚合查询,统计数量
     { //下面有两个聚合字段，会返回两个
     	"aggs":{
     		"group_by_count":{  //这个字段名字自定义
     			"terms":{
     				"field":"word_count"
     			}
     		},
     		"group_by_pubtime":{
     			"terms""{
     				"field":"pubtime"
     			}
     		}
     	}
     }
     //字段计算
     {
     	"aggs":{
     		"grade_word_count":{
     			"stats":{  //也可以是min,max  stats 会做最小最大平均总和的统计等
     				"field":"word_count"
     			}
     		}
     	}
     }
     ~~~

### 高级查询

* 精确查找（习语查询）

  post ip:port/index/_search

  ~~~json
  {
      "query":{
          "match_phrase":{
              "title":"hello"
          }
      }
  }
  ~~~

* 多个字段or方式模糊查找

  ~~~json
  {
      "query":{
          "multi_match":{
              "query":"hello",
              "fields":["title","name"]   //name or title 包含hello
          }
      }
  }
  ~~~

* 语法查询

  ~~~json\
  {
  	"query":{
  		"query_string":{ //表示语法查询
  			"query":"(hello AND good) OR golang",  //必须包含(hello 和 good)两个 或者 golang
  			"fields":["title","name"] //如果不加这句，则在所有字段中进行搜索
  		}
  	}
  }
  ~~~

* 针对字段而非文本的查找  和 范围查找

  ~~~json
  {
      "query":{
          "term":{
              "country":"china"
          }
      }
  }
  //范围查找
  {
      "query":{
          "range":{
              "word_coiunt":{
                  "gte":1000,  //  1000 <= word_count >=2000
                  "lte":2000
              }
          }
      }
  }
  ~~~

  

### filter查询

~~~json
{
    "query":{
        "bool":{
            "filter":{
                "term":{
                    "word_count":1000   //只返回word_count的数据，而且es会对filter数据进行缓存
                }
            }
        }
    }
}
~~~

* 固定分数查询 constant_score  ???

* must mustnot

  ~~~json
  { //should 两个条件是或的关系
      "query":{
          "bool":{
              "should":[
              	{
                      "match":{
                          "title":"hello"
                      }
                  },
                  {
                      "match":{
                          "name":"hello"
                      }
                  }
              ]
          }
      }
  }
  
  { //must 两个条件都包含
      "query":{
          "bool":{
              "must":[
                  {
                     "match":{
                  	"name":"hello"
                  	}
                  },
                  {
                     "match":{
                         "title":"hello"
                     } 
                  }
              ],
              "filter":[
                  {
                      "term":{
                          "word_count":1000  //只有word_count 1000的才会返回
                      }
                  }
              ]
          }
      }
  }
  ~~~

  

