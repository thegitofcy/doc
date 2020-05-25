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

1. **创建流程图**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.activiti.org/test">
    <process id="second_approve" name="二级审批流程" isExecutable="true">
        <startEvent id="startEvent" name="开始"></startEvent>
        <userTask id="submitForm" name="填写审批信息">
            <extensionElements>
                <activiti:formProperty id="message" name="申请信息" type="string" required="true"></activiti:formProperty>
                <activiti:formProperty id="name" name="申请人姓名" type="string" required="true"></activiti:formProperty>
                <activiti:formProperty id="submitTime" name="提交时间" type="date" datePattern="yyyy-MM-dd" required="true"></activiti:formProperty>
                <activiti:formProperty id="submitType" name="确认申请" type="string" required="true"></activiti:formProperty>
            </extensionElements>
        </userTask>
        <sequenceFlow id="flow1" sourceRef="startEvent" targetRef="submitForm"></sequenceFlow>
        <exclusiveGateway id="decideSubmit" name="提交OR取消"></exclusiveGateway>
        <sequenceFlow id="flow2" sourceRef="submitForm" targetRef="decideSubmit"></sequenceFlow>
        <userTask id="tl_approve" name="主管审批">
            <extensionElements>
                <activiti:formProperty id="tlApprove" name="主管审批结果" type="string"></activiti:formProperty>
                <activiti:formProperty id="tlMessage" name="主管备注" type="string" required="true"></activiti:formProperty>
            </extensionElements>
        </userTask>
        <sequenceFlow id="flow3" sourceRef="decideSubmit" targetRef="tl_approve">
            <conditionExpression xsi:type="tFormalExpression"><![CDATA[${submitType == "y" || submitType == "Y"}]]></conditionExpression>
        </sequenceFlow>
        <exclusiveGateway id="decideTLApprove" name="主管审批校验"></exclusiveGateway>
        <sequenceFlow id="flow4" sourceRef="tl_approve" targetRef="decideTLApprove"></sequenceFlow>
        <userTask id="hr_approve" name="人事审批">
            <extensionElements>
                <activiti:formProperty id="hrApprove" name="人事审批结果" type="string" required="true"></activiti:formProperty>
                <activiti:formProperty id="hrMessage" name="人事审批备注" type="string" required="true"></activiti:formProperty>
            </extensionElements>
        </userTask>
        <sequenceFlow id="flow5" sourceRef="decideTLApprove" targetRef="hr_approve">
            <conditionExpression xsi:type="tFormalExpression"><![CDATA[${tlApprove == "y" || tlApprove == "Y"}]]></conditionExpression>
        </sequenceFlow>
        <exclusiveGateway id="decideHRApprove" name="人事审批校验"></exclusiveGateway>
        <sequenceFlow id="flow6" sourceRef="hr_approve" targetRef="decideHRApprove"></sequenceFlow>
        <endEvent id="endEvent" name="结束"></endEvent>
        <sequenceFlow id="flow7" sourceRef="decideHRApprove" targetRef="endEvent">
            <conditionExpression xsi:type="tFormalExpression"><![CDATA[${hrApprove == "y" || hrApprove == "Y"}]]></conditionExpression>
        </sequenceFlow>
        <endEvent id="endEventCancel" name="取消"></endEvent>
        <sequenceFlow id="flow8" sourceRef="decideSubmit" targetRef="endEventCancel">
            <conditionExpression xsi:type="tFormalExpression"><![CDATA[${submitType == "n" || submitType == "N"}]]></conditionExpression>
        </sequenceFlow>
        <sequenceFlow id="flow9" sourceRef="decideTLApprove" targetRef="submitForm">
            <conditionExpression xsi:type="tFormalExpression"><![CDATA[${tlApprove == "n" || tlApprove == "N"}]]></conditionExpression>
        </sequenceFlow>
        <sequenceFlow id="flow10" sourceRef="decideHRApprove" targetRef="submitForm">
            <conditionExpression xsi:type="tFormalExpression"><![CDATA[${hrApprove == "n" || hrApprove == "N"}]]></conditionExpression>
        </sequenceFlow>
    </process>
    <bpmndi:BPMNDiagram id="BPMNDiagram_second_approve">
        <bpmndi:BPMNPlane bpmnElement="second_approve" id="BPMNPlane_second_approve">
            <bpmndi:BPMNShape bpmnElement="startEvent" id="BPMNShape_startEvent">
                <omgdc:Bounds height="35.0" width="35.0" x="160.0" y="180.0"></omgdc:Bounds>
            </bpmndi:BPMNShape>
            <bpmndi:BPMNShape bpmnElement="submitForm" id="BPMNShape_submitForm">
                <omgdc:Bounds height="55.0" width="105.0" x="240.0" y="170.0"></omgdc:Bounds>
            </bpmndi:BPMNShape>
            <bpmndi:BPMNShape bpmnElement="decideSubmit" id="BPMNShape_decideSubmit">
                <omgdc:Bounds height="40.0" width="40.0" x="390.0" y="178.0"></omgdc:Bounds>
            </bpmndi:BPMNShape>
            <bpmndi:BPMNShape bpmnElement="tl_approve" id="BPMNShape_tl_approve">
                <omgdc:Bounds height="55.0" width="105.0" x="475.0" y="171.0"></omgdc:Bounds>
            </bpmndi:BPMNShape>
            <bpmndi:BPMNShape bpmnElement="decideTLApprove" id="BPMNShape_decideTLApprove">
                <omgdc:Bounds height="40.0" width="40.0" x="625.0" y="179.0"></omgdc:Bounds>
            </bpmndi:BPMNShape>
            <bpmndi:BPMNShape bpmnElement="hr_approve" id="BPMNShape_hr_approve">
                <omgdc:Bounds height="55.0" width="105.0" x="710.0" y="172.0"></omgdc:Bounds>
            </bpmndi:BPMNShape>
            <bpmndi:BPMNShape bpmnElement="decideHRApprove" id="BPMNShape_decideHRApprove">
                <omgdc:Bounds height="40.0" width="40.0" x="860.0" y="180.0"></omgdc:Bounds>
            </bpmndi:BPMNShape>
            <bpmndi:BPMNShape bpmnElement="endEvent" id="BPMNShape_endEvent">
                <omgdc:Bounds height="35.0" width="35.0" x="945.0" y="183.0"></omgdc:Bounds>
            </bpmndi:BPMNShape>
            <bpmndi:BPMNShape bpmnElement="endEventCancel" id="BPMNShape_endEventCancel">
                <omgdc:Bounds height="35.0" width="35.0" x="510.0" y="250.0"></omgdc:Bounds>
            </bpmndi:BPMNShape>
            <bpmndi:BPMNEdge bpmnElement="flow1" id="BPMNEdge_flow1">
                <omgdi:waypoint x="195.0" y="197.0"></omgdi:waypoint>
                <omgdi:waypoint x="240.0" y="197.0"></omgdi:waypoint>
            </bpmndi:BPMNEdge>
            <bpmndi:BPMNEdge bpmnElement="flow2" id="BPMNEdge_flow2">
                <omgdi:waypoint x="345.0" y="197.0"></omgdi:waypoint>
                <omgdi:waypoint x="390.0" y="198.0"></omgdi:waypoint>
            </bpmndi:BPMNEdge>
            <bpmndi:BPMNEdge bpmnElement="flow3" id="BPMNEdge_flow3">
                <omgdi:waypoint x="430.0" y="198.0"></omgdi:waypoint>
                <omgdi:waypoint x="475.0" y="198.0"></omgdi:waypoint>
            </bpmndi:BPMNEdge>
            <bpmndi:BPMNEdge bpmnElement="flow4" id="BPMNEdge_flow4">
                <omgdi:waypoint x="580.0" y="198.0"></omgdi:waypoint>
                <omgdi:waypoint x="625.0" y="199.0"></omgdi:waypoint>
            </bpmndi:BPMNEdge>
            <bpmndi:BPMNEdge bpmnElement="flow5" id="BPMNEdge_flow5">
                <omgdi:waypoint x="665.0" y="199.0"></omgdi:waypoint>
                <omgdi:waypoint x="710.0" y="199.0"></omgdi:waypoint>
            </bpmndi:BPMNEdge>
            <bpmndi:BPMNEdge bpmnElement="flow6" id="BPMNEdge_flow6">
                <omgdi:waypoint x="815.0" y="199.0"></omgdi:waypoint>
                <omgdi:waypoint x="860.0" y="200.0"></omgdi:waypoint>
            </bpmndi:BPMNEdge>
            <bpmndi:BPMNEdge bpmnElement="flow7" id="BPMNEdge_flow7">
                <omgdi:waypoint x="900.0" y="200.0"></omgdi:waypoint>
                <omgdi:waypoint x="945.0" y="200.0"></omgdi:waypoint>
            </bpmndi:BPMNEdge>
            <bpmndi:BPMNEdge bpmnElement="flow8" id="BPMNEdge_flow8">
                <omgdi:waypoint x="410.0" y="218.0"></omgdi:waypoint>
                <omgdi:waypoint x="410.0" y="266.0"></omgdi:waypoint>
                <omgdi:waypoint x="510.0" y="267.0"></omgdi:waypoint>
            </bpmndi:BPMNEdge>
            <bpmndi:BPMNEdge bpmnElement="flow9" id="BPMNEdge_flow9">
                <omgdi:waypoint x="645.0" y="219.0"></omgdi:waypoint>
                <omgdi:waypoint x="644.0" y="297.0"></omgdi:waypoint>
                <omgdi:waypoint x="292.0" y="296.0"></omgdi:waypoint>
                <omgdi:waypoint x="292.0" y="225.0"></omgdi:waypoint>
            </bpmndi:BPMNEdge>
            <bpmndi:BPMNEdge bpmnElement="flow10" id="BPMNEdge_flow10">
                <omgdi:waypoint x="880.0" y="180.0"></omgdi:waypoint>
                <omgdi:waypoint x="880.0" y="133.0"></omgdi:waypoint>
                <omgdi:waypoint x="290.0" y="132.0"></omgdi:waypoint>
                <omgdi:waypoint x="292.0" y="170.0"></omgdi:waypoint>
            </bpmndi:BPMNEdge>
        </bpmndi:BPMNPlane>
    </bpmndi:BPMNDiagram>
