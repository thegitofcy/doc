# log4j
# 1. Log4j 简介
log4j 有三个比较重要的组件:
- ==Loggers(记录器)== : 日志类别和级别;
- ==Appenders(输出源)==: 日志要输出的地方;
- ==Layouts(布局)==: 日志以何种形式输出;

## 1. Log4j 配置

### 1 核心组件

- ==***Loggers***== : 记录器. 指定日志级别以及 appenderName;
- ==***Appender***== : 指定日志的输出源;
- ==***Layouts***==: 指定日志信息输出的样式;



#### 1.1 Loggers 记录器

==Loggers== 组件在此系统中被分为五个级别: ==DEBUG==, ==INFO==, ==WARN==, ==ERROR==, ==FATAL==.
这五个级别的优先级为: ==DEBUG < INFO < WARN < ERROR < FATAL==, 分别来指定这条日志信息的重要程度.
log4j有一个规则: ==只输出级别不低于设置级别的日志信息==. 假设设置为 INFO 级别, 那么只会输出 INFO, WARN, ERROR, FATAL 这几个级别的日志信息, 而比 INFO 级别低的 DEBUG 级别的则不会输出.



#### 1.2 Appenders 输出源

**Log4j 允许把日志输出到不同的的位置**, 例如控制台, 文件等, 也可以根据天数或者文件大小来产生新文件.

**常用类如下** 

- ==org.apache.log4j.ConsoleAppender== 输出到控制台
- ==org.apache.log4j.FileAppender== 输出到文件
- ==org.apache.log4j.DailyRollingFileAppender== 输出到文件, 并且在指定的周期后生成新的日志文件.
- ==org.apache.log4j.RollingFileAppender== 输出到文件, 文件打到指定的大小后, 产生一个新的日志文件.同时可以指定生成的新日志文件的最大数.
- ==org.apache.log4j.WriterAppender== 将日志信息以流格式输出到任意指定的地方.



#### 1.3 Layouts

Log4j 可以在 Appenders 的后边添加 Layouts 来完成这个功能.
Layouts 提供四种日志输出样式: ==HTML样式==, ==自由指定样式==, ==包含日志级别与信息的样式==, ==包含日志事件, 线程, 类别等信息的样式==

**常用类如下 **:

- ==org.apache.log4j.HTMLLayout== : 以 HTML 表格形式布局
- ==org.apache.log4j.PatternLayout== : 可以灵活的指定布局模式
- ==org.apache.log4j.SimpleLayout== : 包含日志信息的级别和信息字符串
- ==org.apache.log4j.TTCCLayout== : 包含日志产生的事件, 线程, 类别等信息 



### 2 配置详解

**配置 Log4j 实际上就是对 Loggers, Appender, Layout 进行相应的设定.**
Log4j 支持两种配置文件格式: ==XML==, ==properties==

#### 2.1 配置 Logger (指定日志级别, 日志信息输出的目的地)
```properties
log4j.rootLogger = [level], appenderName1, appenderName2,…
# 例如:
# console 和 dailyFile : 为自定义的字符串, 用于在后边设置日志信息输出的目的地, 类似一个分组的作用.
log4j.rootLogger = INFO, console, dailyFile…

log4j.additivity.org.apache=false：表示Logger不会在父Logger的appender里输出，默认为true。
```
- ==level== : 设定日志记录的最低级别, 除了可设置为Logger 支持的 5 种类别外, 还可指定为自定义级别.
- ==appenderName== : 指定日志信息要输出到哪里. 可以同时指定多个输出目的地, 用逗号隔开.

