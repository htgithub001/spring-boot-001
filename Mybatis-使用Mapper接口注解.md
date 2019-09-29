# mybatis笔记之使用Mapper接口注解

来源：https://www.cnblogs.com/cc11001100/p/7811257.html

## 1. mybatis支持的映射方式

mybatis支持的映射方式有基于xml的mapper.xml文件、基于java的使用Mapper接口class，简单学习一下mybatis使用接口来配置映射的方法。

接口方法注解主要是四个：@Insert、@Delete、@Update、@Select

## 2. 如何使用接口注解来映射

下面的实验都是基于t_user表的，其结构如下：

```
DROP TABLE IF EXISTS t_user;
CREATE TABLE t_user (
    id BIGINT AUTO_INCREMENT PRIMARY KEY ,
    username VARCHAR(100) NOT NULL ,
    passwd CHAR(32) NOT NULL,
    birth_day DATETIME
) CHARSET UTF8;
```

### 2.1 增 @Insert

插入记录的时候麻烦的一点是主键如何生成，对此基本上有三种方案，分别是手动指定(应用层)、自增主键(数据层单表)、选择主键(数据层多表)。

1. 在应用层手动指定主键

手动指定的方式不把主键区别看待，插入之前在应用层生成对象的时候就会给主键一个值，插入的时候与普通字段没啥区别。

```
/**
 * 插入记录，手动分配主键
 *
 * @param user
 * @return
 */
@Insert("INSERT INTO t_user (id, username, passwd) VALUES (#{id}, #{username}, #{passwd})")
int addUserAssignKey(User user);
```

在上面的这个例子中，mybatis并不知道到底哪个字段是主键，id虽然是主键字段，但并没有被区别对待。

注意#{username}这种写法，是把User作为了当前上下文，这样访问User的属性的时候直接写属性名字就可以了。

2. 表自增主键
自增主键对应着XML配置中的主键回填，一个简单的例子：

```
/**
 * 插入记录，数据库生成主键
 *
 * @param user
 * @return
 */
@Options(useGeneratedKeys = true, keyProperty = "id")
@Insert("INSERT INTO t_user (username, passwd) VALUES (#{username}, #{passwd})")
int addUserGeneratedKey(User user);
```

使用Option来对应着XML设置的select标签的属性，userGeneratordKeys表示要使用自增主键，keyProperty用来指定主键字段的字段名。

自增主键会使用数据库底层的自增特性。

3. 选择主键
选择主键从数据层生成一个值，并用这个值作为主键的值。

```
@Insert("INSERT INTO t_user (username, passwd) VALUES (#{username}, #{passwd})")


@SelectKey(statement = "SELECT UNIX_TIMESTAMP(NOW())", keyColumn = "id", keyProperty = "id", resultType = Long.class, before = true)
int addUserSelectKey(User user);
```

### 2.2 删 @Delete

删除的时候只要把语句条件神马的写在@Delete注解的value里就好了，返回一个int类型是被成功删除的记录数。
```
/**
 * 删除记录
 *
 * @param id
 * @return
 */
@Delete("DELETE FROM t_user WHERE id=#{id}")
int delete(Long id);
```

### 2.3 改 @Update

修改的时候和删除一样只要把SQL语句写在@Update的value中就好了，返回一个int类型表示被修改的记录行数。

```
/**
 * 修改记录
 *
 * @param user
 * @return
 */
@Update("UPDATE t_user SET username=#{username}, passwd=#{passwd} WHERE id=#{id}")
int update(User user);
```

### 2.4 查 @Select

查询的时候稍稍有些复杂，因为查询会涉及到如何将查出来的字段设置到对象上，通常有那么三种办法：

#### 1. 在SQL语句中手动指定别名来匹配
在写SQL语句的时候，手动为每一个字段指定一个别名来跟对象的属性做匹配，适用于表字段名与对象属性名差异很大没有规律并且表字段不多的情况。

```
/**
 * 根据ID查询，手动设置别名
 *
 * @param id
 * @return
 */
@Select("SELECT id, username, passwd, birth_day AS birthDay FROM t_user WHERE id=#{id}")
User loadByIdHandAlias(Long id);
```

#### 2. 使用mybatis的自动下划线驼峰转换
mybatis有一个选项叫mapUnderscoreToCamelCase，当表中的字段名与对象的属性名相同只是下划线和驼峰写法的差异时适用。

配置了mapUnderscoreToCamelCase之后mybatis在将ResultSet查出的数据设置到对象的时候会尝试先将下划线转换为驼峰然后前面拼接set去设置属性。

开启转换：

然后查询：
```
/**
 *  根据ID查询，开了自动驼峰转换
 *
 * @param id
 * @return
 */
@Select("SELECT * FROM t_user WHERE id=#{id}")
User loadByIdAutoAlias(Long id);
```

#### 3. 使用ResultMap
对于表的字段名和对象的属性名没有太大相同点并且表中的字段挺多的情况下，应该使用ResultMap做适配。
```
/**
 * 使用ResultMap
 *
 * @param id
 * @return
 */
@Results(id = "userMap", value = {
        @Result(id=true, column = "id", property = "id"),
        @Result(column = "username", property = "username"),
        @Result(column = "passwd", property = "passwd"),
        @Result(column = "birth_day", property = "birthDay")
})
@Select("SELECT * FROM t_user WHERE id=#{id}")
User loadByIdResultMap(Long id);
```

@Results对应着XML中的ResultMap，同时可以为其指定一个id，其它地方可以使用这个id来引用它，比如要引用上面的这个Results：

```
/**
 * 引用其他的Result
 *
 * @param id
 * @return
 */
@ResultMap("userMap")
@Select("SELECT * FROM t_user WHERE id=#{id}")
User loadByIdResultMapReference(Long id);
```

使用@ResultMap来引用一个已经存在的ResultMap，这个ResultMap可以是在Java中使用@Results注解定义的，也可以是在XML中使用resultMap标签定义的。
