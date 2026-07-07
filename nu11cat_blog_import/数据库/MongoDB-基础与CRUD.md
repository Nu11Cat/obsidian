---
title: MongoDB-基础与CRUD
tags: [MongoDB, 数据库]
categories: [Tech, 数据库]
toc: true
---

MongoDB（文档型数据库）以 BSON 文档为存储单元，通过集合（Collection）组织数据。本篇覆盖数据模型与关系型表的对照、`mongosh` 常用 CRUD、索引入门与聚合管道常用阶段，便于与站内 MySQL、Redis 笔记衔接阅读。



<!-- more -->



## MongoDB 是什么

MongoDB 是面向文档的 NoSQL 数据库，数据以 **BSON（Binary JSON）** 形式存放在 **集合（Collection）** 中。集合无强制统一 schema，同一集合内文档字段可以不同，适合结构演进频繁的业务。

与关系型数据库（如 MySQL）的常见对应关系：

| 关系型概念 | MongoDB 概念 |
|------------|----------------|
| 数据库 Database | 数据库 Database |
| 表 Table | 集合 Collection |
| 行 Row | 文档 Document |
| 列 Column | 字段 Field |
| 主键 Primary Key | 默认 `_id` 字段（ObjectId 或自定义） |

-----------

## BSON 与文档模型要点

- **文档**：一条记录是一个 BSON 文档，可嵌套子文档与数组。
- **`_id`**：每个文档在集合内唯一；未显式指定时由服务端生成 `ObjectId`。
- **类型敏感**：同一字段在不同文档中类型可以不同，查询与索引需留意类型一致性问题。

-----------

## 安装与连接（概念级）

官方提供社区版与云服务 Atlas。本地学习常见步骤：安装服务端、使用 **`mongosh`** 连接 `mongodb://host:port`，默认库为 `test` 或业务库名。

连接 URI 常见形式：

```
mongodb://用户名:密码@主机:端口/数据库名?参数=值
```

参数如认证机制、副本集名称等按部署环境配置，具体以官方文档为准。

-----------

## 数据库与集合操作

在 `mongosh` 中常用命令示例（语法以当前版本文档为准）：

```javascript
show dbs
use mydb
db.createCollection("orders")
show collections
db.orders.drop()
```

`use mydb` 表示切换命名空间；若库中尚无数据，部分列表操作可能暂不显示该库，直到有实际写入。

-----------

## CRUD 入门

以下示例集合名为 `users`，字段含 `name`、`age`、`tags`（数组）。

### 插入（Create）

```javascript
db.users.insertOne({ name: "Alice", age: 25, tags: ["dev"] })
db.users.insertMany([
  { name: "Bob", age: 30 },
  { name: "Carol", age: 22, tags: ["ops", "dba"] }
])
```

### 查询（Read）

```javascript
db.users.find({ age: { $gte: 22 } })
db.users.findOne({ name: "Alice" })
db.users.find({}, { name: 1, age: 1, _id: 0 })
```

常用比较运算符：`$eq` `$ne` `$gt` `$gte` `$lt` `$lte` `$in` `$nin`。  
逻辑组合：`$and` `$or` `$not` `$nor`。

### 更新（Update）

```javascript
db.users.updateOne(
  { name: "Alice" },
  { $set: { age: 26 }, $addToSet: { tags: "lead" } }
)
db.users.updateMany(
  { age: { $lt: 25 } },
  { $inc: { score: 1 } }
)
```

`$set` 增改字段，`$unset` 删除字段，`$push` `$pull` 操作数组，`$inc` 数值自增。

### 删除（Delete）

```javascript
db.users.deleteOne({ name: "Bob" })
db.users.deleteMany({ age: { $lt: 20 } })
```

-----------

## 索引基础

索引用于加速查询与支撑部分排序；无合适索引时，大集合上的全表扫描代价会明显升高。

### 创建单列与复合索引

```javascript
db.users.createIndex({ name: 1 })
db.users.createIndex({ age: 1, name: -1 })
```

`1` 表示升序，`-1` 表示降序。复合索引字段顺序影响前缀匹配规则。

### 唯一索引

```javascript
db.users.createIndex({ name: 1 }, { unique: true })
```

### 查看与删除索引

```javascript
db.users.getIndexes()
db.users.dropIndex("age_1_name_-1")
```

### `explain` 入门

```javascript
db.users.find({ age: { $gte: 22 } }).explain("executionStats")
```

关注是否走索引、扫描文档数与实际返回行数之比，用于判断是否需要调整查询或索引。

-----------

## 聚合管道（Aggregation Pipeline）

聚合将多阶段操作串联，上一阶段输出作为下一阶段输入。常用阶段：

| 阶段 | 作用 |
|------|------|
| `$match` | 过滤文档，类似 find 条件 |
| `$project` | 选择或计算输出字段 |
| `$group` | 分组聚合，常配合 `_id` 与累加器 |
| `$sort` | 排序 |
| `$limit` / `$skip` | 限制与跳过条数 |

示例：按 `city` 统计人数并取前 5 名。

```javascript
db.users.aggregate([
  { $match: { age: { $gte: 18 } } },
  { $group: { _id: "$city", cnt: { $sum: 1 } } },
  { $sort: { cnt: -1 } },
  { $limit: 5 }
])
```

-----------

## 常见问题

### 1. 集合需要预先定义字段吗？

不需要强制 schema，但生产环境通常用 JSON Schema 校验或应用层约束，避免脏数据长期堆积。

### 2. `_id` 能自定义吗？

可以。插入时显式提供唯一 `_id` 即可；需保证在集合内唯一。

### 3. 更新操作为何要区分 `updateOne` 与 `replaceOne`？

`updateOne` 默认使用更新运算符修改部分字段；`replaceOne` 会用新文档整体替换匹配文档（除 `_id`），语义不同，误用易导致字段丢失。

-----------

## 总结

MongoDB 的日常开发主线是：**文档与集合建模、CRUD、索引与 explain、聚合管道分阶段加工**。

下一篇从连接与读写关注、副本集与分片概念、事务边界与常见工程坑继续展开，与本文形成「开发用法」与「部署与一致性」两段式阅读路径。
