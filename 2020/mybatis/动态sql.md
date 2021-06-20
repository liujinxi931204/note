### 动态SQL

MyBatis令人喜欢的一大特性就是动态SQL。在使用JDBC的过程中，根据条件进行SQL的拼接是很麻烦且容易出错的事。MyBatis动态SQL的出现，解决了这个麻烦  

MyBatis通过ONGL来进行动态SQL的使用。目前MyBatis支持一下几种标签  

|          元素          |              作用              |                备注                 |
| :--------------------: | :----------------------------: | :---------------------------------: |
|         if标签         |            判断语句            |             单条件分支              |
| choose(when,otherwise) |     相当于java中的if else      |             多条件分支              |
|    trim(where,set)     |            辅助元素            |         用于处理SQL拼接问题         |
|        foreach         |            循环语句            |   批量插入、更新、查询时经常用到    |
|          bind          | 创建一个变量，并绑定到上下文中 | 用于兼容不同的数据库，防止SQL注入等 |

#### 数据准备

![image-20210619183857584](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/image-20210619183857584.png)

#### if标签  

if标签是最常用的标签之一，在查询、删除、更新的时候很可能会使用到，必须结合test属性联合使用  

##### 在WHERE条件中使用if标签  

这是一种常见的现象，当进行查询时，可能会有多种情况  

###### 查询条件  

当只输入书籍的名称时，用这一个条件进行匹配  

当只输入用户ID的时候，用这一个条件进行匹配  

当书籍的名称和用户的ID都存在时，用这两个条件进行匹配 

###### 动态SQL  

接口函数  

```java
/**
* 根据输入的条件进行检索  
* 当只输入书籍的名称时，用这一个条件进行匹配  
* 当只输入用户ID的时候，用这一个条件进行匹配  
* 当书籍的名称和用户的ID都存在时，用这两个条件进行匹配
*/
List<catalog> selectByCatalogSelective(catalog catalog);
```

对应的动态SQL为  

```xml
<select id="selectByCatalogSelective"  resultType="catalog">
    select * from t_catalog
    where 1=1
    <if test="name != null and name !=''">
        and name like concat('%',#{name},'%')
    </if>
    <if test="userId != null and userId !=''">
        and user_id  = #{userId}
    </if>
</select>
```

这里在where条件中，where 1=1是一个多条件拼接时的小技巧，后面的查询条件就都可以用and了  

同时添加了if标签来处理动态SQL  

```xml
<if test="name != null and name !=''">
    and name like concat('%',#{name},'%')
</if>
<if test="userId != null and userId !=''">
    and user_id  = #{userId}
</if>
```

此if标签的test 属性值是一个符合OGNL的表达式，表达式可以是true或者false。如果表达式返回的是数值，则0为false，1为true  

**在test条件中的是字段必须是实体类中出现的字段**  

只有name字段的查询时，发送的语句是  

```sql
select * from t_catalog where 1 = 1 and name like concat ('%','java','%')
```

只有userId字段的查询时，发送的语句是  

```sql
select * from t_catalog where 1 = 1 and user_id = 1
```

同时有name字段和userId字段时，发送的语句是  

```sql
select * from t_catalog where 1 =1 and name like concat('%','java','%') and user_id = 1
```

##### 在update更新列表中使用if标签  

有时候不希望更新所有的字段，只更新有变化的字段  

###### 更新条件  

只更新有变化的条件，空值不更新  

##### 动态SQL  

接口函数  

```java
/**
* 更新非空属性
*/
int updateByPrimaryKeySelective(catalog catalog);
```

对应的SQL  

```xml
<update id="updateByPrimaryKeySelective" parameterType="catalog">
    update t_catalog
    <set>
        <if test="name!=null and name !=''">
            name = #{name}，
        </if>
        <if test="createDate!=null and createDate !=''">
            create_date = #{createDate}，
        </if>
        <if test="status!=null and status !=''">
            status = #{status}，
        </if>
        <if test="userId!=null and userId !=''">
            user_id = #{userId}，
        </if>
        <if test="courseId!=null and courseId !=''">
            course_id = #{courseId}，
        </if>
        <if test="orderNo!=null and orderNo !=''">
            order_no = #{orderNo}，
        </if>
    </set>
    where id = #{id}
</update>
```

**注意这里set标签和if标签搭配使用**

MyBatis在生成update语句时若使用if标签，如果前面的if没有执行，则可能导致有多余的逗号的错误。使用set标签可以将动态的配置SET关键字，和剔除追加到条件末尾的任何不相关的逗号。没有使用if标签时，如果有一个参数为null，都会导致错误

##### 在insert动态插入中使用if标签  

在插入数据库中的记录时，不是每一个字段都有值的，而是动态的变化的。在这时候使用if标签可以解决这个问题  

###### 插入条件  

只有非空属性才插入  

###### 动态SQL  

```java
/**
* 非空字段才进行插入
*/
int insertByPrimaryKeySelective(catalog catalog);
```

