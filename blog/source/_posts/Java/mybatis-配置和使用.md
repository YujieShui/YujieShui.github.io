---
title: mybatis 配置和使用
date: 2017-11-25 19:09:09
categories:
- java
tags:
- java
- mybatis
---

MyBatis 是一款一流的支持自定义 SQL、存储过程和高级映射的持久化框架。MyBatis 几乎消除了所有的 JDBC 代码，也基本不需要手工去设置参数和获取检索结果。MyBatis 能够使用简单的 XML 格式或者注解进行来配置，能够映射基本数据元素、Map 接口和 POJOs（普通 java 对象）到数据库中的记录。

**简而言之：Mybatis 是一个半自动化的持久化框架，帮助我们简化对数据库的操作。**

<!-- more -->

# 封装 SqlSession

Mybatis 使用 SqlSession 调用他封装的一系列方法，而 SqlSession 则交由 SqlSessionFactory 进行管理。

## 注入 SqlSession

第一步就是配置 SqlSessionFactory 和 SqlSession。用 spring 和容易就能实现，在 spring 的配置文件`mybatis-spring.xml`中注入 bean:

```
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <property name="configLocation" value="classpath:mybatis-config.xml" />
        <property name="mapperLocations" value="classpath*:mapper/*.xml" />
    </bean>

    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg index="0" ref="sqlSessionFactory" />
    </bean>
```

其中`configLocation`指向 mybatis 的配置文件，我们使用配置文件可以配置别名，别名可以用来指向一个实体类：

```
<configuration>
    <typeAliases>
        <!-- 指定包名 -->
        <package name="com.shuiyujie.persist"/>
    </typeAliases>
</configuration>
```

```
<configuration>
    <typeAliases>
        <!-- 指定类名 -->
        <typeAlias alias="city" type="com.shuiyujie.persist.City"/>
</configuration>
```

其中`mapperLocations`指定了 mybatis 的 Mapper 文件，配置之后会扫描指定包下的 XML 文件。`dataSource`指向配置的数据源。

## 封装 SqlSession

