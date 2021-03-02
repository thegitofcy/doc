### 1. MySQL

```shell
## $PWD 当前路径
docker run -d --name=mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -v $PWD/mysql-v/data:/var/lib/mysql -v $PWD/mysql-v/config/my.cnf:/etc/mysql/my.cnf mysql:5.7
```