#### 2.2 配置 Appender (日志信息输出目的地)
```properties
log4j.appender.[appenderName] = className

# 例如:
# 表示 log4j.rootLogger 中指定的日志信息目的地为 console 的分组要输出到控制台
log4j.appender.console = org.apache.log4j.ConsoleAppender
# 表示 log4j.rootLogger 中指定的日志信息目的地为 dailyFile 的分组要输出文件中,并且每天产生一个信息的日志文件
log4j.appender.dailyFile = org.apache.log4j.DailyRollingFileAppender
```
- ==appenderName== : 自定义的 appenderName, 就是在在 log4j.rootLogger 中指定的 appenderName1, appenderName2 这些.
- ==className== 指定日志信息输出目的地, 可设值如下:
	- ==org.apache.log4j.ConsoleAppender== 输出到控制台
	- ==org.apache.log4j.FileAppender== 输出到文件
	- ==org.apache.log4j.DailyRollingFileAppender== 输出到文件, 并且在指定的周期后生成新的日志文件.
	- ==org.apache.log4j.RollingFileAppender== 输出到文件, 文件打到指定的大小后, 产生一个新的日志文件.同时可以指定生成的新日志文件的最大数.
	- ==org.apache.log4j.WriterAppender== 将日志信息以流格式输出到任意指定的地方.

##### 2.2.1 org.apache.log4j.==ConsoleAppender== 输出到控制台的日志信息的配置
- ==***Threshold=WARN***== : 表示名称为 appenderName 的分组中, 日志信息输出的最低输出级别. 默认为DEBUG
- ==***ImmediateFlush=true***== : 表示所有日志信息都会被立即输出, 设置为 false 则不输出. 默认为true
- ==***Target=System.err***== : 默认值是System.out

##### 2.2.2 org.apache.log4j.FileAppender 输出到文件的日志信息的配置
- Threshold=WARN 表示名称为 appenderName 的分组中, 日志信息输出的最低输出级别. 默认为DEBUG
- ImmediateFlush=true 表示所有日志信息都会被立即输出, 设置为 false 则不输出. 默认为true
- Append=false true 表示日志信息增加到指定的文件中. false 表示日志信息会覆盖指定的文件. 默认为true
- File=D:/logs/logging.log4j 指定日志信息输出到的文件.这里表示输出到路径为D:/logs/logging.log4j的文件中.

##### 2.2.3 org.apache.log4j.DailyRollingFileAppender 输出到文件中, 并且指定生成新日志文件的周期.
- Threshold=WARN 表示名称为 appenderName 的分组中, 日志信息输出的最低输出级别. 默认为DEBUG
- ImmediateFlush=true 表示所有日志信息都会被立即输出, 设置为 false 则不输出. 默认为true
- Append=false true 表示日志信息增加到指定的文件中. false 表示日志信息会覆盖指定的文件. 默认为true
- File=D:/logs/logging.log4j 指定日志信息输出到的文件.这里表示输出到路径为D:/logs/logging.log4j的文件中.
- DatePattern=“.yyyy-MM” 指定新日志文件生成的周期, “.yyyy-MM”表示每月, DatePattern 可选值如下:
	- ”.yyyy-MM” 每月
	- ”.yyyy-ww” 每周
	- ”.yyyy-MM-dd” 每天
	- ”.yyyy-MM-dd-a” 每天两次
	- ”.yyyy-MM-dd-HH” 每小时
	- ”.yyyy-MM-dd-HH-mm” 每分钟.

##### 2.2.4 org.apache.log4j.RollingFileAppender 输出到文件中, 并且指定当当前日志文件达到多大时生成新的日志文件, 和生成呈日志文件的最大数
- Threshold=WARN 表示名称为 appenderName 的分组中, 日志信息输出的最低输出级别. 默认为DEBUG
- ImmediateFlush=true 表示所有日志信息都会被立即输出, 设置为 false 则不输出. 默认为true
- Append=false true 表示日志信息增加到指定的文件中. false 表示日志信息会覆盖指定的文件. 默认为true
- File=D:/logs/logging.log4j 指定日志信息输出到的文件.这里表示输出到路径为D:/logs/logging.log4j的文件中.
- MaxFileSize=100KB 指定生成新日志文件的阈值. 当达到指定大小后, 生成一个新的日志文件, 后缀可以是KB, MB, GB
- MaxBackupIndex=2 指定可以产生的新日志文件的最大数.