</definitions>
```



2. **创建项目, 引入如下依赖**

```xml
<dependencies>
        <dependency>
            <groupId>org.activiti</groupId>
            <artifactId>activiti-engine</artifactId>
            <version>6.0.0</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>23.0</version>
        </dependency>

        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>1.3.176</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.0</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-api -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.25</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/ch.qos.logback/logback-classic -->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.2.3</version>
        </dependency>
    </dependencies>
```

3. **编写代码**

```java
package com.cy;

import com.google.common.collect.Maps;
import lombok.extern.slf4j.Slf4j;
import org.activiti.engine.*;
import org.activiti.engine.form.FormProperty;
import org.activiti.engine.form.TaskFormData;
import org.activiti.engine.impl.form.DateFormType;
import org.activiti.engine.impl.form.StringFormType;
import org.activiti.engine.repository.Deployment;
import org.activiti.engine.repository.DeploymentBuilder;
import org.activiti.engine.repository.ProcessDefinition;
import org.activiti.engine.runtime.ProcessInstance;
import org.activiti.engine.task.Task;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;
import java.util.Map;
import java.util.Scanner;

/**
 * @program: activiti
 * @description:
 * @author: cy
 * @create: 2020/05/18
 **/
@Slf4j
public class DemoMain {

    public static void main(String[] args) {
        log.info("启动程序");
        // 1. 创建流程引擎
        ProcessEngine processEngine = getProcessEngine();

        // 2. 部署流程定义文件, 获取流程定义对象
        ProcessDefinition processDefinition = getProcessDefinition(processEngine);

        // 3. 启动运行流程, 获取流程实例
        ProcessInstance processInstance = getProcessInstance(processEngine, processDefinition);

        // 4. 处理流程任务
        Scanner scanner = new Scanner(System.in);
        while (null != processInstance && !processInstance.isEnded()) {
            TaskService taskService = processEngine.getTaskService();
            List<Task> list = taskService.createTaskQuery().list();
            log.info("待处理任务数量 [{}]", list.size());
            for (Task task : list) {
                log.info("待处理任务: [{}]", task.getName());
                FormService formService = processEngine.getFormService();
                TaskFormData taskFormData = formService.getTaskFormData(task.getId());
                List<FormProperty> formProperties = taskFormData.getFormProperties();
                Map<String, Object> variables = Maps.newHashMap();
                for (FormProperty formProperty : formProperties) {
                    if (StringFormType.class.isInstance(formProperty.getType())) {
                        log.info("请输入: [{}]", formProperty.getName());
                        String line = scanner.nextLine();
                        log.info("您输入的是: [{}]", line);
                        variables.put(formProperty.getId(), line);
                    } else if (DateFormType.class.isInstance(formProperty.getType())) {
                        log.info("请输入: [{}], 格式: [yyyy-MM-dd]", formProperty.getName());
                        String line = scanner.nextLine();
                        SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
                        Date parse = null;
                        try {
                            parse = format.parse(line);
                        } catch (ParseException e) {
                            e.printStackTrace();
                        }
                        log.info("您输入的是: [{}]", line);
                        variables.put(formProperty.getId(), parse);
                    } else {
                        log.info("类型暂不支持!", formProperty.getType());
                    }
                }
                // 提交任务
                taskService.complete(task.getId(), variables);
                processInstance = processEngine.getRuntimeService().createProcessInstanceQuery().processInstanceId(processInstance.getId()).singleResult();
            }
        }
        log.info("结束程序");
    }

