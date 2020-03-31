### if

```xml
<if test="null != id">
	and id = #{id}
</if>
```







### choose, when, otherwise

提供类似 ==switch== 的功能. 如果传入参数 1, 就按参数 1 查找, 如果传入参数 2, 就按参数 2 查找, 如果都没有传入, 则按默认的查找

```xml
<choose>
  <when test="null != param1">
    and param1 = #{param1}
  </when>
  <when test="null != param2">
    and param2 = #{param2}
  </when>
  <otherwise>
    and 
  </otherwise>
</choose>
```





### foreach

遍历.

场景: 批量插入时.

属性:

```properties
collection: 假如参数是 list, 那 collection 必须是 list.
item: item 表示集合中的每个元素.
separator: foreach 生成的多条记录之间的分隔符.
```

example:

```xml
<insert id="insertUsers" keyProperty="id" useGeneratedKeys="true" parameterType="java.util.List">
    insert into user (username, password)
    values
    <foreach collection="list" item="info" separator=",">
        (#{info.username}, #{info.password})
    </foreach>
    ;
</insert>
```



### set

在进行 update 的时候, 如果没有使用 if 标签进行判断, 那么当一个值为 null 的时候, 就会报错.

如果使用了 if 标签进行判断, 如果前边的 if 判断没有通过, 就会导致逗号多余错误.

那么此时使用 set 标签就可以动态的配置 set 关键字, 并剔除追加到条件末尾的任何不相关的逗号.

使用 set + if 后, 如果某项为 null, 则不进行更新, 保留原值.

```xml
update user
<set>
	<if test="username != null and username != ''">
    username = #{username},
  </if>
  <if test="password != null and password != ''">
    password = #{password},
  </if>
</set>
where id = #{id
```

