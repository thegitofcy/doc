# Mybatis 不可描述的秘密

## 1. 参数处理

传递给 XXXMapper 接口放的参数最终都会放到一个 ==**map**== 里边. 假如传递给 XXXMapper 接口的是名为 properties 的参数, 类型如下:

1. List 类型: 那么这个 ==**map**== 里就会有一条 =={list : properties}==的数据, ==key 就是当前传入参数的类型, Value 为传入参数的值==
2. 数组类型 : 那么这个 ==**map**== 里就会有一条 =={array : properties}==的数据, ==key 就是当前传入参数的类型, Value 为传入参数的值==

3. map 类型: 那么此时不做任何转换.==**map**== 中就会有传入的参数的值. =={key1:value1, key2:value2...}==
4. Bean 对象: 那么 ==**map**== 中就是以 bean 的属性名为 key,属性值为 Value, =={username:tom, age:12}==
5. 基本类型: 那么 ==**map**==中 key 是 ==变量名==, Value 值是 ==变量值==.



涉及到的秘密:

使用 ==**foreach**== 的时候, ==**foreach**==的属性 ==**collection**==的取值问题:

- 假如传入的参数是个 list, 那么 ==**collection**==的值就是 ==list==, 相当于取出 ==**map**== 中 key 为 ==list==的 Value 值. 然后 foreach 的 ==item== 属性取值就是 list 内的元素
- 假如传入的参数是个数组, 同理, ==**collection**==的值就是 ==array



## 2. 实体类属性类型

在 java 中,==**基本类型是有默认值的**==, 比如实体类 User 中有属性 int age. 当创建 User 的时候, User 的 age 属性会有默认值 0,当使用 age 属性的时候, 它总是会有值

```xml
<if test="null != age">
	and age = #{age}
</if>
```

因此在XML 中, 想通过以上方式, 判断的结果就总是为 true. 那么导致一些问题.

结论: ==**实体类的属性不适用基本类型, 使用包装类型**==.