    private static ProcessInstance getProcessInstance(ProcessEngine processEngine, ProcessDefinition processDefinition) {
        RuntimeService runtimeService = processEngine.getRuntimeService();
        ProcessInstance processInstance = runtimeService.startProcessInstanceById(processDefinition.getId());
        log.info("启动流程 ==> [{}]", processInstance.getProcessDefinitionKey());
        return processInstance;
    }

    private static ProcessDefinition getProcessDefinition(ProcessEngine processEngine) {
        RepositoryService repositoryService = processEngine.getRepositoryService();
        DeploymentBuilder deploymentBuilder = repositoryService.createDeployment();
        deploymentBuilder.addClasspathResource("second_approve.bpmn20.xml");
        Deployment deploy = deploymentBuilder.deploy();

        ProcessDefinition processDefinition = repositoryService.createProcessDefinitionQuery()
                .deploymentId(deploy.getId())
                .singleResult();
        log.info("流程定义对象 processDefinition ==> 流程定义文件: [{}], 流程 ID: [{}]", processDefinition.getName(), processDefinition.getId());
        return processDefinition;
    }

    private static ProcessEngine getProcessEngine() {
        ProcessEngineConfiguration configuration = ProcessEngineConfiguration.createStandaloneInMemProcessEngineConfiguration();
        ProcessEngine processEngine = configuration.buildProcessEngine();
        String name = processEngine.getName();
        String version = processEngine.VERSION;
        log.info("process ==> name: [{}], version: [{}]", name, version);
        return processEngine;
    }
}