SqlSession 可以调用 mybatis 为我们封装的一系列增删改查操作，具体可以看[参考文档](http://www.mybatis.org/mybatis-3/java-api.html)。

我们可以将它封装一下变得更加友好。

```java

@Component
public class DaoRouter implements IDaoRouter{
    /**
     * 普通查询
     * @param statement sql语句定义的id
     * @param parameters 参数
     * @return
     */
    public <T> List<T> query(String statement, Object parameters) {
        return sqlSession.selectList(statement, parameters);
    }

    /**
     * 分页查询，注意此处不需要在sql语句中定义startRow和endRow，如果定义，那就用上面的普通查询即可。
     * @param statement
     * @param parameters
     * @param offset
     * @param limit
     * @return
     */
    public <T> List<T> query(String statement, Object parameters, int offset, int limit) {
        return sqlSession.selectList(statement, parameters,new RowBounds(offset,limit));
    }

    /**
     * 返回第一条记录查询结果，如果没有就返回null
     * @param statementName
     * @param parameters
     * @return
     */
    public <T> T queryForObject(String statementName, Object parameters) {
        return sqlSession.selectOne(statementName, parameters);
    }

    /**
     * 更新数据
     * @param statement
     * @param parameters
     * @return
     */
    public int update(String statement, Object parameters) {
        return sqlSession.update(statement, parameters);
    }

    /**
     * 删除
     * @param statement
     * @param parameters
     * @return
     */
    public int delete(String statement, Object parameters) {
        return sqlSession.delete(statement, parameters);
    }

    /**
     * 新增数据，建议在定义sql语句的时候，使用selectKey保证可以返回新增后的主键
     * @param statement
     * @param parameters
     * @return
     */
    public int insert(String statement, Object parameters) {
        return sqlSession.insert(statement, parameters);
    }

    /**
     * 批量删除
     * @param statementName
     * @param dataList
     */
    public void deleteBatch(String statementName, Collection<?> dataList) {
        for (Object data : dataList) {
            this.delete(statementName, data);
        }
    }

    /**
     * 批量新增
     * @param statementName
     * @param dataList
     */
    public void insertBatch(String statementName, Collection<?> dataList) {
        for (Object data : dataList) {
            this.insert(statementName, data);
        }
    }

    /**
     * 批量修改
     * @param statementName
     * @param dataList
     */
    public void updateBatch(String statementName, Collection<?> dataList) {
        for (Object data : dataList) {
            this.update(statementName, data);
        }
    }

    @Autowired
    private SqlSession sqlSession;
}
```
# 查询

首先创建两个实体类省和市，他们之间为一对多的关系。

```java
public class Province implements Serializable{

    private Long id;

    private String name;

    private String description;

    private List<City> cityList;
```

```java
public class City implements Serializable{

    private Long id;

    private Long provinceId;

    private String cityName;

    private String description;

    private Province province;
```

## 一对一查询

现在要查询一个城市，同时查询出该城市所在的省。也就是获取 City 对象的时候，同时返回他的 province 成员变量。我们要使用`resultMap`中的`association`元素。

采用关联查询的方式，sql 语句如下：

```
select *
        from city c
        LEFT JOIN province p ON c.province_id = p.id
```

这种情况下有可能是关联查询一整个对象，比如 province 的所有变量；也有可能只想查询 province 的少数变量，比如说查个名字。

**提供有两种解决方案。**

### association 关联一个对象查询

关联查询时在查出 city 表中的结果集时也会查出 province 表的结果集，得到两个结果集之后要做的事情就是将他们各自映射到各自的对象中，mybatis 采用在`resultMap`中指定`association`的方式来实现以上效果。

*注：字段名出现重复的现象可以为其指定唯一的别名*

**City.java**

```java
public class City implements Serializable {

    @Id
    @GeneratedValue
    private Long id;
    
    @Column(nullable = false)
    private Long provinceId;

    @Column(nullable = false)
    private String cityName;

    private String description;

    private Province province;
```

`private Province province;`在 Ctiy 中加一个 Province 对象，查询 City 时也查询 Province。

**CityMapper.xml**

```xml
    <resultMap id="BaseResultMap" type="city">
        <result column="id" property="id" />
        <result column="province_id" property="provinceId" />
        <result column="city_name" property="cityName" />
        <result column="description" property="description" />
        <association property="province" column="province_id" javaType="province"
                     resultMap="Province.provinceResult"/>
    </resultMap>


<select id="loadCity" resultMap="BaseResultMap" >
        select *
        from city c
        LEFT JOIN province p ON c.province_id = p.id
    </select>
```

**ProvinceMapper.xml**

```xml
<select id="loadProvinceAndCity" resultMap="provinceResult" parameterType="Long">
        SELECT * FROM province
        WHERE id = #{id}
    </select>
```

重点来看`CityMapper.xml`的`association`

- property：表示变零名称，即 City 对象中的 province 属性
- column：关联的字段
- javaType:关联对象的类型
- resultMap:关联对象的结果集

### 扩展 ResultMap 来扩展字段

这次不查整个 Province 只是查询 Province 的一个 name，可以不用`association`。

**CityVO.java**

```java
public class CityVO extends City {
    private String provinceName;
```

给`City.java`一个包装类，添加成员变量 provinceName。

**扩展 ResultMap:**

```xml
<resultMap id="cityResultMap" type="city">
        <result column="id" property="id" />
        <result column="province_id" property="provinceId" />
        <result column="city_name" property="cityName" />
        <result column="description" property="description" />
    </resultMap>

    <resultMap id="cityProvinceResultMap" extends="cityResultMap" type="cityVO">
        <result column="name" property="provinceName" />
    </resultMap>
```

注意`cityProvinceResultMap`:

- extends:表示继承关系，是被继承 ResultMap 的扩展
- column: sql 语句查询出来的需要映射的字段名称
- property: 实体类中对应的成员变量的名称

**XML 查询语句:**

```xml
<select id="loadCityAndProvinceName" resultMap="cityProvinceResultMap" >
        select c.*,p.name
        from city c
        LEFT JOIN province p ON c.province_id = p.id
    </select>
```

## 集合嵌套查询

现在要在查询省的时候，返回该省下市的信息。也就是获取 Province 对象的时候，同时返回他的 cityList 成员变量。我们要使用`resultMap`中的`collection`元素。

**ProvinceMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="Province">
    <resultMap id="provinceResult" type="province">
        <result column="id" property="id"/>
        <result column="name" property="name"/>
        <result column="description" property="description"/>

        <collection property="cityList" javaType="ArrayList" column="id" ofType="city"
                    select="City.loadCityForProvince"/>
    </resultMap>

    <select id="loadProvinceAndCity" resultMap="provinceResult" parameterType="Long">
        SELECT * FROM province
        WHERE id = #{id}
    </select>
</mapper>
```

**CityMapper.xml**

```xml
<mapper namespace="City">
    <select id="loadCityForProvince" resultType="city">
        select * from city
        WHERE province_id = #{id};
    </select>
```

重点关注`ProvinceMapper.xml`的`collection`部分，他表示返回集合类型

```xml
<collection property="cityList" javaType="ArrayList" column="id" ofType="city"
                    select="City.loadCityForProvince"/>
```

- property:匹配的属性名称，也就是`Province`中的`cityList`成员变量
- javaType：集合的类型(可以省略)
- column:关联的字段
- ofType:集合中包含元素的类型，这里用了别名
- select：关联查询的方法

## 集合迭代器查询

比如想查询几个省中的市，可以传入一个包含省数据的 list，然后使用 mybatis 迭代遍历这个 list 集合查询获取所有下属的市。

要用到一个动态 sql 的标签`<foreach>`,XML 如下：

```xml
<select id="loadCityByProvinceList" resultType="city">
        select * from city
        WHERE province_id IN
        <foreach item="item" index="index" collection="list"
                 open="(" separator="," close=")">
            #{item}
        </foreach>
    </select>
```

# 插入

## 插入设置主键自增

```
<insert id="insertXXX" parameterType="fileNameVO"
            useGeneratedKeys="true" keyProperty="id">
        INSERT INTO file_name (
            ...
        )VALUES(
            ...
        )
    </insert>

```

- useGeneratedKeys：是否自增
- keyProperty：主键名称

# 注解方式调用 sql

注解方式开发

```java
package org.mybatis.example;
public interface BlogMapper { 
    @Select("SELECT * FROM blog WHERE id = #{id}”) 
    Blog selectBlog(int id); 
}
```

简单的 sql 语句可以用注解，但是复杂的不适宜用注解开发的方式。一般采用 XML 方式进行开发，有必要知道有注解开发这种方式。














