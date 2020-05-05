# Activiti6.0

## 1. 简介以及快速开始

### 1.1 BPMN

业务流程建模标记法(BPMN) 提供了标准的标记法,规范定义了业务流程的符号及模型, 并且为流程定义设定了转换格式. 

BPMN 定义了 5 个基础的元素类型:

- 流对象: ==Flow Objects==, 是用于在一个业务流程中定义 **==行为==** 的图形元素. 主要有 ==**事件(Events),活动(Activities),网关(Gateways) 三种流对象**==.
- 数据: 
- 连接对象:
- 泳道:
- 制品:



## 2. 源码解析以及运行流程解读

==**核心模块**==:

- ==module/activiti-engine==: 核心引擎
- ==module/activiti-Spring==: Spring 集成模块
- ==module/activiti-spring-boot==: SpringBoot 集成模块
- ==module/activiti-rest==: 对外提供 rest api 模块
- ==module/activiti-form-engine==: 表单引擎模块
- ==module/activiti-ldap==: 集成 ldap 模块.



#### 2.2 运行流程解读

- 首先创建 ProcessEngineConfiguration. (默认加载 activiti.cfg.xml). 可进行一些对于 ProcessEngine 的配置(数据库啊什么的)

- 通过 ProcessEngineConfiguration 创建 ProcessEngine 对象.

- 通过 ProcessEngine 对象可以创建各种在流程中的服务

- ```java
  ProcessEngine processEngine = configuration.buildProcessEngine();
  
          /**
           * 获取 RepositoryService: 提供管理和控制发布包 和 流程定义的操作.
           *  部署流程定义
           *  查询引擎中的已有发布包和流程定义
           *  暂定和激活发布包
           *  获取发布包中的资源, 比如 xml 文件或者是流程图片等
           */
          RepositoryService repositoryService = processEngine.getRepositoryService();
  
          /**
           * 获取 RuntimeService: 负责启动一个流程实例.
           *  对于每个流程来说, 同一时间内可以有多个实例在执行.
           *  RuntimeService 可以获取和保存流程实例中的变量.
           *  或者是用于查询流程实例, 执行实例, 触发实例等.
           */
          RuntimeService runtimeService = processEngine.getRuntimeService();
  
          /**
           * 获取 TaskService: 任务相关的服务
           *  查询分配和用户或者用户组的任务信息
           *  创建独立运行与流程外的任务.
           *  手动设置任务与用户的关联关系
           *  认领任务(claim)
           *  完成任务(complete)
           */
          TaskService taskService = processEngine.getTaskService();
  
          /**
           * 获取 ManagementService: 管理服务.
           *  提供查询和管理异步的功能.
           *      异步操作的用途包含定时器, 延迟, 暂停, 激活等.
           */
          ManagementService managementService = processEngine.getManagementService();
  
          /**
           * 获取 IdentityService: 负责管理(创建, 更新, 删除, 查询)群组和用户.
           *  activi 执行时不会对用户执行检查. 任务可以分配给任何人, 无论这个用户是否存在.
           */
          IdentityService identityService = processEngine.getIdentityService();
  
          /**
           * 获取 FormService: 表单服务(提供启动表单和任务表单两个概念)
           *  即在流程实例启动前展示给用户的(启动表单). 和完成任务时展示给用户的(任务表单)两种表单
           *  这是个可选服务, 表单不一定要嵌入到流程定义中.
           */
          FormService formService = processEngine.getFormService();
  
          /**
           * 获取 HistoryService: 历史数据服务
           *  执行流程时, 引擎会保存如实例启动事件,任务参与者, 完成时间, 执行路径等数据. HistoryService 通过查询功能获取这些数据
           */
          HistoryService historyService = processEngine.getHistoryService();
  ```

  

## 3. Helloworld

==**注意 :**== 要加载的流程定义文件的后缀必须是 ==**.bpmn20.xml**==

1. 创建流程图()
2. 



## 4. ProcessEngineConfiguration 配置

### 4.1 ProcessEngineConfiguration 简介

Activiti中, 流程引擎 ProcessEngine 的配置是通过 ==**ProcessEngineConfiguration**== 配置类来完成的.

==**ProcessEngineConfiguration**== 会==**默认**==用 ==**activiti.cfg.xml**== 来作为配置的辅助, 通过这个配置文件的配置, 构建自己的配置对象, 然后构建出 ProcessEngine.

- 默认加载并解析 ==activiti.cfg.xml== 配置文件.
- 提供了多个静态方法创建配置对象.
- 实现了多个==**基于不同场景的子类**==, 配置方式非常灵活.