```



## 4. ProcessEngineConfiguration 配置

### 4.1 ProcessEngineConfiguration 简介

Activiti中, 流程引擎 ProcessEngine 的配置是通过 ==**ProcessEngineConfiguration**== 配置类来完成的.

==**ProcessEngineConfiguration**== 会==**默认**==用 ==**activiti.cfg.xml**== 来作为配置的辅助, 通过这个配置文件的配置, 构建自己的配置对象, 然后构建出 ==ProcessEngine==. 然后就可以通过 ==ProcessEngine== 获取到需要用的各种 Service

- 默认加载并解析 ==activiti.cfg.xml== 配置文件.
- 提供了==多个静态方法(7 个)==创建配置对象.
- 实现了多个==**基于不同场景的子类(6 个)**==, 配置方式非常灵活.

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



**实现类**

![image-20200518170015684](/Users/cy/Library/Application Support/typora-user-images/image-20200518170015684.png)

1. ProcessEngineConfigurationImpl : 抽象实现类
2. StandaLoneProcessEngineConfiguration : 独立部署
3. SpringProcessEngineConfiguration: 基于 Spring, 提供了事务的扩展, 数据源的扩展, 定义了一个可以自动装载

某个路径下符合规范的流程定义文件.





### 4.2 ProcessEngineConfiguration 数据库配置

- 缺省配置默认使用 H2 内存数据库

```properties
databaseSchemaUpdate : 数据库更新策略,ProcessEngine 启动时检查数据库版本. 
		fasle: 表示发生不匹配抛异常. 不做更新. 一般生产环境都采用这种方式.
		true: 启动时自动检查并更新数据库表, 不存在会创建. 一般开发环境会使用这种方式.
		create-drop: 启动时创建数据库表结构, 结束时删除表结构. 一般单元测试的时候使用这种方式. 当使用这种方式的时候, 需要保证数据库中没有要创建的表.

