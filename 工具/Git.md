# 设置

## 1. git status 中文文件名编码问题

如果显示的中文文件名和路径为 八进制 的字符编码, 则可以通过设置 `core.quotepath` 为 `false` 来正常的显示中文.

设置方法: `git config --global core.quotepath false`



## 2. Git 忽略文件

### 1. 编写 `.gitignore` 文件. 然后在文件中添加要忽略的文件或者目录. 

==**注意: 忽略文件路径的添加, 都是相对于 .gitignore 文件的**==



### 2. 全局忽略文件

Mac 环境:

1. 编写 `~/.gitignore_global` 全局忽略文件

2. 编辑`~/.gitconfig`, 引用全局忽略文件

   ```
   [user]
           name = cy
           email = theemailofcy@163.com
   [core]
           quotepath = false
           excludesfile = .gitignore_global	// 在这一句引用
   ```

   



# 操作

## 1. 远程版本和本地版本冲突

原理: 通过 `git stash [save "备注"]` 命令==**暂存**==本地修改, 然后更新代码, 根据==**暂存标记(类似 stash@{0})**==还原暂存内存, 在本地结局冲突, 最后提交.

```shell
git stash save "本地修改"  // 保存本地修改
git stash list           // 将 Git stash 信息列表打印出来.  备注信息前边的是刚刚保存的标记. 
git pull								 // 更新内容
git stash pop [暂存标记]  // 根据暂存标记恢复暂存的内容.
本地结局冲突......
提交本地代码
git stash drop [暂存标记] // 删除暂存内容
git stash clear         // 和上边一样,也是删除暂存内容,但是这里是删除所有stash 内容
```



# 附录

## 常见问题

### 1. 当使用 idea 工具时, 提交时无法 add

1. 可能是当前项目的 ==.gitginore== 文件配置了忽略文件.

2. 可能是 Git 的全局 ==.gitginore_global== 文件配置了忽略文件.

   

解决方案: 根据情况修改上述两个文件即可.