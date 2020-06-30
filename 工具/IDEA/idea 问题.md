# 1. IDEA启动项目无法自动编译的问题

IDEA中的==**.iml**==文件是项目标识文件，缺少了这个文件，IDEA就无法识别项目。

缺少==**.iml**==文件,run项目无法自动compile生成target文件

生成iml文件:

进入项目所在目录,执行`mvn idea:module`命令

