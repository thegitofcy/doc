```
#Mac使用ll命令
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
alias apps='cd /Applications'

#set java environment
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_201.jdk/Contents/Home
export PATH=$PATH:$JAVA_HOME/bin

#配置Tomcat
export PATH=$PATH:/usr/local/tomcat/bin

#配置ES (没什么用,因为集群的话还是要一个一个启动)
#export PATH=$PATH:/usr/local/elasticsearch-7.4.2/bin

#配置maven
export MAVEN_HOME=/usr/local/maven
export PATH=$PATH:$MAVEN_HOME/bin

#配置MySQL
PATH=$PATH:/usr/local/mysql/bin

#升级系统自带 GIT,brew 安装 Git,然后配置环境变量 GIT 指向自己安装的版本
export GIT=/usr/local/Cellar/git/2.24.0_1
export PATH=$GIT/bin:$PATH

#升级系统自带 SVN,安装后,添加以下环境变量
export PATH=/opt/subversion/bin:$PATH

#配置 btrace
export PATH=/usr/local/btrace-bin-1.3.11.3/bin:$PATH

#配置 gradle
export GRADLE_HOME=/usr/local/gradle
export PATH=$PATH:$GRADLE_HOME/bin

#配置 Groovy
export GROOVY_HOME=/usr/local/groovy
export PATH=$PATH:$GROOVY_HOME/lib

#配置 ant
export ANT_HOME=/usr/local/ant
export PATH=$PATH:$ANT_HOME/bin

#配置 zookeeper
export ZOOKEEPER_HOME=/usr/local/zookeeper
export PATH=$PATH:$ZOOKEEPER_HOME/bin

#配置 kafka
export ZOOKEEPER_HOME=/usr/local/kafka
export PATH=$PATH:$ZOOKEEPER_HOME/bin
```



