# Mybatis连接池



## mybaits中的连接池的分类

mybaits中的连接池提供三种配置方式：

| POOLED | UNPOLLED | JNDI |
| :----: | :------: | :--: |
|        |          |      |

MyBatis 内部分别定义了实现了 java.sql.DataSource 接口的 UnpooledDataSource， PooledDataSource 类来表示 UNPOOLED、POOLED 类型的数据源

在这三种数据源中，我们一般采用的是 POOLED 数据源（很多时候我们所说的数据源就是为了更好的管理数据 库连接，也就是我们所说的连接池技术）。

## Mybatis中的事务与自动提交



使用mybatis中的自动提交只需要在opensession时传入一个true参数即可实现自动提交