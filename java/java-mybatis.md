# mybatis

MyBatis是一个基于Java的持久层框架，它支持`自定义 SQL`、`存储过程`以及`高级映射`。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

持久层：可以将业务数据存储到磁盘，具备长期存储能力，只要磁盘不损坏，在断电或者其他情况下，重新开启系统仍然可以读取到这些数据。

- 优点： 可以使用巨大的磁盘空间存储相当量的数据，并且很廉价
- 缺点：慢（相对于内存而言）

## 执行流程

1. mapper.xml中的配置⽂件⾥的每条sql语句，每一个\<select\>、\<insert\>、\<update\>、\<delete\>标签，都会被解析为带有Id信息的一个个MappedStatement对象，再通过⼀个HashMap集合保存起来。
2. Mapper接口是没有实现类的，当调用接口方法时，接口全限名+方法名拼接字符串作为 key 值，可唯一定位一个MappedStatement。
3. 执⾏getMapper()⽅法，判断是否注册过mapper接⼝，注册了就会使⽤mapperProxyFactory去⽣成代理类MapperProxy执⾏⽬标⽅法时，会调⽤MapperProxy代理类的invoke()⽅法
4. 此时会使用boundSql和对应的mapperStatement构造cacheKey,先进行缓存查询，命中直接返回。缓存Map\<Method, MapperMethodInvoker\> methodCache =  
5. 缓存无命中,则创建connect连接，通过statement对象执行execute方法。
6. 执⾏execute()⽅法返回结果使用resultSetHandler进行结果集的封装，添加到缓存中，最后返回结果。

![mybatis](!/../pic/mybatisgo.png)

## mybatis与Hibernate区别

1. 编写sql方面：hibernate不需要自己写sql语句，只需要写hql语句。而mybatis需要自己在配置文件中写sql语句，对开发人员的sql要求较高。
2. sql优化方面：由于hibernate自动生成sql语句，生成的语句开发人员不易优化。而mybatis的sql完全体现在配置文件中，就便于优化。
3. 数据库迁移方面：由于hibernate的sql是自动生成的，会根据不同的数据库生成对应的语法，迁移性较高。而mybatis的因为都是写在配置文件，迁移数据库，就可能造成语法不支持的情况。
4. 日志方面：hibernate拥有完整的日志系统，包括：sql记录、关系异常、优化警告、缓存提示、脏数据警告等。而mybatis仅有基本的记录功能。
5. 缓存方面：hibernate有更好的二级缓存机制，可以使用第三方缓存。而mybatis支持两级缓存，实际应用中应用性不高。
    - mybatis一级缓存sqlSession级的缓存。二级缓存Mapper级别的缓存。
    - hibernate一级缓存session级别的。Hibernate二级缓存是SessionFactory级的缓存。 SessionFactory的缓存分为内置缓存和外置缓存

## mybatis的缓存

一级缓存的作用域是SQlSession, Mabits默认开启一级缓存。 在同一个SqlSession中，执行相同的SQL查询时；第一次会去查询数据库，并写在缓存中，第二次会直接从缓存中取。 当执行SQL时候两次查询中间发生了增删改的操作，则SQLSession的缓存会被清空。

### 一级缓存

`为 ​sqlsesson​ 缓存，缓存的数据只在 SqlSession 内有效`。在操作数据库的时候需要先创建 SqlSession 会话对象，在对象中有一个 HashMap 用于存储缓存数据，此 HashMap 是当前会话对象私有的，别的 SqlSession 会话对象无法访问。

- 第一次执行 select 完毕会将查到的数据写入 SqlSession 内的 HashMap 中缓存起来
- 第二次执行 select 会从缓存中查数据，如果 select 同传参数一样，那么就能从缓存中返回数据，不用去数据库了，从而提高了效率

### 二级缓存

是​ mapper​ 级别的缓存，也就是同一个 namespace 的 mapper.xml ，当多个 SqlSession 使用同一个 Mapper 操作数据库的时候，得到的数据会缓存在同一个二级缓存区域

## MyBatis和ORM的区别

mybatis属于半orm，因为sql语句需要自己写。

与其他比较标准的 ORM 框架（比如 Hibernate ）不同，`mybatis 并没有将 java 对象与数据库关联起来，而是将 java 方法与 sql 语句关联起来`，mybatis 允许用户充分利用数据库的各种功能，例如存储、视图、各种复杂的查询以及某些数据库的专有特性。

自己写 sql 语句的好处: 可以根据自己的需求，写出最优的 sql 语句。灵活性高。但是，由于是自己写 sql 语句，导致平台可移植性不高。MySQL 语句和 Oracle 语句不同
