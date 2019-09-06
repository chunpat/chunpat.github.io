---
layout: post
title: elasticsearch学习
date: 2019-09-02 
tags: elasticsearch
---

## 安装
下载相应版本 https://github.com/elastic/elasticsearch/releases

## 配置
修改配置，其他主机可以访问，单节点部署
```
vim config/elasticsearch.yml
```

![FhoCc45P5ISRy7oFsr5ZQLdhK-3P](http://img.zzhpeng.cn/FqYRhHL4hv5NrrJMumI_wtDGIng1)


> 其他配置

|  属性名   | 说明  |
|  ----  | ----  |
| cluster.name  | 配置elasticsearch的集群名称，默认是elasticsearch。建议修改成一个有意义的名称。 |
| node.name  | 节点名，es会默认随机指定一个名字，建议指定一个有意义的名称，方便管理 |
| path.conf  | 设置配置文件的存储路径，tar或zip包安装默认在es根目录下的config文件夹，rpm安装默认在/etc/ elasticsearch |
| path.data  | 设置索引数据的存储路径，默认是es根目录下的data文件夹，可以设置多个存储路径，用逗号隔开 |
| path.logs  | 设置日志文件的存储路径，默认是es根目录下的logs文件夹 |
| path.plugins  | 设置插件的存放路径，默认是es根目录下的plugins文件夹 |
| bootstrap.memory_lock | 设置为true可以锁住ES使用的内存，避免内存进行swap |
| network.host | 设置bind_host和publish_host，设置为0.0.0.0允许外网访问 |
| http.port | 设置对外服务的http端口，默认为9200。 |
| transport.tcp.port | 集群结点之间通信端口 |
| discovery.zen.ping.timeout | 设置ES自动发现节点连接超时的时间，默认为3秒，如果网络延迟高可设置大些 |
| discovery.zen.minimum_master_nodes | 主结点数量的最少值 ,此值的公式为：(master_eligible_nodes / 2) + 1 ，比如：有3个符合要求的主结点，那么这里要设置为2 |


## lk分词插件
中文分词器，同lucene一样，在使用中文全文检索前，需要集成IK分词器。找到相应的发行版本安装。
```
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.3.1/elasticsearch-analysis-ik-7.3.1.zip

```
如果报错，多数是网络问题，文件没下载完整。

## Kibana图形管理安装
待。。。

## php实战

### 基本概念

|  概念   | 说明  |
|  ----  | ----  |
| 索引库（indices)  | indices是index的复数，代表许多的索引， |
| 文档（document）  | 存入索引库原始的数据。比如每一条商品信息，就是一个文档 |
| 字段（field）  | 文档中的属性 |
| 映射配置（mappings） | 字段的数据类型、属性、是否索引、是否存储等特性 |

### 代码实现
这里用官方组出的客户端轮子https://github.com/elastic/elasticsearch-php.git

> 生成索引（index)

```

 protected static $client = null;

/**
 * @return mixed
 */
 public static function getClient()
 {
    if(!isset(self::$client)){
        $hosts = [
            env('ELASTICSEARCH.HOST','192.168.199.254:9200'),     // IP + Port
        ];
        self::$client = ClientBuilder::create()           // Instantiate a new ClientBuilder
        ->setHosts($hosts)      // Set the hosts
        ->build();              // Build the client object
    }
    return self::$client;
 }
 
/**
 * 加属性
 * @author: zzhpeng
 * Date: 2019/9/5
 * @param array $param
 */
public static function addCommonParams(array &$param){
    // 统一添加404报错忽略
    $param = array_merge($param,['client' => [ 'ignore' => 404 ]]);
}

 /**
 * 创建索引
 * @author: zzhpeng
 * Date: 2019/9/5
 * @param string $index
 * @param array  $param
 *
 * @return mixed
 */
public static function setIndex(string $index, array $param = []){
    $client = self::getClient();
    if(empty($param)){
        $params = [
            //索引名
            'index' => $index,
            'body' => [
                'settings' => [
                //一般以（节点数*1.5或3倍）来计算，比如有4个节点，分片数量一般是6个到12个，每个分片一般分配一个副本
                    'number_of_shards' => 1,
                    'number_of_replicas' => 1
                ],
                //mappings是定义字段和类型的。
                'mappings' => [
                    //文档的源数据
                    '_source' => [
                        'enabled' => true
                    ],
                    //设置属性类型
                    'properties' => [
                        'name' => [
                            'type' => 'text',
                        ],
                        'search_name' => [
                            //type：类型，可以是text、long、short、date、integer、object等
                            'type' => 'text',
                            //analyzer：分词器，这里的ik_max_word即使用ik分词器
                            'analyzer' => 'ik_max_word',
                            'search_analyzer' => 'ik_smart',
                        ],
                        'goods_detail_id' => [
                            'type' => 'integer'
                        ],
                        'classify_type_id' => [
                            'type' => 'integer'
                        ],
                        'store_id' => [
                            'type' => 'integer'
                        ]
                    ]
                ]
            ]
        ];
    }
    $response = $client->indices()->create($params);
    return $response;
}
```

