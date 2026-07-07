---
title: Mybatis
tags: [Mybatis]
categories: [Tech, Spring与框架]
toc: true
---

Mybatis知识等，如Mybatis 、原理流程、一二级缓存等。



<!-- more -->



## Mybatis 

MyBatis 是一款**优秀的持久层框架**，它本质上是一个 **ORM（对象关系映射）框架**，用于帮助我们将 **Java 对象与数据库中的 SQL 操作绑定起来**。

它的核心思想是：**通过 XML 或注解方式将 SQL 与 Java 方法进行映射**，程序员自己编写 SQL，而 MyBatis 负责将结果映射成对象并执行。

---

### 特点

1. **半自动化 ORM 框架**：不像 Hibernate 那样全自动，SQL 需要开发者自己写，更加灵活、可控。
2. **支持自定义 SQL**：适合对 SQL 性能 要求高、业务复杂的场景。
3. **支持动态 SQL**：通过 XML 标签动态生成 SQL（如 if、where、foreach）。
4. **提供映射机制**：支持将数据库结果映射为 Java 对象，支持一对一、一对多等关系映射。
5. **集成简单**：可以轻松与 Spring/Spring Boot 集成。

------

#### 与传统的JDBC相比，MyBatis的优点？

- 基于 SQL 语句编程，相当灵活，不会对应用程序或者数据库的现有设计造成任 何影响，SQL 写在 XML 里，解除 sql 与程序代码的耦合，便于统一管理；提供 XML 标签，支持编写动态 SQL 语句，并可重用。
- 与 JDBC 相比，减少了 50%以上的代码量，消除了 JDBC 大量冗余的代码，不 需要手动开关连接；
- 很好的与各种数据库兼容，因为 MyBatis 使用 JDBC 来连接数据库，所以只要 JDBC 支持的数据库 MyBatis 都支持。
- 能够与 Spring 很好的集成，开发效率高。
- 提供映射标签，支持对象与数据库的 ORM 字段关系映射；提供对象关系映射 标签，支持对象关系组件维护。

------

#### 与全自动化 ORM的区别

**全自动 ORM**

- 自动生成 SQL
- 自动管理实体状态（增删改查）
- 自动处理关联映射（1-1 / 1-N / N-N）
- 有完整的缓存、事务、脏检查等机制

开发者**不用写 SQL，操作对象就行**。

**MyBatis**

- SQL 手写 → 灵活
- 映射自动 → 简化 JDBC
- 不管理实体状态、不生成 SQL、不做关联推断

----

#### 有什么缺点

- SQL 语句的编写工作量较大，尤其当字段多、关联表多时，对开发人员编写 SQL 语句的功底有一定要求
- SQL 语句依赖于数据库，导致数据库移植性差，不能随意更换数据库

----

### 其他

#### Mybatis里的 # 和 $ 的区别？

- Mybatis 在处理 #{} 时，会创建预编译的 SQL 语句，将 SQL 中的 #{} 替换为 ? 号，在执行 SQL 时会为预编译 SQL 中的占位符（?）赋值，调用 PreparedStatement 的 set 方法来赋值，预编译的 SQL 语句执行效率高，并且可以防止SQL 注入，提供更高的安全性，适合传递参数值。
- Mybatis 在处理 ${} 时，只是创建普通的 SQL 语句，然后在执行 SQL 语句时 MyBatis 将参数直接拼入到 SQL 里，不能防止 SQL 注入，因为参数直接拼接到 SQL 语句中，如果参数未经过验证、过滤，可能会导致安全问题。

---

## 原理

### 流程/原理

- **读取配置**
  加载 `mybatis-config.xml`、映射文件、数据源等信息，创建 `SqlSessionFactory`。
- **创建 SqlSession**
  从 `SqlSessionFactory` 打开一个 `SqlSession`，本质上是获取数据库连接并准备执行环境。
- **找到对应的 MappedStatement**
  根据调用的 Mapper 方法（例如 `UserMapper.selectById`）找到对应的 SQL 配置。
- **参数处理**
  MyBatis 把方法参数绑定到 SQL 中（预编译的 `?` 占位符）。
- **执行 SQL**
  调用 JDBC 执行：`PreparedStatement` → 发给数据库 → 得到结果集。
- **结果映射**
  将结果集映射成 Java 对象（自动根据 `<resultMap>` 或字段名映射）。
- **返回对象**
  把映射后的对象返回给调用者，`SqlSession` 关闭或进入事务管理流程。

----

###  在Mapper中如何传递多个参数？  

1、若Dao层函数有多个参数，那么其对应的xml中，#{0}代表接收的是Dao层中的第一个参数，#{1}代表Dao中的第二个参数，以此类推。

2、使用@Param注解：在Dao层的参数中前加@Param注解,注解内的参数名为传递到Mapper中的参数名。

3、多个参数封装成Map，以HashMap的形式传递到Mapper中。

----

动态sql有什么用？有哪些动态sql？

 动态 SQL 用来 根据条件拼接 SQL，让查询逻辑更灵活  

### 常见动态 SQL 标签有哪些？

MyBatis 内置的动态 SQL 标签包括：

1. `**<if>**` —— 条件判断
2. `**<choose> <when> <otherwise>**` —— 类似 switch-case
3. `**<where>**` —— 自动加 WHERE，并去除多余 AND、OR
4. `**<set>**` —— 自动处理 SET 子句的逗号
5. `**<foreach>**` —— 遍历集合（常用于 IN、批量插入）
6. `**<trim>**` —— 自定义前后缀、去除多余字符
7. `**<bind>**` —— 创建新的 SQL 变量

-----

### 一级、二级缓存

1、 一级缓存：基于PerpetualCache的HashMap本地缓存，其存储作用域为Session，当Session flush或close之后，该Session中的所有Cache就将清空，默认打开一级缓存。  

2、 二级缓存与一级缓存机制相同，默认也是采用PerpetualCache，HashMap存储，不同在于其存储作用域为Mapper（namespace），并且可自定义存储源，如Ehcache。默认打不开二级缓存，要开启二级缓存，使用二级缓存属性类需要实现Serializable 序列化接口（可用来保存对象的状态），可在它的映射文件中配置。