databaseTablePrefix: 可以为表添加前缀
databaseType: 数据库类型

isDbHistoryUsed: 执行完的流程是否被迁移到历史数据内.
		true: 执行完的流程数据会被迁移到历史数据库表内.
		false: 不会迁移
isDbIdentityUsed: 
```



### 4.3 日志和数据记录配置

- 日志组建的关系及 MDC
- 配置历史记录级别 (HistoryLevel)
- 配置基于 JDBC 的事件日志(Event logging)



###  4.3 ==**MDC**==

MDC 是指将一些上下文数据存储在==**ThreadLocal**== 中, 当需要的时候, 可以将这些信息输出出来.

- Activiti的 MDC 默认没有开启, 需要手动设置 ==**LogMDC.setMDCEnable(true)**==. 开启后就会将 activiti 的一些上下文信息存储在ThreadLocal 中

- 开启 MDC 后, 配置 logback.xml 日志模板 %X{MDCProcessInstanceID}
- Activiti 流程==只有在执行过程出现异常的时候才会记录 MDC 信息==



### 4.4 ==**HistoryLevel**==: 历史记录配置

- ==**none**==: 不记录历史流程, 性能高. 流程结束后不可读取.
- ==**activiti**==: 会记录流程实例和活动实例. 流程变量不同步.就是只会记录什么时候发生了什么事, 但是具体的细节并没有记录下来.
- ==**audit**==: 默认值. 在 Activiti 基础上同步变量值, 保存表单数据.
- ==**full**==: 性能较差. 记录所有实例和变量细节变化.



### 4.5 事件处理及监听器配置

- ==**EventLog**==: 基于 DB的事件日志
- ==**EventListener**==: 事件监听器.

#### 4.5.1 EventLog 基于 DB 的事件日志.

==**Event Logging**== : 基于 DB 的事件日志

- ==**实验性**==的事件记录机制. 性能影响较大. 默认没有开启
- 打开后, 会记录整个流程的所有数据的变化过程. 表数据量的增长会很快
- 日志内容是 json 格式.
- ==**enableDatabaseEventLogging**== : activity 配置文件配置项, 是否开启基于 DB 的事件日志. 默认为 false.
- ==**ManagementService**== : 通过此 Service 来读取 EventLog 数据

配置:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" 
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans   http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneInMemProcessEngineConfiguration">
    <property name="enableDatabaseEventLogging" value="true"/>
  </bean>

</beans>
```



代码演示:

```java
@Slf4j
public class MyUnitTest_EventLog {

	@Rule
	public ActivitiRule activitiRule = new ActivitiRule("activiti_eventLog.cfg.xml");

	@Test
	@Deployment(resources = {"my-process.bpmn20.xml"})
	public void test() {
		ProcessInstance processInstance = activitiRule.getRuntimeService().startProcessInstanceByKey("my-process");
		Task task = activitiRule.getTaskService().createTaskQuery().singleResult();
		activitiRule.getTaskService().complete(task.getId());

		List<EventLogEntry> eventLogEntries = activitiRule
				.getManagementService()
				//.getEventLogEntriesByProcessInstanceId(processInstance.getId());	// 根据流程实例 ID 获取 EventLog 数据.
				.getEventLogEntries(0l, 100l);

		eventLogEntries.forEach(eventLogEntry -> {
			log.info("eventLogEntry ==> type: [{}], data: [{}]", eventLogEntry.getType(), new String(eventLogEntry.getData()));
		});
		log.info("eventLogEntries size: [{}]", eventLogEntries.size());
	}

}
```



#### 4.5.2 Listener监听器配置(2 种配置方式)

==**方式一:**==配置文件配置.在配置文件中也有两种配置方式.

- 方式一: 配置 ==**eventListeners**==, 监听所有事件派发通知.不区分事件类型
- 方式二: 配置 ==**typedEventListeners**==, 监听指定事件类型的通知.



==**方式二:**== bpmn流程定义文件配置. 通过标签进行配置

- ==**activiti:eventListener**==: 只监听特定流程定义的事件. 不是全局的



#### 4.5.3 Activiti 的事件监听







### 4.3 构建 ProcessEngine

通过 ==**ProcessEngineConfiguration#buildProcessEngine()**== 方法构建 ProcessEngine 对象.