> 删除索引（index)

```
/**
 * 删除索引
 * @author: zzhpeng
 * Date: 2019/9/5
 * @param string $index
 *
 * @return mixed
 */
public static function deleteIndex(string $index){
    $client = self::getClient();
    $params = ['index' => $index];
    self::addCommonParams($params);
    $response = $client->indices()->delete($params);
    return $response;
}

```

> 查看索引（index)

```
/**
 * 查询索引
 * @author: zzhpeng
 * Date: 2019/9/5
 * @param string $index
 *
 * @return mixed
 */
static function getIndex(string $index){
    // Get mappings for all indexes
    $client = self::getClient();

    $params = ['index' => $index];
    $response = $client->indices()->getMapping($params);
    return $response;
// Get mappings in 'my_index'
    $params = ['index' => 'my_index'];
    $response = $client->indices()->getMapping($params);
}

```

> 创建文档（document）

```
/**
 * 建立文档
 * @author: zzhpeng
 * Date: 2019/9/5
 * @param string $index
 * @param array  $documents
 *
 * @return bool
 */
static function setDocoments(string $index,array $documents){
    
    // Get mappings for all indexes
    $client = self::getClient();

    //二位数组 $documents
    foreach ($documents as $key => $document){
        $params = [
            'index' => $index,
//                'id'    => $key,
            'body'  => $document

        ];
        
        $client->index($params);
        unset($params);
    }
    return true;
}
```

> 删除文档（document）

```

/**
 * 获取文档
 * @author: zzhpeng
 * Date: 2019/9/5
 * @param $index
 * @param $id
 *
 * @return mixed
 */
static function deleteDocuments($index,$id){
    $client = self::getClient();
    $params = [
        'index' => $index,
        'id'    => $id
    ];

// Delete doc at /my_index/_doc_/my_id
    return $client->delete($params);
}
```

> 搜索文档（document）

* match是经过analyer的，也就是说，文档首先被分析器给处理了。根据不同的分析器，分析的结果也稍显不同，然后再根据分词结果进行匹配。

* term则不经过分词，它是直接去倒排索引中查找了精确的值了。

```
$body = [
    'query' => [
        'bool' => [
            'must' => [
                [ 'match' => [ 'search_name' => $key ] ],
                [ 'bool'=>
                    ['should' => [
                        [ 'match' => [ 'store_id' => 681 ] ],
                        [ 'match' => [ 'store_id' => 0 ] ],
                    ]
                    ]
                ]
            ]
        ]
    ]
];


//搜索文档
static function searchDocuments($index,array $body){
    $client = self::getClient();

    $params = [
        'index' => $index,
        'body'  => $body
    ];

// Get doc at /my_index/_doc/my_id
    return $response = $client->search($params);
}

```

## 参考

1、[图解Elasticsearch中的_source、_all、store和index属性 - weixin_33692284的博客 - CSDN博客](https://blog.csdn.net/weixin_33692284/article/details/92170069)

2、[官方说明文档：](https://www.elastic.co/guide/en/elasticsearch/client/php-api/current/overview.html)

3、[官方开发的php客户端组件:](https://github.com/elastic/elasticsearch-php)

4、[Elasticsearch7.*版本 1.入门 - 简书](https://www.jianshu.com/p/88f0546d5955)