#### 2.3 配置 LayOut (日志信息输出的格式)
```properties
# console 为 log4j.rootLogger 指定的appenderName, 也就是分组, layout 是说此配置项是配置 console 分组的日志输出格式
log4j.appender.console.layout = className

# 例如:
# org.apache.log4j.TTCCLayout 表示 console 下的日志输出格式为包含日志产生的事件, 线程, 类别等信息
log4j.appender.console.layout = org.apache.log4j.TTCCLayout
```
- className : 表示当前分组下的日志输出格式, 可选值如下:
	- org.apache.log4j.HTMLLayout : 以 HTML 表格形式布局
	- org.apache.log4j.PatternLayout : 可以灵活的指定布局模式
	- org.apache.log4j.SimpleLayout : 包含日志信息的级别和信息字符串
	- org.apache.log4j.TTCCLayout : 包含日志产生的事件, 线程, 类别等信息 

##### 2.3.1 log4j.appender.console.layout = org.apache.log4j.HTMLLayout 以 HTML 表格形式输出的日志的配置
- LocationInfo=true 输出 Java 文件名和行号, 默认为false
- Title=My Logging 默认值是Log4J Log Messages.

##### 2.3.2 log4j.appender.console.layout = org.apache.log4j.PatternLayout 灵活的指定布局模式的配置
- ConversionPattern=%m%n 设置以怎样的格式显示消息. 可选值如下:
	- %p：输出日志信息的优先级，即DEBUG，INFO，WARN，ERROR，FATAL。
	- %d：输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，如：%d{yyyy/MM/dd HH:mm:ss,SSS}。
	- %r：输出自应用程序启动到输出该log信息耗费的毫秒数。
	- %t：输出产生该日志事件的线程名。
	- %l：输出日志事件的发生位置，相当于%c.%M(%F:%L)的组合，包括类全名、方法、文件名以及在代码中的行数。例如：test.TestLog4j.main(TestLog4j.java:10)。
	- %c：输出日志信息所属的类目，通常就是所在类的全名。
	- %M：输出产生日志信息的方法名。
	- %F：输出日志消息产生时所在的文件名称。
	- %L:：输出代码中的行号。
	- %m:：输出代码中指定的具体日志信息。
	- %n：输出一个回车换行符，Windows平台为”\r\n”，Unix平台为”\n”。
	- %x：输出和当前线程相关联的NDC(嵌套诊断环境)，尤其用到像java servlets这样的多客户多线程的应用中。
	- %%：输出一个”%”字符。

另外，还可以在%与格式字符之间加上修饰符来**控制其最小长度、最大长度、和文本的对齐方式**。如：
	1. c：指定输出category的名称，最小的长度是20，如果category的名称长度小于20的话，默认的情况下右对齐。
		* 2)%-20c：”-“号表示左对齐。
		* 3)%.30c：指定输出category的名称，最大的长度是30，如果category的名称长度大于30的话，就会将左边多出的字符截掉，但小于30的话也不会补空格。

### 3. 其他高级功能
### 3.1 不同类的日志输出到不同的日志文件
log4j的强大功能无可置疑，但实际应用中免不了遇到某个功能需要输出独立的日志文件的情况, 也就是不同的类的日志, 要输出到不同的文件，怎样才能把所需的内容从原有日志中分离，形成单独的日志文件呢？其实只要在现有的log4j基础上稍加配置即可轻松实现这一功能。

首先一个常见的配置, 它在控制台和myweb.log文件中记录日志:
```properties
log4j.rootLogger=DEBUG, stdout, logfile
 
log4j.category.org.springframework=ERROR
log4j.category.org.apache=INFO
 
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n
 
log4j.appender.logfile=org.apache.log4j.RollingFileAppender
log4j.appender.logfile.File=${myweb.root}/WEB-INF/log/myweb.log
log4j.appender.logfile.MaxFileSize=512KB
log4j.appender.logfile.MaxBackupIndex=5
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout
log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n
```