## 5. 单元测试

**相关注解**

- ==**@Deployment**==: 在启动单元测试之前, 部署指定路径下的流程文件.
- ==**@Rule**==: 





## 6. 核心对象

通过 ==**activiti.cfg.xml**== 可以构建 ==**ProcessEngineConfiguration**== 流程引擎配置对象. 

通过 ==**ProcessEngineConfiguration**== 对象可以构建 ==**ProcessEngine**== 流程引擎对象.

通过 ==**ProcessEngine**== 可以构建其他的比较重要的 API:

- ==RepositoryService== : 流程存储服务. 管理流程定义文件的管理.
- ==RuntimeService==: 对流程进行控制的服务. 启动流程实例, 对流程实例进行暂停等. 可查询正在执行的流程实例等
- ==TaskService==: 对运行中的 UserTask 进行管理.
- ==IdentityService==: 对用户和用户组的管理. 创建用户或者用户组, 维护用户和用户组之间的关系等.
- ==FormService==: 解析流程定义中涉及的表单. (比较重要的表单: 启动表单以及 UserTask 的表单)
- ==HistoryService==: 对运行结束的流程实例的查询功能.
- ==ManagementService==: 对定时任务等的管理, 获取 EventLog 数据等.
- DynamicBPMService



### 6.1 RepositoryService: 流程存储服务

- 管理流程定义文件 xml 及静态资源的服务
- 对特定的流程(指==**流程定义**==)的暂停和激活
- 流程定义启动权限管理.(添加用户和用户组)
- 需要注意的是, 每次部署, id 和 version 都会更新.



==**涉及到的 API:**==

- 流程部署对象: ==**Deployment**==, 
- 流程定义文件对象: ==**ProcessDefinition**==, 
- 部署文件构造器: ==**DeploymentBuilder**==. 
- 部署文件查询器: ==**DeploymentQuery**==.
- 流程定义文件查询对象: ==**ProcessDefinitionQuery**==. 
- 流程定义的 java 格式: ==**BpmnModel**==



一般开发步骤:

```properties
获取 ProcessEngine 对象
1: 创建 ProcessEngineConfiguration 对象 config
2: 创建 ProcessEngine对象. processEngine = config.buildProcessEngine();

部署步骤 : 
1: 获取 RepositoryService. repositoryService = processEngine.getRepositoryService();
2: 创建 Deploymentbuilder 对象. builder = repositoryService.createDeployment();
3: 可以对将要部署的流程进行命名. builder.name("测试部署资源");
4: 添加要部署的资源. builder.addClasspathResource("xxxx.bpmn20.xml"); // xml 的名称必须要以 .bpmn20.xml 结尾, 否则会找不到文件, 可以一次部署一个文件也可以一次部署多个文件. 总之一次无论部署多少个文件, 都只有一个部署对象
5: 完成部署. Deployment deployment = builder.deploy();

相关查询:(通过 Repository 获取到 Deployment 查询对象, ProcessDefinition查询对象等)
```



==**Demo:**==

