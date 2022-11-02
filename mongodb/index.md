# 

MongoDB是一款文档型数据库



特性

- 文档数据库（schema free），底层存储基于二进制JSON存储文档
- 高性能、高可用、直接加机器即可解决扩展性问题
- 支持丰富的CRUD操作，例如：聚合统计、全文检索、坐标检索

&gt; 什么是文档数据库?

MongoDB存储的一个个JSON，我们称这些JSON为文档，里面有若干个field:value，value可以为任何类型，支持数组、json嵌套

```json
{
    name： “marry&#34;,					&lt;== field:value
    age: 26,
    status: &#34;A&#34;,
    groups: [ &#34;news&#34;,&#34;sports&#34; ]
}
```

**概念对比**

| MySQL       | MongoDB                           |
| :---------- | --------------------------------- |
| database    | database                          |
| table       | collection                        |
| row         | document(bson)                    |
| colume      | field                             |
| index       | index                             |
| table joins | $lookup​                           |
| primary key | _id                               |
| group by    | aggregation pipeline 流水线式聚合 |

&gt; **用法实例**

**选择数据库**

- 列举数据库 `show databases`
- 选择数据库 `use db_name`
- 结论：数据库无需创建，只是一个命名空间

**创建collection**

- 列举数据表 `show collections`
- 建立数据表 `db.createCollection(&#34;db_name&#34;)` db.为固定前缀
- 结论：数据表是schema free，无需定义字段

**插入文档**

- `db.table_name.insertOne({uid:10000,name:&#34;anna&#34;, likes:[&#34;football&#34;,&#34;game&#34;]})`

  ```monggo
  &gt; db.test.insertOne({uid:10000, name:&#34;anna&#34;, likes:[&#34;football&#34;,&#34;game&#34;]})
  {
  	&#34;acknowledged&#34; : true,
  	&#34;insertedId&#34; : ObjectId(&#34;5d9562bf8ea878a253d8caae&#34;)
  }
  ```

  

- 结论1：任意嵌套层级的BSON（二进制的JSON）

- 结论2：文档ID是自动生成的，通常无需自己指定。

**查询document**

- `db.table_name.find(likes: &#39;football&#39;, name: {$in: [&#39;anna&#39;, &#39;judy&#39;]}).sort({uid:1})` `.sort({uid:1})` 表示升序

- 结论1：可以基于任意BSON层级过滤

- 结论2：支持的功能与MYSQL相当

  ```
  &gt; db.test.find({uid:10000})
  { &#34;_id&#34; : ObjectId(&#34;5d9562bf8ea878a253d8caae&#34;), &#34;uid&#34; : 10000, &#34;name&#34; : &#34;anna&#34;, &#34;likes&#34; : [ &#34;football&#34;, &#34;game&#34; ] }
  ```

**更新document**

- `db.table_name.updateMany({likes: &#39;football&#39;}, {$set: {name: &#34;anna&#34;}})`  $set操作符
- 结论1：第一个参数是过滤条件
- 结论2：第二个参数是更新操作

**删除document**

- `db.table_name.deleteMany({name: &#39;anna&#39;})`
- 结论：参数是过滤条件

**创建index**

如需要加速查询，需要建立索引，否则执行全表扫描

- `db.table_name.createIndex({uid:1, name: -1})` 这里创建的联合索引uid,name。MongoDB中索引的顺序影响到排序的效率

- 结论：可以指定建立索引时的正反顺序

  ```
  &gt; db.test.createIndex(&#39;{uid=1}&#39;)
  {
  	&#34;ok&#34; : 0,
  	&#34;errmsg&#34; : &#34;The field &#39;key&#39; must be an object, but got string&#34;,
  	&#34;code&#34; : 14,
  	&#34;codeName&#34; : &#34;TypeMismatch&#34;
  }
  ```

  

聚合类比

| Mysql    | MongoDB  |
| -------- | -------- |
| where    | $match   |
| group by | $group   |
| having   | $match   |
| select   | $project |
| order by | $sort    |
| limit    | $limit   |
| sum      | $sum     |
| count    | $sum     |

**聚合统计实例**

```json
db.table_name.aggregate([{
    {$unwind: &#39;$like&#39;},
    {$group: { _id:{likes: &#39;$likes&#39;}}}},
    {$project: { _id:0, like: &#39;$_id.likes&#39;, total:{$sum:1}}}
}])
```

$unwind 一行拆成两行，anna like football anna like game



MongoDB整体存储架构

- Mongod：单机版数据库，不具备高可用性。
- Replica Set：复制集，由多个Mongod进程组成的高可用存储单元。
- Sharding：分布式集群，由多个Replica Set组成的可扩展集群。

&lt;center&gt;&lt;strong&gt;Mongod架构&lt;/strong&gt;&lt;/center&gt;
![image-20220506224743828](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220506224743828.png)

- 默认采用WiredTiger高性能存储引擎
- 基于journaling log宕机恢复（类比mysql的redo log），Mongo存储引擎，当写入数据时，将要做的事情顺序追加到log中，数据是暂时修改在内存中，会定期将内存中的修改写入磁盘。如在写入磁盘前down机，重启后会根据journaling log将数据恢复。实现了高存储写入



&lt;center&gt;&lt;strong&gt;Replica Set架构&lt;/strong&gt;&lt;/center&gt;


![image-20220506224801168](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220506224801168.png)

- 至少3个节点组成，其中1个可以充当arbiter
- 主从基于oplog复制同步（类比mysql binlog）
- 客户端默认读写primary节点

&lt;center&gt;&lt;strong&gt;Sharding架构&lt;/strong&gt;&lt;/center&gt;
![image-20220506224815054](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220506224815054.png)

mongos：mongos作为代理，路由请求到特定shard

config server：配置服务器，用于存储元数据，3个mongd节点组成config server，保存数据元信息

每个shard是一个replica set，可以无限扩容

&lt;center&gt;&lt;strong&gt;Colection分片&lt;/strong&gt;&lt;/center&gt;
![image-20220506224828779](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220506224828779.png)

- Collection自动分裂成多个chunk
- 每个chunk被自动负载均衡到不同的shard
- 每个shard可以保证其上的chunk高可用



&lt;center&gt;&lt;strong&gt;按range拆分chunk&lt;/strong&gt;&lt;/center&gt;
​		![image-20220506224843347](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220506224843347.png)			
​				

   - Shard key可以是单索引字段者联合索引字段

   - 超过16MB的chunk被一分为二

   - 一个collection的所有chunk首尾相连，构成整个表

     &lt;center&gt;&lt;strong&gt;按hash拆分chunk&lt;/strong&gt;&lt;/center&gt;
解决写入热点问题				 

![image-20220506224857365](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220506224857365.png)

MongoDB支持hash index，会先将字段做hash，得到一个int，将int去建立索引

hash是用重复率的，不同的数据会出现相同的值，MongoDB不允许对hash索引建立唯一

- 5hard key必须是希索引字段
- Shard key经过hash函数打散，避免写热点
- 支持预分配chunk，避免运行时分裂影响写入



### shard用法示例

- 为DB激活特性：`sh.enableSharding(table_name)` 
- 配置hash分片：`sh.shardcollection(&#34;db_name.table_name&#34;，{_id：&#34;hashed&#34;}，false，{numInitialChunks：10240})` 基于自增_id做hash索引，\_id是MongoDB自行生成。当插入数据时会基于ID取模存储到10240的chunk上。

提示：

- 按非shard key查询，请求被扇出给所有shard





启动MongoDB

```
bin/mongod --dbpath=./data --bind_ip=0.0.0.0 &amp;
```






