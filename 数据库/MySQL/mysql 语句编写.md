## 1. MySQL 不存在则插入, 存在则更新或者忽略

前提: 需要有 ==**UNIQUE 索引**==或者 ==**PRIMARY KEY**==



添加 unique 索引: 

```SQL
alter table customer add unique (cus_name);
```



### 1. 不存在则插入, 存在则更新.

#### on duplicate key update

如果插入的数据 会导致  ==**UNIQUE 索引**==或者 ==**PRIMARY KEY**==冲突, 则执行 update 语句.

```SQL
INSERT INTO customer
    (id, cus_name, birthday, cus_sex, cus_addr, cus_iphone)
VALUES
       (1, 'tom1', '2019-07-11', '2', 'beijing1', '1111')
       # 加入一次多条数据, 有更新的有插入的, 则用逗号隔开
       , (2, 'jim2', '2019-07-11', '2', 'beijing1', '1111')
  ON DUPLICATE KEY
  #UPDATE `cus_iphone`=1111, cus_name='tom1';
  UPDATE `cus_iphone`=VALUES(cus_iphone), cus_name=VALUES(cus_name); #动态, 会被修改为对应上边的值
  
  id 为主键, 或者不用主键, 只用 unique
```



#### replace into

如果插入的数据 会导致  ==**UNIQUE 索引**==或者 ==**PRIMARY KEY**==冲突, 则先删除旧数据再插入最新的数据.

```sql
replace into customer
    (id, cus_name, birthday, cus_sex, cus_addr, cus_iphone)
values
      (1, 'tom2', '2019-07-21', '4', 'beijing12', '21111');
      
id 为主键
```



### 2. 避免重复插入

如果插入的数据 会导致  ==**UNIQUE 索引**==或者 ==**PRIMARY KEY**==冲突, 则忽略本次操作/不插入数据

```sql
insert ignore into customer
    (id, cus_name, birthday, cus_sex, cus_addr, cus_iphone)
values
      (1, 'tom22', '2019-09-21', '9', 'beijing1222', '431111');
      
  id 为主键
```



# 2. 关键字--操作字符串

## locate(str, subStr)

`locate(subStr, str)` : 子字符串 subStr 第一次出现在字符串 str 中的位置.(结果从 1 开始)

`locate(subStr, str, post)` : 从 post 开始, 子字符串 subStr 第一次出现在字符串 str 中的位置.(结果从 1 开始)

```sql
-- result: 3
select locate('3', '1234');  -- 3 出现在 1234 中的位置.

-- result: 3
select locate('3','1234', 2); -- 从位置 2 开始查找, 3 出现在字符串中 1234 中的位置.
```



## substring_index(s, delimiter, number)

delimiter: 分隔符

`substring_index(str, delimiter, number)`: 返回字符串 str 中第 number 个出现的分隔符 delimiter 之后的子字符串

- number 是正数: 返回(从左向右数)第 number 个 delimiter 左侧的字符串.
- number 是负数: 返回(从右向左数)第 number 个 delimiter 右侧的字符串.

```sql
-- 结果: name1
select substring_index('name1,name2,name3', ',', 1) from aaa; 

-- 结果: name3
select substring_index('name1,name2,name3', ',', -1) from aaa;
```



## left(arg, length) 和 right(arg, length)

获取 arg 左边或者右边的 lenght 个字符串.   arg 可以是数字也可以是字符串.

```sql
-- 结果: 12
select left(1234, 2);

-- 结果: 34
select right(1234, 2);
```



## 字符串拼接

```sql
create table aaa
(
    id   int         null,
    name varchar(21) null,
    age int null
);
```



### 1. concat(str1, str2...) 多个字符串拼接成一个字符串

将==**多个**==字符串拼接这一个新的字符串. 如果任何一个参数为 null, 则返回结果就为 null

```sql
-- 结果: 123456
select concat('123', '456')

-- 结果: null
select concat('123', '456', null)
```



### 2. concat_ws(separator, str1, str2...) 多个字符串拼接成一个字符串, 使用separator分割

separator 不能为 null, 如果 separator 为 null, 则结果为 null

```SQL
-- 结果: 123,456,789
select concat_ws(',','123', '456', '789');

-- 结果: null
select concat_ws(null, '123', '456', '789');
```





### 3. group_concat(column [order by column] [separator '分隔符']) 多行转一行

在使用 group by 分组的时候, 既能显示要分组的字段, 又能显示分组后,在同一组的字段

```sql
-- 结果: 除了显示分组字段name外, 还会默认以 , 为分隔符, 显示同一分组的 age 字段
select name, group_concat(age) from aaa group by name;

-- 结果: 除了显示分组字段name外, 还会以 _ 为分隔符, 显示同一分组的 age 字段, 并且按照 age 进行排序.
select name,group_concat(age order by age asc separator '_') from aaa group by name;
```



### 4. mysql.help_topic 一行转多行



```sql
-- 首先模拟数据   结果为 a,b,c,d
select length(name) from aaa where id = 6;

-- 结果为 4 条数据, id 都为 6, name1 分别为 a, b, c, d
select 
id, 
substring_index(substring_index(a.name, ',', topic.help_topic_id + 1), ',', -1) as name1 
from aaa a
join mysql.help_topic topic on topic.help_topic_id < (length(a.name) - length(replace(a.name, ',', '')) + 1)
where id = 6;
```



### 5. ifnull(arg1, arg2) :判断arg1 是否为 null, 如果为 null, 则返回 arg2, 否则返回 arg1

```sql
-- 结果: 123
select ifnull(null, '123');

-- 结果: abc
select ifnull('abc', '123');
```



# 3. mysql.help_topic