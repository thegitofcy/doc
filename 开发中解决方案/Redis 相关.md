# Redis 和 MySQL 数据不一致

理论上来说, `给缓存设置过期时间, 是保证数据最终一致性`的解决方案. 也就是说, 对缓存数据设置过期时间, 所有的`写操作`都以数据库为准, 对缓存操作最最大努力即可, 也就是说如果数据库写入成功, 缓存更新失败, 那么只要达到过期时间, 后边的读操作自然会从数据库读取新值, 然后回填到缓存中.

## 读操作

```shell
if (null == read redis) {
	read db
}
```



## 写操作

==**有如下几种方案:**==

```shell
update_db then update_redis (导致 DB 和 Redis 数据不一致)
update_redis then update_db (导致 DB 和 Redis 数据不一致)
update_db then rm_redis
rm_redis then update_db
rm_redis then update_db then sleep xx ms then rm_redis
```



### 1. update_db then update_redis

先更新db, 再更新 Redis, 会导db和 Redis致数据不一致

```sql
ThreadA update db; 	//a
ThreadB update db;	//b, DB 最终值为 b
ThreadB update redis;	//b
ThreadA update redis;	//a,	Redis 最终值为 a
```



### 2. update_redis then update_db

先更新 Redis, 再更新 db, 会导致 Redis 和 DB 数据并不一致

```SQL
ThreadA update redis;	// a
ThreadB update redis;	// b, Redis最终值为 b
ThreadB update db;	// b
THreadA update db;	// a, DB 最终值为 a
```



### 3.rm_redis then update db

先删除 Redis, 再更新 DB, 也会导致数据不一致

```SQL
ThreadA rm redis;
if (ThreadB read redis == null) then ThreadB read old DB;	// 线程 B 获取到旧值 b
ThreadB update DB;	// b
ThreadA update DB;	// a
ThreadA rm redis;	// 延时删除
```

- 1~3 操作后, DB 的值依然为旧值
- 4~5 假如前边添加一个 ThreadC read old DB, C 查询获取到旧值. 然后添加一个第 6 步: C 将旧值插入缓存, 最终依然会造成 Redis 和 DB 数据不一致.
- 第 5 步的延时时间越长, 越能规避上边的问题.