###### 对应的SQL  

```xml
<insert id="insertByPrimaryKeySelective">
    insert into t_catalog
    <trim prefix="(" suffix=")" suffixOverrides=",">
        <if test="name!=null and name!=''">
            name,
        </if>
        <if test="createDate!=null and createDate!=''">
            create_date,
        </if>
        <if test="status!=null and status!=''">
            status,
        </if>
        <if test="userId!=null and userId!=''">
            user_id,
        </if>
        <if test="courseId!=null and courseId!=''">
            course_id,
        </if>
        <if test="orderNo!=null and orderNo!=''">
            order_no,
        </if>
    </trim>
    <trim prefix="values (" suffix=")" suffixOverrides=",">
        <if test="name!=null and name!=''">
            #{name},
        </if>
        <if test="createDate!=null and createDate!=''">
            #{createDate},
        </if>
        <if test="status!=null and status!=''">
            #{status},
        </if>
        <if test="userId!=null and userId!=''">
            #{userId},
        </if>
        <if test="courseId!=null and courseId!=''">
            #{courseId},
        </if>
        <if test="orderNo!=null and orderNo!=''">
            #{orderNo},
        </if>
    </trim>
</insert>
```

这里if标签和trim标签搭配使用，trim标签是在SQL语句拼接中常用的标签，prefix指拼接的前缀，suffix指拼接的后缀，suffixOverrides指去除trim标签内SQL语句多余的后缀。一般是在if条件失效时，可能导致多余的逗号导致错误，所以使用suffixOverrides来去除这些逗号。

#### choose标签  

choose when otherwise 标签可以帮助实现if else的逻辑。一个choose标签至少有一个when标签，最多有一个otherwise标签  

###### 查询条件  

假设name具有唯一性，查询一本书  

当id有值时，使用id查询

当id没有值时，使用name查询  

###### 接口方法  

```java
/**
* 
*/
catalog selectByIdOrName(catalog catalog);
```

###### 对应的SQL  

```xml
<select id="selectByIdOrName" resultType="catalog">
    select * from t_catalog where 1=1
    <choose>
        <when test="id !=null and id !=''">
            and id = #{id}
        </when>
        <when test="name !=null and name !=''">
            and name = #{name}
        </when>
        <otherwise/>
    </choose>
</select>
```

只有id时或者id和name都有时，发送的SQL语句为

因为实现的逻辑时if else的逻辑  

```sql
select * from t_catalog where 1=1 and id = ?;
```

只有name时，发送的SQL语句为

```SQL
select * from t_catalog where 1 =1 and name = ?;
```

当id和name都不存在时，会走otherwise的逻辑，因为otherwise中没有实现任何逻辑，所以为空。因此发送的SQL语句为  

```sql
select * from t_catalog where 1 =1;
```

#### trim(set,where)  

这三个其实解决的是同样的问题。之前的例子中，在where条件中都会有where 1=1，但是并不希望这样的条件存在

##### 查询条件    

当只输入书籍的名称时，用这一个条件进行匹配  

当只输入用户ID的时候，用这一个条件进行匹配  

当书籍的名称和用户的ID都存在时，用这两个条件进行匹配  

不使用where1 =1这样的条件  

##### 动态SQL  

很显然需要解决这几个问题  

当条件都不满足时，SQL语句中不能有where，否则会出错  

当if条件满足时，SQL中需要where，且第一个成立的if条件中的and | or等需要去掉  

这种情况下可以使用where标签  

###### 接口方法  

```java
/**
* 根据输入的条件进行检索  
* 当只输入书籍的名称时，用这一个条件进行匹配  
* 当只输入用户ID的时候，用这一个条件进行匹配  
* 当书籍的名称和用户的ID都存在时，用这两个条件进行匹配
*/
List<catalog> selectByCatalogSelective(catalog catalog);
```

###### 对应的SQL  

```xml
<select id="selectByCatalogSelective"  resultType="catalog">
    select * from t_catalog
    <where>
        <if test="name != null and name !=''">
            and name like concat('%',#{name},'%')
        </if>
        <if test="userId!=null and userId!=''">
            and user_id = #{userId}
        </if>
    </where>
</select>
```

只有name字段的查询时，发送的语句是  

```sql
select * from t_catalog where name like concat ('%','java','%')
```

只有userId字段的查询时，发送的语句是  

```sql
select * from t_catalog where user_id = 1
```

同时有name字段和userId字段时，发送的语句是  

```sql
select * from t_catalog where name like concat('%','java','%') and user_id = 1
```

#### set  

set标签也类似，在前面更新条件时使用了set标签  

#### trim  

set和where其实都是trim标签的一种类型，这两种功能的都可以使用trim标签进行实现  

##### trim来表示where  

以上的where标签也可以写成  

```xml
<trim prefix="where" prefixOverrides="AND|OR">
</trim>
```

表示当trim中含有内容时，添加where，且第一个满足的条件中的and或者or会被去掉；如果trim中没有内容，则不添加where  





