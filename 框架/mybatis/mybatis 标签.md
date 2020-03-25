### if









### choose

提供类似 ==switch== 的功能. 如果传入参数 1, 就按参数 1 查找, 如果传入参数 2, 就按参数 2 查找, 如果都没有传入, 则按默认的查找

```xml
<choose>
  <when test="null != param1">
    and 
  </when>
  <when test="">
  </when>
  <otherwize>
    
  </otherwize>
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