```java
/**
 * 流程存储服务
 */
@Slf4j
public class MyUnitTest_RepositoryService {

	@Rule
	public ActivitiRule activitiRule = new ActivitiRule();

	/**
	 * 部署流程定义文件
	 */
	@Test
	public void deployTest() {
		RepositoryService repositoryService = activitiRule.getRepositoryService();

		DeploymentBuilder deploymentBuilder = repositoryService.createDeployment();
		deploymentBuilder.name("第一次部署: 一次部署多个");
		deploymentBuilder.addClasspathResource("my-process.bpmn20.xml");
		deploymentBuilder.addClasspathResource("second_approve.bpmn20.xml");
		Deployment deploy1 = deploymentBuilder.deploy();

		DeploymentBuilder deploymentBuilder2 = repositoryService.createDeployment();
		deploymentBuilder2.name("第二次部署: 一次部署多个");
		deploymentBuilder2.addClasspathResource("my-process.bpmn20.xml");
		deploymentBuilder2.addClasspathResource("second_approve.bpmn20.xml");
		Deployment deploy2 = deploymentBuilder2.deploy();

		List<Deployment> deployments = repositoryService
				.createDeploymentQuery()
//				.deploymentId(deploy.getId())
				.orderByDeploymenTime().asc()
				.listPage(0, 100);
		deployments.forEach(deployment -> {
			log.info("deployment ==> [{}]", deployment);
		});
		log.info("deployments size ==> [{}]", deployments.size());

		List<ProcessDefinition> processDefinitions = repositoryService
				.createProcessDefinitionQuery()
//				.deploymentId(deploy.getId())
				.orderByProcessDefinitionKey().asc()
				.listPage(0, 100);
		processDefinitions.forEach(processDefinition -> {
			log.info("processDefinition ==> id: [{}], name: [{}], key: [{}], version: [{}]",
					processDefinition.getId(),
					processDefinition.getName(),
					processDefinition.getKey(),
					processDefinition.getVersion());
		});
	}

	/**
	 * 维护用户和用户组的关系
	 * 	将用户和用户组关联起来
	 */
	@Test
	@org.activiti.engine.test.Deployment(resources = "my-process.bpmn20.xml")
	public void candidateTest(){
		RepositoryService repositoryService = activitiRule.getRepositoryService();
		ProcessDefinition processDefinition = repositoryService.createProcessDefinitionQuery().singleResult();

		// 添加用户和用户组
		repositoryService.addCandidateStarterUser(processDefinition.getId(), "user");
		repositoryService.addCandidateStarterGroup(processDefinition.getId(), "groupM");

		List<IdentityLink> identityLinks = repositoryService
				.getIdentityLinksForProcessDefinition(processDefinition.getId());
		identityLinks.forEach(identityLink -> {
			log.info("identityLink ==> [{}]", identityLink);
		});
	}

}
```



### 6.2 RuntimeService 流程运行控制服务

- ==**启动流程以及对流程数据的控制**==
- ==**流程实例(ProcessInstance)与执行流(Execution) 查询**==



==**RuntimeService 启动流程及变量管理:**==

- 启动流程的常用方式: 根据 id, key, message. ==每次部署后, id 和 version 都会更新, 所以使用 key 启动的时候, 默认使用对应的最新版本的==
- 启动流程可选参数: businessKey(业务唯一标志), variables, tenantId.
- 变量(variables) 的设计和获取.



==**ProcessInstance**== 和 ==**Execution**==

- 流程实例(ProcessInstance)表示一次工作流业务的数据实体, 每次启动工作流引擎, 都会创建一个流程实例对象.
- 执行流(Execution) 表示流程实例中具体的执行路径.  当==只有一条线的话, 每次的流程实例就对应一个执行流, 他们的 ID 是一致的.==
- ==ProcessInstance== 继承 ==Execution==, 相当于 ProcessInstance 是在 Execution 的基础上进行了一些扩展.



==**流程触发**==

- 使用 trigger 触发 ReceiveTask 节点.
- 触发信号捕获事件. signalEventReceived. 可以全局的发送一个信号
- 触发消息捕获事件 messageEventReceived. 针对一个流程实例发送一个消息



### 6.3 TaskService 任务管理服务

`MyUnitTest_TaskService`

- 对==**用户任务(UserTask)**==和涉及到用户任务的流程进行控制
- 设置用户任务的==**权限信息**==(拥有者, 候选人, 办理人)
- 针对用户任务添加任务附件, 任务评论和事件记录



==**TaskService 对 Task 管理与流程控制.**==

- Task 对象的创建,删除(但是一般不会手动的操作)
- 查询 Task, 并驱动 Task 节点完成执行.
- 在 UserTask 处理过程中, 对参数变量(variable)进行设置.



==**TaskService 设置 Task 权限信息**==

- 候选用户(candidateUser) 和候选组(candidateGroup)
- 指定拥有人(Owner, 一般就是流程的发起人) 和办理人(Assignee). 
- 通过 claim 设置办理人(一般使用这种方式来设置办理人), 使用 claim 设置的时候, 如果发现当前已经有办理人了, 就会进行判断, 如果发现当前办理人不是当前的参数, 就会报一个异常. 



==**TaskService 设置 Task 附加信息**==

- 任务附件(Attachment) 创建与查询. 场景: 比如报销审批时, 需要一发票电子文件等.
  - `taskService.createAttachment`
  - `taskService.getTaskAttachments`
- 任务评论(comment) 创建与查询. 场景: 比如和业务关系不是特别强的备注等.
- 事件记录(Event) 创建与查询