现在实现不同的类的日志输出到不同的日志文件
```properties
# 首先在类中获取 logger
private static Log logger = LogFactory.getLog(Test.class);

# 然后在log4j.properties中加入:
log4j.logger.cn.com.Test= DEBUG, test
log4j.appender.test=org.apache.log4j.FileAppender
log4j.appender.test.File=${myweb.root}/WEB-INF/log/test.log
log4j.appender.test.layout=org.apache.log4j.PatternLayout
log4j.appender.test.layout.ConversionPattern=%d %p [%c] - %m%n
# 也就是让cn.com.Test中的logger使用log4j.appender.test所做的配置。
```

### 3.2 同一个类的日志输出到多个日志文件
同一个类的日志要输出到多个日志文件, 如下:
```properties
# 首先在类中获取 logger,需要输出到几个日志文件就获取几个 logger,然后针对每一个 logger 进行配置
private static Log logger1 = LogFactory.getLog(“myTest1”);
private static Log logger2 = LogFactory.getLog(“myTest2”);

# 针对每一个 logger 进行配置
log4j.logger.myTest1= DEBUG, test1
log4j.appender.test1=org.apache.log4j.FileAppender
log4j.appender.test1.File=${myweb.root}/WEB-INF/log/test1.log
log4j.appender.test1.layout=org.apache.log4j.PatternLayout
log4j.appender.test1.layout.ConversionPattern=%d %p [%c] - %m%n
　　
log4j.logger.myTest2= DEBUG, test2
log4j.appender.test2=org.apache.log4j.FileAppender
log4j.appender.test2.File=${myweb.root}/WEB-INF/log/test2.log
log4j.appender.test2.layout=org.apache.log4j.PatternLayout
log4j.appender.test2.layout.ConversionPattern=%d %p [%c] - %m%n
```

也就是在用logger时给它一个自定义的名字(如这里的”myTest1”)，然后在log4j.properties中做出相应配置即可。别忘了不同日志要使用不同的logger(如输出到test1.log的要用logger1.info(“abc”))。

还有一个问题，就是这些自定义的**日志默认是同时输出到log4j.rootLogger所配置的日志中的**，如何能只让它们输出到自己指定的日志中呢？别急，这里有个开关：
log4j.additivity.myTest1 = false
它用来设置是否同时输出到log4j.rootLogger所配置的日志中，设为false就不会输出到其它地方啦！注意这里的”myTest1”是在程序中给logger起的那个自定义的名字！
如果只是不想同时输出这个日志到log4j.rootLogger所配置的logfile中，stdout里我还想同时输出呢！那也好办，把你的log4j.logger.myTest1 = DEBUG, test1改为下式就OK啦！
log4j.logger.myTest1=DEBUG, test1