```java
// 加载默认配置文件, 创建默认的 ProcessEngineConfiguration, bean 名称为 processEngineConfiguration
public static ProcessEngineConfiguration createProcessEngineConfigurationFromResourceDefault() {
    return createProcessEngineConfigurationFromResource("activiti.cfg.xml", "processEngineConfiguration");
  }
// 加载指定配置文件, 创建 ProcessEngineConfiguration, bean 名称为 processEngineConfiguration
  public static ProcessEngineConfiguration createProcessEngineConfigurationFromResource(String resource) {
    return createProcessEngineConfigurationFromResource(resource, "processEngineConfiguration");
  }
// 加载指定配置文件, 和 bean 名称的方式创建 ProcessEngineConfiguration
  public static ProcessEngineConfiguration createProcessEngineConfigurationFromResource(String resource, String beanName) {
    return BeansConfigurationHelper.parseProcessEngineConfigurationFromResource(resource, beanName);
  }

  public static ProcessEngineConfiguration createProcessEngineConfigurationFromInputStream(InputStream inputStream) {
    return createProcessEngineConfigurationFromInputStream(inputStream, "processEngineConfiguration");
  }

  public static ProcessEngineConfiguration createProcessEngineConfigurationFromInputStream(InputStream inputStream, String beanName) {
    return BeansConfigurationHelper.parseProcessEngineConfigurationFromInputStream(inputStream, beanName);
  }
// 直接创建
  public static ProcessEngineConfiguration createStandaloneProcessEngineConfiguration() {
    return new StandaloneProcessEngineConfiguration();
  }
// 创建基于内存的 ProcessEngineConfiguration
  public static ProcessEngineConfiguration createStandaloneInMemProcessEngineConfiguration() {
    return new StandaloneInMemProcessEngineConfiguration();
  }

```



### 4.2 ProcessEngineConfiguration 可配置项

#### 4.2.1 数据库配置

- 缺省配置默认使用 H2 内存数据库

```java
//数据库更新策略: false:ProcessEngine 启动时检查数据库版本, 发生不匹配抛异常. 不做更新. 一般都会采用这种方式.
public static final String DB_SCHEMA_UPDATE_FALSE = "false";
//数据库更新策略: create-drop:ProcessEngine 启动时创建数据库表, 结束时删除表结构.
public static final String DB_SCHEMA_UPDATE_CREATE_DROP = "create-drop";
//数据库更新策略: true:ProcessEngine 启动时自动检查并更新数据库, 不存在会创建. 一般开发环境会使用这种方式.
public static final String DB_SCHEMA_UPDATE_TRUE = "true";
// 数据库类型, 默认是基于内存的 H2 数据库.
protected String databaseType;
// 可以对 Activities 的表名加前缀
protected String databaseTablePrefix = "";
// 执行完的流程会被迁移到历史数据内
protected boolean isDbHistoryUsed = true;
// 
protected boolean isDbIdentityUsed = true;
```



#### 4.2.2 日志和数据记录配置

- 日志组建的关系及 MDC
- 配置历史记录级别 (HistoryLevel)
- 配置基于 JDBC 的事件日志(Event logging)



==**MDC**==

MDC 是指将一些上下文数据存储在ThreadLocal 中, 当需要的时候, 可以将这些信息输出出来.

- Activiti的 MDC 默认没有开启, 需要手动设置 ==**LogMDC.setMDCEnable(true)**==. 开启后就会将 activiti 的一些上下文信息存储在ThreadLocal 中,

- 开启 MDC 后, 配置 logback.xml 日志模板 %X{MDCProcessInstanceID}
- 流程只有在执行过程出现异常的时候才会记录 MDC 信息



==**HistoryLevel**==

- none: 不记录历史流程, 性能高. 流程结束后不可读取.
- activiti: 会记录流程实例和活动实例. 流程变量不同步.就是只会记录什么时候发生了什么事, 但是具体的细节并没有记录下来.
- audit: 默认值. 在 Activiti 基础上同步变量值, 保存表单数据.
- full: 性能较差. 记录所有实例和变量细节变化.



==**Event Logging**==

- ==**实验性**==的事件记录机制. 性能影响较大. 默认没有开启
- 打开后, 会记录整个流程的所有数据的变化过程. 表数据量的增长会很快
- 日志内容是 json 格式.

### 4.3 构建 ProcessEngine

通过 ==**ProcessEngineConfiguration#buildProcessEngine()**== 方法构建 ProcessEngine 对象.



## 5. 核心对象

通过 ==**activiti.cfg.xml**== 可以构建 ==**ProcessEngineConfiguration**== 对象. 

通过 ==**ProcessEngineConfiguration**== 对象可以构建 ==**ProcessEngine**== 对象.

通过 ==**ProcessEngine**== 可以构建其他的比较重要的 API:

- ==RepositoryService== : 管理流程定义文件的管理.
- ==RuntimeService==: 对流程进行控制的服务. 启动流程实例, 对流程实例进行暂停等. 可查询正在执行的流程实例等
- ==TaskService==: 对运行中的 UserTask 进行管理.
- ==IdentityService==: 对用户组的管理. 创建等
- ==FormService==: 解析流程定义中涉及的表单. 
- ==HistoryService==: 对运行结束的流程实例的查询功能.
- ==ManagementService==: 对定时任务等的管理.
- DynamicBPMService



### 5.1 RepositoryService

- 管理流程定义文件 xml 及静态资源的服务
- 对特定的流程的暂停和激
- 流程定义启动权限管理.



涉及到的 API:

- 流程部署对象: Deployment
- 流程定义文件对象: ProcessDefinition
- 部署文件构造器: DeploymentBuilder. 
- 部署文件查询器: DeploymentQuery.
- 流程定义文件查询对象: ProcessDefinitionQuery. 
- 流程定义的 java 格式: BpmnModel





