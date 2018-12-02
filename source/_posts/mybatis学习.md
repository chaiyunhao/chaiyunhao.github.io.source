---
title: mybatis学习
date: 2018-12-02 12:27:19
tags:
    - Java
    - Mybatis
---

### 传统JDBC 的弊端

1. 底层没有连接池，操作数据库需要频繁的创建和关闭链接，消耗资源
2. 远程JDBC修改sql需要整体编译，不利于系统维护
3. 使用PreparedStatement预编译对变量设置序号，这样的序号不利于维护
4. 返回结果集result需要硬编码

### ORM框架MyBatis

Object Relation Mapping  对象关系映射
#### 引入Maven依赖

```xml
        <!-- mybatis -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.6</version>
        </dependency>
        <!-- mysql 数据库连接 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.38</version>
        </dependency>
```

#### 添加mybatis-config .xml文件

| 属性名      | 作用    |
| --------- | -------- |
| properties    | 系统属性 |
| settings     | 用于修改Mybatis运行时行为     |
| typeAliases     | 类别名 |
| typeHanders         |  将预编译语句或者结果集中的JDBC类型转化为java类型 |
| objectFactory | 提供默认构造器或者执行构造参数初始化目标类型的对象    |
| plugins           |  拦截映射        |
| environments | 允许配置多个环境  |
| databaseIdProvider          | 数据库标识提供商          |
| mappers          | sql映射文件          |

* 要注意以上属性在xml文件中存在顺序要求，dtd 文件中定义了属性的顺序 

*  MyBatis 允许你在已映射语句执行过程中的某一点进行拦截调用。默认情况下，MyBatis 允许使用插件来拦截的方法调用包括： 
1. Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
2. ParameterHandler (getParameterObject, setParameters)
3. ResultSetHandler (handleResultSets, handleOutputParameters)
4. StatementHandler (prepare, parameterize, batch, update, query)

*  mappers：
1. 使用相对于类路径的资源引用

```xml
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
</mappers>
```

2. 使用完全限定资源定位符（URL）

```xml
<mappers>
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>
</mappers>
```

3. 使用映射器接口实现类的完全限定类名

```xml
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
</mappers>
```

4. 将包内的映射器接口实现全部注册为映射器

```xml
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
```