# 一个 log4j 比较全面的配置文件
此配置文件包括: 输出到控制台, 文件, 回滚文件, 发送日志邮件, 输出到数据库日志表, 自定义标签等全套功能:
```properties
log4j.rootLogger=DEBUG,console,dailyFile,im
log4j.additivity.org.apache=true
# 控制台(console)
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.Threshold=DEBUG
log4j.appender.console.ImmediateFlush=true
log4j.appender.console.Target=System.err
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=[%-5p] %d(%r) —> [%t] %l: %m %x %n
# 日志文件(logFile)
log4j.appender.logFile=org.apache.log4j.FileAppender
log4j.appender.logFile.Threshold=DEBUG
log4j.appender.logFile.ImmediateFlush=true
log4j.appender.logFile.Append=true
log4j.appender.logFile.File=D:/logs/log.log4j
log4j.appender.logFile.layout=org.apache.log4j.PatternLayout
log4j.appender.logFile.layout.ConversionPattern=[%-5p] %d(%r) —> [%t] %l: %m %x %n
# 回滚文件(rollingFile)
log4j.appender.rollingFile=org.apache.log4j.RollingFileAppender
log4j.appender.rollingFile.Threshold=DEBUG
log4j.appender.rollingFile.ImmediateFlush=true
log4j.appender.rollingFile.Append=true
log4j.appender.rollingFile.File=D:/logs/log.log4j
log4j.appender.rollingFile.MaxFileSize=200KB
log4j.appender.rollingFile.MaxBackupIndex=50
log4j.appender.rollingFile.layout=org.apache.log4j.PatternLayout
log4j.appender.rollingFile.layout.ConversionPattern=[%-5p] %d(%r) —> [%t] %l: %m %x %n
# 定期回滚日志文件(dailyFile)
log4j.appender.dailyFile=org.apache.log4j.DailyRollingFileAppender
log4j.appender.dailyFile.Threshold=DEBUG
log4j.appender.dailyFile.ImmediateFlush=true
log4j.appender.dailyFile.Append=true
log4j.appender.dailyFile.File=D:/logs/log.log4j
log4j.appender.dailyFile.DatePattern=‘.’yyyy-MM-dd
log4j.appender.dailyFile.layout=org.apache.log4j.PatternLayout
log4j.appender.dailyFile.layout.ConversionPattern=[%-5p] %d(%r) —> [%t] %l: %m %x %n
# 应用于socket
log4j.appender.socket=org.apache.log4j.RollingFileAppender
log4j.appender.socket.RemoteHost=localhost
log4j.appender.socket.Port=5001
log4j.appender.socket.LocationInfo=true
# Set up for Log Factor 5
log4j.appender.socket.layout=org.apache.log4j.PatternLayout
log4j.appender.socket.layout.ConversionPattern=[%-5p] %d(%r) —> [%t] %l: %m %x %n
# Log Factor 5 Appender
log4j.appender.LF5_APPENDER=org.apache.log4j.lf5.LF5Appender
log4j.appender.LF5_APPENDER.MaxNumberOfRecords=2000
# 发送日志到指定邮件
log4j.appender.mail=org.apache.log4j.net.SMTPAppender
log4j.appender.mail.Threshold=FATAL
log4j.appender.mail.BufferSize=10
log4j.appender.mail.From = xxx@mail.com
log4j.appender.mail.SMTPHost=mail.com
log4j.appender.mail.Subject=Log4J Message
log4j.appender.mail.To= xxx@mail.com
log4j.appender.mail.layout=org.apache.log4j.PatternLayout
log4j.appender.mail.layout.ConversionPattern=[%-5p] %d(%r) —> [%t] %l: %m %x %n
# 应用于数据库
log4j.appender.database=org.apache.log4j.jdbc.JDBCAppender
log4j.appender.database.URL=jdbc:mysql://localhost:3306/test
log4j.appender.database.driver=com.mysql.jdbc.Driver
log4j.appender.database.user=root
log4j.appender.database.password=
log4j.appender.database.sql=INSERT INTO LOG4J (Message) VALUES('=[%-5p] %d(%r) —> [%t] %l: %m %x %n’)
log4j.appender.database.layout=org.apache.log4j.PatternLayout
log4j.appender.database.layout.ConversionPattern=[%-5p] %d(%r) —> [%t] %l: %m %x %n

# 自定义Appender
log4j.appender.im = net.cybercorlin.util.logger.appender.IMAppender
log4j.appender.im.host = mail.cybercorlin.net
log4j.appender.im.username = username
log4j.appender.im.password = password
log4j.appender.im.recipient = corlin@cybercorlin.net
log4j.appender.im.layout=org.apache.log4j.PatternLayout
log4j.appender.im.layout.ConversionPattern=[%-5p] %d(%r) —> [%t] %l: %m %x %n

```

#功能组件/日志/log4j/log4j配置

```java

```

