---
layout: post
title: MySQL的Replace操作
date: 2020-11-29
tags: thinkphp5.1 mysql
---

## 缘由

想复现两条重复的数据，但是一直被替换，死死不出来。查看sql日志，发现出现了Replace操作，之前没见过，词义为替换。
第一时间想的是和索引有关，看了下还真是，去掉索引之后就复现了数据。

## 日志

tp5.1日志，出现了REPLACE

```
[ sql ] [ SQL ] REPLACE INTO `store_system_depot_goods_detail` 
(`goods_name` , `classify_type_id` , `skus_json` , `store_id` , 
`account_id` , `goods_sn` , `classify_id` , `goods_id` , `create_time` ,
 `update_time`) VALUES ('定义商品编码' , 1 , '{\"\\u5c0f\\u7c73\\u673a\\u578b\":\"HM-\\u7ea2\\u7c73NOTE5\"}' , 
 0 , 2269 , '6931755509057' , 1 , 426741 , '2021-04-19 10:23:45' , 
 '2021-04-19 10:23:45') [ RunTime:0.008989s ]
```

## 怎么实现
tp5.1，php代码,路径thinkphp/library/think/model/relation/HasMany.php，一对多关联的时候使用
```
    /**
     * 保存（新增）当前关联数据对象
     * @access public
     * @param  mixed    $data       数据 可以使用数组 关联模型对象 和 关联对象的主键
     * @param  boolean  $replace    是否自动识别更新和写入
     * @return Model|false
     */
    public function save($data, $replace = true)
    {
        if ($data instanceof Model) {
            $data = $data->getData();
        }

        // 保存关联表数据
        $data[$this->foreignKey] = $this->parent->{$this->localKey};

        $model = new $this->model;

        return $model->replace($replace)->save($data) ? $model : false;
    }
```

tp5.1 最终Sql转换code，thinkphp/library/think/db/Builder.php
```
     /**
      * 生成Insert SQL
      * @access public
      * @param  Query     $query   查询对象
      * @param  bool      $replace 是否replace
      * @return string
      */
     public function insert(Query $query, $replace = false)
     {
         $options = $query->getOptions();
 
         // 分析并处理数据
         $data = $this->parseData($query, $options['data']);
         if (empty($data)) {
             return '';
         }
 
         $fields = array_keys($data);
         $values = array_values($data);
 
         return str_replace(
             ['%INSERT%', '%TABLE%', '%FIELD%', '%DATA%', '%COMMENT%'],
             [
                 $replace ? 'REPLACE' : 'INSERT',
                 $this->parseTable($query, $options['table']),
                 implode(' , ', $fields),
                 implode(' , ', $values),
                 $this->parseComment($query, $options['comment']),
             ],
             $this->insertSql);
     }
```

结论：Replace是由Mysql自己做处理的

## 总结

**慎用MySQL Replace语句**，存在PRIMARY KEY或者UNIQUE索引，有利有弊。利：可以做去重处理。
弊：删除旧数据新增新数据。

## 迭代
* 2021年04月19日 11:26:40 初稿

## 参考：
* 1、[慎用MySQL replace语句 - sunss - 博客园](https://www.cnblogs.com/sunss/p/4493803.html)