#### 构建Mapping文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="mybatis.OrderNodeInstancesMapper">
  <resultMap id="BaseResultMap" type="cn.xingchen.model.OrderNodeInstances">
    <id column="id" jdbcType="BIGINT" property="id" />
    <result column="fid" jdbcType="BIGINT" property="fid" />
    <result column="names" jdbcType="VARCHAR" property="names" />
    <result column="desc" jdbcType="VARCHAR" property="desc" />
    <result column="created_at" jdbcType="TIMESTAMP" property="created_at" />
    <result column="updated_at" jdbcType="TIMESTAMP" property="updated_at" />
    <result column="first_node" jdbcType="TINYINT" property="first_node" />
  </resultMap>
  <sql id="Base_Column_List">
    id, fid, names, `desc`, created_at, updated_at, first_node
  </sql>
  <select id="selectByPrimaryKey" parameterType="java.lang.Long" resultMap="BaseResultMap">
    select 
    <include refid="Base_Column_List" />
    from parana_order_node_instances
    where id = #{id,jdbcType=BIGINT}
  </select>
  <delete id="deleteByPrimaryKey" parameterType="java.lang.Long">
    delete from parana_order_node_instances
    where id = #{id,jdbcType=BIGINT}
  </delete>
   <insert id="insert" parameterType="cn.xingchen.model.OrderNodeInstances">
    insert into parana_order_node_instances (id, fid, names, 
      desc, created_at, updated_at, 
      first_node)
    values (#{id,jdbcType=BIGINT}, #{fid,jdbcType=BIGINT}, #{names,jdbcType=VARCHAR}, 
      #{desc,jdbcType=VARCHAR}, #{created_at,jdbcType=TIMESTAMP}, #{updated_at,jdbcType=TIMESTAMP}, 
      #{first_node,jdbcType=TINYINT})
  </insert>
  <update id="updateByPrimaryKeySelective" parameterType="cn.xingchen.model.OrderNodeInstances">
    update parana_order_node_instances
    <set>
      <if test="fid != null">
        fid = #{fid,jdbcType=BIGINT},
      </if>
      <if test="names != null">
        names = #{names,jdbcType=VARCHAR},
      </if>
      <if test="desc != null">
        desc = #{desc,jdbcType=VARCHAR},
      </if>
      <if test="created_at != null">
        created_at = #{created_at,jdbcType=TIMESTAMP},
      </if>
      <if test="updated_at != null">
        updated_at = #{updated_at,jdbcType=TIMESTAMP},
      </if>
      <if test="first_node != null">
        first_node = #{first_node,jdbcType=TINYINT},
      </if>
    </set>
    where id = #{id,jdbcType=BIGINT}
  </update>
</mapper>
```

#### 构建注解的Interface文件

```java

import cn.xingchen.model.OrderNodeInstances;
import cn.xingchen.model.OrderNodeInstancesExample;
import java.util.List;
import org.apache.ibatis.annotations.Delete;
import org.apache.ibatis.annotations.DeleteProvider;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.InsertProvider;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.annotations.Result;
import org.apache.ibatis.annotations.Results;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.annotations.SelectProvider;
import org.apache.ibatis.annotations.Update;
import org.apache.ibatis.annotations.UpdateProvider;
import org.apache.ibatis.type.JdbcType;

public interface OrderNodeInstancesMapper {
    @SelectProvider(type=OrderNodeInstancesSqlProvider.class, method="countByExample")
    long countByExample(OrderNodeInstancesExample example);

    @DeleteProvider(type=OrderNodeInstancesSqlProvider.class, method="deleteByExample")
    int deleteByExample(OrderNodeInstancesExample example);

    @Delete({
        "delete from parana_order_node_instances",
        "where id = #{id,jdbcType=BIGINT}"
    })
    int deleteByPrimaryKey(Long id);

    @Insert({
        "insert into parana_order_node_instances (id, fid, names, ",
        "`desc`, created_at, ",
        "updated_at, first_node)",
        "values (#{id,jdbcType=BIGINT}, #{fid,jdbcType=BIGINT}, #{names,jdbcType=VARCHAR}, ",
        "#{`desc`,jdbcType=VARCHAR}, #{created_at,jdbcType=TIMESTAMP}, ",
        "#{updated_at,jdbcType=TIMESTAMP}, #{first_node,jdbcType=TINYINT})"
    })
    int insert(OrderNodeInstances record);

    @InsertProvider(type=OrderNodeInstancesSqlProvider.class, method="insertSelective")
    int insertSelective(OrderNodeInstances record);

    @SelectProvider(type=OrderNodeInstancesSqlProvider.class, method="selectByExample")
    @Results({
        @Result(column="id", property="id", jdbcType=JdbcType.BIGINT, id=true),
        @Result(column="fid", property="fid", jdbcType=JdbcType.BIGINT),
        @Result(column="names", property="names", jdbcType=JdbcType.VARCHAR),
        @Result(column="`desc`", property="`desc`", jdbcType=JdbcType.VARCHAR),
        @Result(column="created_at", property="created_at", jdbcType=JdbcType.TIMESTAMP),
        @Result(column="updated_at", property="updated_at", jdbcType=JdbcType.TIMESTAMP),
        @Result(column="first_node", property="first_node", jdbcType=JdbcType.TINYINT)
    })
    List<OrderNodeInstances> selectByExample(OrderNodeInstancesExample example);

    @Select({
        "select",
        "id, fid, names, `desc`, created_at, updated_at, first_node",
        "from parana_order_node_instances",
        "where id = #{id,jdbcType=BIGINT}"
    })
    @Results({
        @Result(column="id", property="id", jdbcType=JdbcType.BIGINT, id=true),
        @Result(column="fid", property="fid", jdbcType=JdbcType.BIGINT),
        @Result(column="names", property="names", jdbcType=JdbcType.VARCHAR),
        @Result(column="`desc`", property="`desc`", jdbcType=JdbcType.VARCHAR),
        @Result(column="created_at", property="created_at", jdbcType=JdbcType.TIMESTAMP),
        @Result(column="updated_at", property="updated_at", jdbcType=JdbcType.TIMESTAMP),
        @Result(column="first_node", property="first_node", jdbcType=JdbcType.TINYINT)
    })
    OrderNodeInstances selectByPrimaryKey(Long id);

    @UpdateProvider(type=OrderNodeInstancesSqlProvider.class, method="updateByExampleSelective")
    int updateByExampleSelective(@Param("record") OrderNodeInstances record, @Param("example") OrderNodeInstancesExample example);

    @UpdateProvider(type=OrderNodeInstancesSqlProvider.class, method="updateByExample")
    int updateByExample(@Param("record") OrderNodeInstances record, @Param("example") OrderNodeInstancesExample example);

    @UpdateProvider(type=OrderNodeInstancesSqlProvider.class, method="updateByPrimaryKeySelective")
    int updateByPrimaryKeySelective(OrderNodeInstances record);

    @Update({
        "update parana_order_node_instances",
        "set fid = #{fid,jdbcType=BIGINT},",
          "names = #{names,jdbcType=VARCHAR},",
          "`desc` = #{desc,jdbcType=VARCHAR},",
          "created_at = #{created_at,jdbcType=TIMESTAMP},",
          "updated_at = #{updated_at,jdbcType=TIMESTAMP},",
          "first_node = #{first_node,jdbcType=TINYINT}",
        "where id = #{id,jdbcType=BIGINT}"
    })
    int updateByPrimaryKey(OrderNodeInstances record);
}
```

#### 测试执行
* xml

```java
try {
            String resource = "mybatis-config.xml";
            InputStream inputStream  = Resources.getResourceAsStream(resource);
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
            SqlSession sqlSession = sqlSessionFactory.openSession();
            OrderNodeInstances orderNodeInstances = sqlSession.selectOne("mybatis.OrderNodeInstancesMapper.selectByPrimaryKey",1L);
            System.out.println(orderNodeInstances.getId());
        } catch (IOException e) {
            e.printStackTrace();
        }
```

* mapper

```java
try {
            String resource = "mybatis-config.xml";
            InputStream inputStream  = Resources.getResourceAsStream(resource);
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
            SqlSession sqlSession = sqlSessionFactory.openSession();
            OrderNodeInstancesMapper mapper = sqlSession.getMapper(OrderNodeInstancesMapper.class);
            OrderNodeInstances orderNodeInstances = mapper.selectByPrimaryKey(1L);
            System.out.println(orderNodeInstances.getId());
        } catch (IOException e) {
            e.printStackTrace();
        }
```


#### Mybatis 注解和xml优缺点

xml：增加xml文件麻烦、条件不确定、容易出错、特殊字符转义
注解：不适合复杂sql，收集sql复杂，需要重新编译

#### $ 和 # 区别
`#` 预编译，防止sql注入
`$` 作为参数传入，无法避免sql注入


### 逆向工程

通过数据库中表，生成pojo和mapper
#### 引入maven的plugins

```xml
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.7</version>
                <!-- 将 mgb 加入 maven 构建 -->
                <executions>
                    <execution>
                        <id>Generate MyBatis Artifacts</id>
                        <goals>
                            <goal>generate</goal>
                        </goals>
                    </execution>
                </executions>

                <dependencies>
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        <version>5.1.38</version>
                    </dependency>
                    <dependency>
                        <groupId>org.mybatis.generator</groupId>
                        <artifactId>mybatis-generator-core</artifactId>
                        <version>1.3.7</version>
                    </dependency>
                    <dependency>
                        <groupId>org.mybatis</groupId>
                        <artifactId>mybatis</artifactId>
                        <version>3.4.6</version>
                    </dependency>
                </dependencies>
            </plugin>
```

#### 在 resources 目录配置 generatorConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>

    <context id="MySqlTables" targetRuntime="MyBatis3">

        <commentGenerator>
            <property name="suppressDate" value="true"/>
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://127.0.0.1:3306/inyes"
                        userId="root"
                        password="123456">
        </jdbcConnection>
        <javaTypeResolver >
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>

        <javaModelGenerator targetPackage="cn.xingchen.model" targetProject="/Users/leaf/IdeaProjects/xingchen-stu/stu-mybatis/src/main/java">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>

        <sqlMapGenerator targetPackage="mybatis"  targetProject="/Users/leaf/IdeaProjects/xingchen-stu/stu-mybatis/src/main/resources">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>

        <!--<javaClientGenerator type="XMLMAPPER" targetPackage="cn.xingchen.test.mapper"  targetProject="/Users/leaf/IdeaProjects/xingchen-stu/stu-mybatis/src/main/java">-->
            <!--<property name="enableSubPackages" value="true" />-->
        <!--</javaClientGenerator>-->

        <javaClientGenerator type="ANNOTATEDMAPPER" targetPackage="cn.xingchen.mapper"  targetProject="/Users/leaf/IdeaProjects/xingchen-stu/stu-mybatis/src/main/java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>

        <table schema="inyes" tableName="parana_order_node_instances" domainObjectName="OrderNodeInstances" >
            <property name="useActualColumnNames" value="true"/>
        </table>
    </context>
</generatorConfiguration>
```


#### 配置好相应目录以及数据库后执行
    mvn mybatis-generator:generate

### Mybatis 核心概念

|名称          |意义 |
|--------------|----|
|Configuration |管理mysql-config.xml全局配置关系类|
|SqlSessionFactory|Session工厂接口|
|Session|面向用户操作数据库的接口|
|Executor|sqlSession操作执行器|
|MappedStatement|对操作数据库存储封装，包括sql语句、输入输出参数|
|StatementHandler|操作数据库相关的handler接口|
|ResultSetHandler|操作数据库返回结果的handler接口|

### 源码分析

1. Mybatis 之 Configuration org.apache.ibatis.builder.xml.XMLConfigBuilder#parseConfiguration
2. Properties 文件解析: org.apache.ibatis.builder.xml.XMLConfigBuilder#propertiesElement
3. Setting 文件解析:
org.apache.ibatis.builder.xml.XMLConfigBuilder#settingsElement
4. environments 文件解析:
org.apache.ibatis.builder .xml.XMLConfigBuilder#environmentsElement


#### Mapper 执行sql过程

##### 通过解析 config 返回 SqlSessionFactory

```
org.apache.ibatis.session.SqlSessionFactoryBuilder.build(java.io.InputStream)
 >org.apache.ibatis.builder.xml.XMLConfigBuilder 构造函数
   >org.apache.ibatis.builder.xml.XMLConfigBuilder.parse
     >org.apache.ibatis.builder.xml.XMLConfigBuilder.parseConfiguration mybatis-config.xml内容
       >org.apache.ibatis.parsing.XPathParser.evaluate
        >org.apache.ibatis.builder.xml.XMLConfigBuilder.mapperElement
         >org.apache.ibatis.session.SqlSessionFactoryBuilder.build(org.apache.ibatis.session.Configuration)
          >org.apache.ibatis.session.defaults.DefaultSqlSessionFactory.DefaultSqlSessionFactory
```

#####  拿到SqlSession 进行对执行器进行初始化 SimpleExecutor

```
org.apache.ibatis.session.defaults.DefaultSqlSessionFactory.openSession()
 >org.apache.ibatis.session.defaults.DefaultSqlSessionFactory.openSessionFromDataSource
  >org.apache.ibatis.transaction.TransactionFactory.newTransaction(javax.sql.DataSource, org.apache.ibatis.session.TransactionIsolationLevel, boolean)
    >org.apache.ibatis.session.Configuration.newExecutor(org.apache.ibatis.transaction.Transaction, org.apache.ibatis.session.ExecutorType)
      >org.apache.ibatis.executor.SimpleExecutor
        >org.apache.ibatis.executor.CachingExecutor  一级缓存 自动
          >org.apache.ibatis.plugin.InterceptorChain.pluginAll 
```

##### 拦截器

如果需要可以在这里添加相应拦截器，比如日志打印sql等

##### 操作数据库

```
org.apache.ibatis.session.defaults.DefaultSqlSession.selectOne(java.lang.String, java.lang.Object)
 >org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(java.lang.String, java.lang.Object)
   >org.apache.ibatis.session.Configuration.getMappedStatement(java.lang.String)
    >org.apache.ibatis.executor.CachingExecutor.query(org.apache.ibatis.mapping.MappedStatement, java.lang.Object, org.apache.ibatis.session.RowBounds, org.apache.ibatis.session.ResultHandler)
     >org.apache.ibatis.executor.CachingExecutor.createCacheKey 缓存的key
      >org.apache.ibatis.executor.CachingExecutor.query(org.apache.ibatis.mapping.MappedStatement, java.lang.Object, org.apache.ibatis.session.RowBounds, org.apache.ibatis.session.ResultHandler, org.apache.ibatis.cache.CacheKey, org.apache.ibatis.mapping.BoundSql)
         >org.apache.ibatis.executor.BaseExecutor.queryFromDatabase
           >org.apache.ibatis.executor.BaseExecutor.doQuery
             >org.apache.ibatis.executor.statement.PreparedStatementHandler.query
               >org.apache.ibatis.executor.resultset.ResultSetHandler.handleResultSets
                >org.apache.ibatis.executor.resultset.DefaultResultSetHandler
  
```

_缓存 cache key: id +sql+limit+offsetxxx_

#### @Select 注解执行sql过程

##### SqlSessionFactory 创建

```    >org.apache.ibatis.session.SqlSessionFactoryBuilder.build(java.io.InputStream)  读取xml配置文件
        >org.apache.ibatis.builder.xml.XMLConfigBuilder.XMLConfigBuilder(java.io.InputStream, java.lang.String, java.util.Properties) 解析xml
            >org.apache.ibatis.session.SqlSessionFactoryBuilder.build(org.apache.ibatis.session.Configuration)
                >org.apache.ibatis.session.defaults.DefaultSqlSessionFactory.DefaultSqlSessionFactory  根据配置生成SqlSessionFactory
```
##### SqlSession 开启

```    >org.apache.ibatis.session.defaults.DefaultSqlSessionFactory.openSessionFromDataSource
        >org.apache.ibatis.transaction.jdbc.JdbcTransactionFactory
            >org.apache.ibatis.transaction.jdbc.JdbcTransaction.JdbcTransaction(javax.sql.DataSource, org.apache.ibatis.session.TransactionIsolationLevel, boolean)
                >org.apache.ibatis.executor.SimpleExecutor  默认的执行器
                >org.apache.ibatis.executor.CachingExecutor 默认开启一级缓存 对 SimpleExecutor 进行包装
                >org.apache.ibatis.plugin.InterceptorChain.pluginAll  插件(监控、埋点)
                    >org.apache.ibatis.session.defaults.DefaultSqlSession.DefaultSqlSession(org.apache.ibatis.session.Configuration, org.apache.ibatis.executor.Executor, boolean) 返回SqlSession
```

##### SqlSession 执行对应sql 返回结果集

```  >org.apache.ibatis.session.defaults.DefaultSqlSession.selectOne(java.lang.String, java.lang.Object)
        >org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(java.lang.String, java.lang.Object) 使用 selectList 查询完成如果结果集多于一条抛出异常
            >org.apache.ibatis.session.Configuration.getMappedStatement(java.lang.String, boolean)  获取MappedStatement
                >org.apache.ibatis.mapping.MappedStatement.getBoundSql
                    >org.apache.ibatis.builder.StaticSqlSource.getBoundSql  获取需要执行的sql
                        >org.apache.ibatis.executor.CachingExecutor.createCacheKey  生成缓存的key
                            >org.apache.ibatis.cache.TransactionalCacheManager.getObject   如果缓存此处返回结果不在向下执行
                                >org.apache.ibatis.executor.BaseExecutor.query(org.apache.ibatis.mapping.MappedStatement, java.lang.Object, org.apache.ibatis.session.RowBounds, org.apache.ibatis.session.ResultHandler)
                                    >org.apache.ibatis.executor.BaseExecutor.queryFromDatabase
                                        >org.apache.ibatis.executor.SimpleExecutor.doQuery
                                            >org.apache.ibatis.executor.statement.PreparedStatementHandler.PreparedStatementHandler
                                                >org.apache.ibatis.executor.statement.PreparedStatementHandler.query   调用java.sql.PreparedStatement
                                                    >org.apache.ibatis.executor.resultset.DefaultResultSetHandler.handleResultSets
                                                        >org.apache.ibatis.executor.resultset.DefaultResultSetHandler.handleRowValuesForNestedResultMap  将结果集与对象绑定
```

##### 通过 @Select 注解执行sql 需要在 SqlSession 执行之前的方法调用

```
>org.apache.ibatis.session.defaults.DefaultSqlSession.getMapper
    >org.apache.ibatis.binding.MapperRegistry.getMapper                 
        >org.apache.ibatis.binding.MapperProxyFactory.newInstance(org.apache.ibatis.session.SqlSession)
            >org.apache.ibatis.binding.MapperProxy
                >org.apache.ibatis.binding.MapperMethod.execute
                    >org.apache.ibatis.session.defaults.DefaultSqlSession.selectOne(java.lang.String, java.lang.Object) 最终调用 SqlSession 对应方法
```


