# MySQL 不存在则插入, 存在则更新或者忽略

前提: 需要有 ==**UNIQUE 索引**==或者 ==**PRIMARY KEY**==



添加 unique 索引: 

```SQL
alter table customer add unique (cus_name);
```



## 1. 不存在则插入, 存在则更新.

### on duplicate key update

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



### replace into

如果插入的数据 会导致  ==**UNIQUE 索引**==或者 ==**PRIMARY KEY**==冲突, 则先删除旧数据再插入最新的数据.

```sql
replace into customer
    (id, cus_name, birthday, cus_sex, cus_addr, cus_iphone)
values
      (1, 'tom2', '2019-07-21', '4', 'beijing12', '21111');
      
id 为主键
```



## 2. 避免重复插入

如果插入的数据 会导致  ==**UNIQUE 索引**==或者 ==**PRIMARY KEY**==冲突, 则忽略本次操作/不插入数据

```sql
insert ignore into customer
    (id, cus_name, birthday, cus_sex, cus_addr, cus_iphone)
values
      (1, 'tom22', '2019-09-21', '9', 'beijing1222', '431111');
      
  id 为主键
```

