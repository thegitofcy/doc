# Maven

## 基于自己的项目创建脚手架

**以 Activities 为例:**

1. 源码中, 有一个 







## 出现坑

### 1. 执行 mvn 命令, 出现 Process terminated

本次执行的命令是 ==mvn clean==,  出现的原因是删除子模块后, POM 文件内的 module 没有删掉. 删掉后再次执行就 OK 了