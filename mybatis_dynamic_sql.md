# Mybatis 동적 SQL


#### if 구문

```xml
<select id="findActiveBlogLike" resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’ 
  <if test="title != null">
    AND title like #{title}
  </if>
  <if test="author != null and author.name != null">
    AND author_name like #{author.name}
  </if>
</select>
```

#### choose, when, otherwise 구문

```xml
<select id="findActiveBlogLike" resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select>
```

#### trim, where, set 구문

  * where
```xml
<select id="findActiveBlogLike" 
     resultType="Blog">
  SELECT * FROM BLOG 
  <where> 
    <if test="state != null">
         state = #{state}
    </if> 
    <if test="title != null">
        AND title like #{title}
    </if>
    <if test="author != null and author.name != null">
        AND author_name like #{author.name}
    </if>
  </where>
</select>
```

  * trim
     
```xml
<trim prefix="WHERE" prefixOverrides="AND |OR ">
  ... 
</trim>
```

  * set
  
  update하고자 하는 칼럼을 동적으로 포함 
```xml
<update id="updateAuthorIfNecessary">
  update Author
    <set>
      <if test="username != null">username=#{username},</if>
      <if test="password != null">password=#{password},</if>
      <if test="email != null">email=#{email},</if>
      <if test="bio != null">bio=#{bio}</if>
    </set>
  where id=#{id}
</update>
```

     set element는 동적으로 'SET' 키워드를 붙이고, 필요 없는 ','는 제거.

```xml
<trim prefix="SET" suffixOverrides=",">
  ...
</trim>
```



#### foreach

  collection의 내용으로 반복 처리.
```xml
<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT *
  FROM POST P
  WHERE ID in
  <foreach item="item" index="index" collection="list"
      open="(" separator="," close=")">
        #{item}
  </foreach>
</select>
```

#### bind

  변수를 반든 뒤, context에 바인딩.
```xml
<select id="selectBlogsLike" resultType="Blog">
  <bind name="pattern" value="'%' + _parameter.getTitle() + '%'" />
  SELECT * FROM BLOG
  WHERE title LIKE #{pattern}
</select>
```

### Multi DB vendor 지원

  "_databaseid" 변수로 설정된 databaseidprovider라면 동적인 코드에도 사용 가능
```xml
<insert id="insert">
  <selectKey keyProperty="id" resultType="int" order="BEFORE">
    <if test="_databaseId == 'oracle'">
      select seq_users.nextval from dual
    </if>
    <if test="_databaseId == 'db2'">
      select nextval for seq_users from sysibm.sysdummy1"
    </if>
  </selectKey>
  insert into users values (#{id}, #{name})
</insert>
```

### Pluggable Scripting Languages For Dynamic SQL

  mybatis 3.2 부터 플러그인 형태의 스크립트 언어 사용 가능.

  * xml : 기본적으로 xml로 쿼리 설정. 대부분의 동적 태크를 사용할 수 있다.
  
  * raw : Annoration으로 지정한 parameter를 이용해서 구문 전달 가능. xml 보다 속도는 빠르지만, 사용 가능한 기능이 부족하다.

  xml
```xml
<select id="selectBlog" lang="raw">
  SELECT * FROM BLOG
</select>
```

  raw (Lang 속성을 추가해서 사용할 언어 명시 가능)
  
```java
public interface Mapper {
  @Lang(RawLanguageDriver.class)
  @Select("SELECT * FROM BLOG")
  List<Blog> selectBlog();
}
```

  직접 Lang Driver를 구현할 수도 있음
  
```
public interface LanguageDriver {
  ParameterHandler createParameterHandler(MappedStatement mappedStatement, Object parameterObject, BoundSql boundSql);
  SqlSource createSqlSource(Configuration configuration, XNode script, Class<?> parameterType);
  SqlSource createSqlSource(Configuration configuration, String script, Class<?> parameterType);
}
```
