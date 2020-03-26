Activiti

## 工作流的概念

A .人物：美女，程序员，领导

B.事件(动作)：请假，批准，不批准

工作流(Workflow)，就是"业务过程的部分或整体再计算机应用环境下的自动化"，它主要的解决的是''使在多个参与者之间按照某种预定义的规则传递文档，信息或任务等过程自动进行，从而实现某哥预期的业务目标，或者促使此目标的实现"。

工作流管理系统(Workflow Management System,WfMS)是一个软件系统，它完成工作量的定义和管理，并按照在系统中预先定义好的工作流规则进行工作流实例的执行。工作流管理系统不是企业的业务系统，而是为企业的业务系统的运行提供了一个软件的支撑环境。

工作流管理联盟(WfMC，Workflow Management Coalition)给出的关于工作流管理系统的定义是：工作流管理系统是一个软件系统，它通过执行经过计算的流程定义去支持一批专门设定的业务流程。工作流管理系统被用来定义，管理，和执行工作流程。

工作流管理系统的目标：管理工作的流程以确保工作在正确的事件被期望的人员所执行——在自动化进行业务过程中插入人工的执行和干预。

## Activiti介绍

### 1.概述

Activiti5是由Alfresco软件在2010年5月17日发布的业务流程管理（BPM）框架，它是覆盖了业务流程管理、工作流、服务协作等领域的一个开源的、灵活的、易扩展的可执行流程语言框架。Activiti基于Apache许可的开源BPM平台，创始人Tom Baeyens是JBoss jBPM的项目架构师，它特色是提供了eclipse插件，开发人员可以通过插件直接绘画出业务流程图。

![](D:\images\2020-1-28\NMK((%QP}($~A{2V(Y8E)7.png)

### 2.工作流引擎

ProcessEngine对象，这是Activiti工作的核心。负责生成流程运行时的各种实例及数据，监控和管理流程的运行。

### 3.BPMN

业务流程建模与标注(Business Process Model and Notation，BPMN)，描述流程的基本符号，包括这些图元如何合成一个业务流程图(Business process Diagram)

### 4.数据库

Activiti数据库支持：

Activiti的后台是有数据库的支持，所有的表都以ACT_开头。二部分是表示表的用于的两个字母标识。用于也和服务的API对应。

ACT_ER_*:"RE"表示repository。这个前缀的表包含了流程定义和流程静态资源(图片，规则，等等)。

ACT_RU_*: 'RU'表示runtime。 这些运行时的表，包含流程实例，任务，变量，异步任务，等运行中的数据。 Activiti只在流程实例执行过程中保存这些数据， 在流程结束时就会删除这些记录。 这样运行时表可以一直很小速度很快。

ACT_ID_*: 'ID'表示identity。 这些表包含身份信息，比如用户，组等等。

ACT_HI_*: 'HI'表示history。 这些表包含历史数据，比如历史流程实例， 变量，任务等等。

ACT_GE_*: 通用数据， 用于不同场景下，如存放资源文件。

通用数据表：

1)act_ge_bytearray 二进制数据表

2)act_ge_property属性数据表存储整个流程引擎级别的数据，初始化表结构时，会默认插入三条记录

## 开发环境

### 1.添加maven依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.baerwang.activiti</groupId>
    <artifactId>Activiti-Demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!-- 配置版本 -->
    <properties>
        <spring.version>4.3.17.RELEASE</spring.version>
        <mysql.version>5.1.39</mysql.version>
        <!-- 注意只能使用2.0以下的版本 -->
        <activiti.version>5.22.0</activiti.version>
        <mybatis.version>3.4.6</mybatis.version>
        <!-- 注意只能使用2.0以下的版本 -->
        <log4j.version>1.2.17</log4j.version>
    </properties>

    <dependencies>
        <!-- activiti的依赖 -->
        <dependency>
            <groupId>org.activiti</groupId>
            <artifactId>activiti-engine</artifactId>
            <version>${activiti.version}</version>
        </dependency>
        <!-- ssm集成的时候使用 -->
        <dependency>
            <groupId>org.activiti</groupId>
            <artifactId>activiti-spring</artifactId>
            <version>${activiti.version}</version>
        </dependency>
        <!-- mysql驱动 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-expression</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <!-- mybatis -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>${mybatis.version}</version>
        </dependency>
        <!-- log4j -->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>${log4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.25</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-simple</artifactId>
            <version>1.7.25</version>
        </dependency>
    </dependencies>
</project>
```

### 2.配置日志log4j.properties

```properties
log4j.rootLogger=INFO, stdout

# Console Appender
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern= %d{hh:mm:ss,SSS} [%t] %-5p %c %x - %m%n
# Custom tweaks
log4j.logger.com.codahale.metrics=WARN
log4j.logger.com.ryantenney=WARN
log4j.logger.com.zaxxer=WARN
log4j.logger.org.apache=WARN
log4j.logger.org.hibernate=WARN
log4j.logger.org.hibernate.engine.internal=WARN
log4j.logger.org.hibernate.validator=WARN
log4j.logger.org.springframework=WARN
log4j.logger.org.springframework.web=WARN
log4j.logger.org.springframework.security=WARN
```

### 3.配置db.properties

```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/myactiviti
username=root
password=root
```

### 4.初始化数据库

#### 1.初始化表方式(创建类)

```java
    public static void main(String[] args) throws IOException {
		/*读取数据文件属性*/
        InputStream inputStream = InitTable.class.getClassLoader().getResourceAsStream("db.properties");
        Properties properties = new Properties();
        properties.load(inputStream);

        /*创建数据源*/
        DriverManagerDataSource driverManagerDataSource = new DriverManagerDataSource();
        driverManagerDataSource.setDriverClassName(properties.getProperty("driver"));
        driverManagerDataSource.setUrl(properties.getProperty("url"));
        driverManagerDataSource.setUsername(properties.getProperty("root"));
        driverManagerDataSource.setPassword(properties.getProperty("root"));

        /*创建流程引擎的配置*/
        ProcessEngineConfiguration configuration = ProcessEngineConfiguration.createStandaloneInMemProcessEngineConfiguration();
        /*configuration.setJdbcDriver(properties.getProperty("driver"));
        configuration.setJdbcUrl(properties.getProperty("url"));
        configuration.setJdbcUsername(properties.getProperty("root"));
        configuration.setJdbcPassword(properties.getProperty("root"));*/
        configuration.setDataSource(driverManagerDataSource);

        /**
         * ProcessEngineConfiguration.DB_SCHEMA_UPDATE_FALSE  如果数据库里面没有activit的表，也不会创建
         * ProcessEngineConfiguration.DB_SCHEMA_UPDATE_CREATE_DROP 创建表，使用完之后删除
         * ProcessEngineConfiguration.DB_SCHEMA_UPDATE_TRUE  如果数据库里面没有表，就创建
         *
         * dorp-create 代表如果数据库里面有表，那么先删除再创建
         *
         */
        /*配置表的初始化方式*/
        configuration.setDatabaseSchemaUpdate(ProcessEngineConfiguration.DB_SCHEMA_UPDATE_TRUE);

        /*得到流程引擎*/
        ProcessEngine processEngine = configuration.buildProcessEngine();
        System.out.println(processEngine);
    }
```

#### 2.初始化表方式(创建activiti.cfg.xml)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans.xsd">
    
    <bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
        <property name="jdbcDriver" value="com.mysql.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/myactiviti"/>
        <property name="jdbcUsername" value="root"/>
        <property name="jdbcPassword" value="root"/>
        <!--
        flase： 默认值。activiti在启动时，会对比数据库表中保存的版本，如果没有表或者版本不匹配，将抛出异常。
        true： activiti会对数据库中所有表进行更新操作。如果表不存在，则自动创建。
        create_drop： 在activiti启动时创建表，在关闭时删除表（必须手动关闭引擎，才能删除表）。
        drop-create： 在activiti启动时删除原来的旧表，然后在创建新表（不需要手动关闭引擎）。 -->
        <property name="databaseSchemaUpdate" value="drop-create"/>
    </bean>
</beans>
```

```java
public static void main(String[] args){
        ProcessEngineConfiguration configuration = ProcessEngineConfiguration.createProcessEngineConfigurationFromResource("/activiti.cfg.xml");
        /*得到流程引擎*/
        ProcessEngine processEngine = configuration.buildProcessEngine();
        System.out.println(processEngine);
    }
```

#### 3.创建类去初始化表

必须创建activiti.cfg.xml否则会报错，如图为ProcessEnginess源码

![](D:\images\2020-1-28\ZLGD9JE{QXU2Y$81KOINS2C.png)

```java
 public static void main(String[] args) {
        /*必须创建activiti.cfg.xml  并配置好数据库的信息*/
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        System.out.println(processEngine);
    }
```

## 核心API

### 1.ProcessEngine

1.在Activiti中最核心的类，其他的类都是由他而来。

2.产生方式

```java
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
```

在前面看到了两种创建ProcessEngine(流程引擎)的方式，这里要简化很多，调用ProcessEngines的getDefaultProcessEngine方法时回自动加成classpath下名为：activiti.cfg.xml文件。

3.可以产生RepositoryService

```java
RepositoryService repositoryService = processsEngine.getRepositoryService();
```

4.可以产生RuntimeService

```java
RuntimeService runtimeService = processEngine.getRuntimeService();
```

5.可以产生TaskService

```java
TaskService taskService = processEngine.getTaskService();
```

6.各个Service的作用

| RepositoryService | 管理流程定义                                 |
| ----------------- | -------------------------------------------- |
| RuntimeService    | 执行管理，包括启动，推进，删除流程实例等操作 |
| TaskService       | 任务管理                                     |
| HistoryService    | 历史管理(执行完的数据的管理)                 |
| IdentityService   | 组织机构管理                                 |
| FormService       | 一个可选服务，任务表单管理                   |
| ManagerService    | 管理器服务                                   |

### 2.RepositoryService

是Activiti的仓库服务类。所谓的仓库指流程定义文档的两个文件，bpmn文件和流程图片。

1.产生方式

```java
RepositoryService repositoryService = processsEngine.getRepositoryService();
```

2.可以产生DeploymentBuilder，用来定义流程部署的相关参数

```java
DeploymentBuilder deploymentBuilder = repositoryService.createDeployment();
```

3.删除流程定义

```java
repositoryService.deleteDeployment(deploymentId);
```

### 3.RuntimeService

是activiti的流程执行服务类。可以从这个服务类中获取很多关于流程执行相关的信息。

### 4.TaskService

是activiti的任务服务类。可以从这个类中获取任务的信息。

### 5.HistoryService

是activiti的查询历史信息的类。在一个流程执行完成后，这个对象为我们提供查询历史信息。

### 6.ProcessDefinition

流程定义类。可以从这里获得资源文件等。

### 7.ProcessInstance

代表流程定义的执行实例。如某个员工请了一天的假，他必须发出一个流程实例的申请。一个流程实例包括所有的运行节点。可以利用这个对象来了解当前流程实例的进度等信息。流程实例就表示一个流程从开始到结束的最大的流程分支，即一个流程中流程实例只有一个。

### 8.Execution

Activiti用这个对象去描述流程执行的每一个节点。在没有并发的情况下，Execution就是同ProcessInstance。流程按照流程定义的规则执行一次的过程，就可以表示执行对象Execution。

![](D:\images\2020-1-28\20200128120301.jpg)

在单线程流程中，如上图的贷款流程，ProcessInstance与Execution是一致的。

这个例子有一个特点，wire money(汇钱)和archive(存档)是并发执行的，这个时候，总线路代表ProcessInstance，而分线路中每个活动代表Execution。

总结：

- 一个流程中，执行对象可以存在多个，但是流程实例只能有一个。
- 当流程按照规则只执行一次的时候，那么流程实例就是执行对象。

## HelloWorld程序(模拟流程图)

我这用的是idea软件，需要安装一个插件[actiBPM](https://plugins.jetbrains.com/plugin/7429-actibpm/versions)，eclipse自己查把，idea 用的舒服，哈哈

### 1.画流程图

#### 1.画流程图

![](D:\images\2020-1-28\20200128123444.png)

#### 2.设置流程的ID和name

![](D:\images\2020-1-28\20200128124215.png)

#### 3.设置任务办理人

![](D:\images\2020-1-28\20200128123710.png)

![](D:\images\2020-1-28\20200128123754.png)

![](D:\images\2020-1-28\20200128123829.png)

#### 3.添加HelloWorld.xml

把刚才的Helloworld.bpmn复制到resources然后把后缀的bpmn修改程xml

![](D:\images\2020-1-28\2020128143310.PNG)

![](D:\images\2020-1-28\2020128143349.PNG)

#### 4.右键xml文件选中，导出图片

![](D:\images\2020-1-28\20200128143738.jpg)

![](D:\images\2020-1-28\20200128143843.jpg)

### 2.工作流流程



#### 1.部署流程定义

```java
  /*得到引擎流程*/
    private ProcessEngine processEngines = ProcessEngines.getDefaultProcessEngine();

    @Test
    public void deployProcess() {
        RepositoryService repositoryService = this.processEngines.getRepositoryService();
        Deployment deploy = repositoryService.createDeployment()
                .name("请假流程001")
                .addClasspathResource("HelloWorld.bpmn")
                .addClasspathResource("HelloWorld.png")
                .deploy();
        System.out.println("部署ID：" + deploy.getId());
        System.out.println("部署名称：" + deploy.getName());
    }
```

查看是否添加成功

```sql
#RepositoryService
SELECT * FROM act_ge_bytearray; #二进制文件表
SELECT * FROM act_re_deployment;#流程部署表
SELECT * FROM act_re_procdef;#流程定义
SELECT * FROM act_ge_property;#工作流的ID算法和版本信息表

#RuntimeService   TaskService
SELECT * FROM act_ru_execution;#流程启动一次只要没有执行完，就会有一条数据
SELECT * FROM act_ru_task;#可能有多条数据
SELECT * FROM act_ru_variable;#记录流程运行时的流程变量
SELECT * FROM act_ru_identitylink;#存放流程办理人的信息

#HistroyService
SELECT * FROM act_hi_procinst;#历史流程实例
SELECT * FROM act_hi_taskinst;#历史任务实例
SELECT * FROM act_hi_actinst;#历史活动节点表
SELECT * FROM act_hi_varinst;#历史流程变量表
SELECT * FROM act_hi_identitylink;##历史办理人表
SELECT * FROM act_hi_comment;#批注表
SELECT * FROM act_hi_attachment;#附件表

#IdentityService
SELECT * FROM act_id_group #角色
SELECT * FROM act_id_membership#用户和角色之间的关系
SELECT * FROM act_id_info#用户的详细信息
SELECT * FROM act_id_user#用户表
```

#### 2.启动流程实例

启动前要修改activiti.cfg.xml，databaseSchemaUpdate属性设置为true，否则删表添加表会找不到数据而报错误

```java
    @Test
    public void startProcess() {
        RuntimeService runtimeService = this.processEngines.getRuntimeService();
        String processDefinitionKey = "HelloWorld";
        runtimeService.startProcessInstanceByKey(processDefinitionKey);
        System.out.println("流程启动成功");
    }
```

#### 3.查看个人任务

```java
    @Test
    public void findTask() {
        TaskService taskService = this.processEngines.getTaskService();
        String assignee = "张三";
        List<Task> list = taskService.createTaskQuery().taskAssignee(assignee).list();
        list.forEach(task -> {
            System.out.println("任务ID：" + task.getId());
            System.out.println("流程ID：" + task.getProcessInstanceId());
            System.out.println("执行ID：" + task.getExecutionId());
            System.out.println("流程定义ID：" + task.getProcessDefinitionId());
            System.out.println("任务名称：" + task.getName());
            System.out.println("任务办理人：" + task.getAssignee());
        });
```

#### 4.个人办理任务

办理任务完成，查看任务是否有信息，如果没有消息就是正在办理， 部门经理正在办

```java
    @Test
    public void completeTask() {
        TaskService taskService = this.processEngines.getTaskService();
        String taskId = "2504";
        taskService.complete(taskId);
        System.out.println("任务完成");
    }
```

#### 5.查看部门经理任务

```java
    @Test
    public void findTask() {
        TaskService taskService = this.processEngines.getTaskService();
        String assignee = "李四";
        List<Task> list = taskService.createTaskQuery().taskAssignee(assignee).list();
        list.forEach((task) -> {
            System.out.println("任务ID：" + task.getId());
            System.out.println("流程ID：" + task.getProcessInstanceId());
            System.out.println("执行ID：" + task.getExecutionId());
            System.out.println("流程定义ID：" + task.getProcessDefinitionId());
            System.out.println("任务名称：" + task.getName());
            System.out.println("任务办理人：" + task.getAssignee());
        });
    }
```

#### 6.部门经理办理任务

```java
@Test
public void completeTask() {
    TaskService taskService = this.processEngines.getTaskService();
    String taskId = "5002";
    taskService.complete(taskId);
    System.out.println("任务完成");
}
```

#### 7.查看总经理任务

```java
	@Test
    public void findTask() {
        TaskService taskService = this.processEngines.getTaskService();
        String assignee = "王五";
        List<Task> list = taskService.createTaskQuery().taskAssignee(assignee).list();
        list.forEach(task -> {
            System.out.println("任务ID：" + task.getId());
            System.out.println("流程ID：" + task.getProcessInstanceId());
            System.out.println("执行ID：" + task.getExecutionId());
            System.out.println("流程定义ID：" + task.getProcessDefinitionId());
            System.out.println("任务名称：" + task.getName());
            System.out.println("任务办理人：" + task.getAssignee());
        });
    }
```

#### 8.总经理办理任务

```java
    @Test
    public void completeTask() {
        TaskService taskService = this.processEngines.getTaskService();
        String taskId = "7502";
        taskService.complete(taskId);
        System.out.println("任务完成");
    }
```

## 管理流程定义

### 1.设置流程定义文档

#### 1.流程图

![](D:\images\2020-1-28\20200128123444.png)

#### 2.bpmn文件

BPMN2.0根节点是definitions节点。这个元素中，可以定义多个流程定义(不过我们建议每个文件只包含一个流程定义，可以简化开发过程中的维护难度)。一个空的流程定义看起来像下面这样。注意definitions元素最少也要包含xmlns和targetNamespace的声明。

targetNamespace可以是任意值，它用来对流程实例进行分类。

说明：流程定义文档有两部分组成：

##### 1.bpmn文件

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" xmlns:tns="http://www.activiti.org/test" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" expressionLanguage="http://www.w3.org/1999/XPath" id="m1580191613310" name="" targetNamespace="http://www.activiti.org/test" typeLanguage="http://www.w3.org/2001/XMLSchema">
  <process id="HelloWorld" isClosed="false" isExecutable="true" name="HelloWorld" processType="None">
    <startEvent id="_2" name="StartEvent"/>
    <endEvent id="_3" name="EndEvent"/>
    <userTask activiti:assignee="张三" activiti:exclusive="true" id="_4" name="提交申请"/>
    <userTask activiti:assignee="李四" activiti:exclusive="true" id="_5" name="审批[部门经理]"/>
    <userTask activiti:assignee="王五" activiti:exclusive="true" id="_6" name="审批[总经理]"/>
    <sequenceFlow id="_7" sourceRef="_2" targetRef="_4"/>
    <sequenceFlow id="_8" sourceRef="_4" targetRef="_5"/>
    <sequenceFlow id="_9" sourceRef="_5" targetRef="_6"/>
    <sequenceFlow id="_10" sourceRef="_6" targetRef="_3"/>
  </process>
  <bpmndi:BPMNDiagram documentation="background=#3C3F41;count=1;horizontalcount=1;orientation=0;width=842.4;height=1195.2;imageableWidth=832.4;imageableHeight=1185.2;imageableX=5.0;imageableY=5.0" id="Diagram-_1" name="New Diagram">
    <bpmndi:BPMNPlane bpmnElement="HelloWorld">
      <bpmndi:BPMNShape bpmnElement="_2" id="Shape-_2">
        <omgdc:Bounds height="32.0" width="32.0" x="380.0" y="20.0"/>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="32.0" width="32.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="_3" id="Shape-_3">
        <omgdc:Bounds height="32.0" width="32.0" x="385.0" y="445.0"/>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="32.0" width="32.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="_4" id="Shape-_4">
        <omgdc:Bounds height="55.0" width="85.0" x="355.0" y="120.0"/>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="55.0" width="85.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="_5" id="Shape-_5">
        <omgdc:Bounds height="55.0" width="85.0" x="355.0" y="210.0"/>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="55.0" width="85.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="_6" id="Shape-_6">
        <omgdc:Bounds height="55.0" width="85.0" x="355.0" y="310.0"/>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="55.0" width="85.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="_7" id="BPMNEdge__7" sourceElement="_2" targetElement="_4">
        <omgdi:waypoint x="396.0" y="52.0"/>
        <omgdi:waypoint x="396.0" y="120.0"/>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="_8" id="BPMNEdge__8" sourceElement="_4" targetElement="_5">
        <omgdi:waypoint x="397.5" y="175.0"/>
        <omgdi:waypoint x="397.5" y="210.0"/>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="_9" id="BPMNEdge__9" sourceElement="_5" targetElement="_6">
        <omgdi:waypoint x="397.5" y="265.0"/>
        <omgdi:waypoint x="397.5" y="310.0"/>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="_10" id="BPMNEdge__10" sourceElement="_6" targetElement="_3">
        <omgdi:waypoint x="401.0" y="365.0"/>
        <omgdi:waypoint x="401.0" y="445.0"/>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>

```

流程规则文件。在部署后，每次系统启动时都回被解析，把内容封装程流程定义放入项目缓存中。Activiti框架结合这个xml文件自动管理流程，流程的执行就是按照bpmn文件定义的规则执行的，bpmn文件是给计算机执行用的。

##### 2.展示流程图的图片在系统里需要展示流程的进展图片，图片是给用户看的。

### 2.部署流程定义(classpath部署)

```java
    private ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();

    @Test
    public void deployProcess_Classpath() {
        /*与流程定义和部署对象相关的Service*/
        RepositoryService repositoryService = this.processEngine.getRepositoryService();
                            /*创建一个部署对象*/
        Deployment deploy = repositoryService.createDeployment()
                /*添加部署的名称*/
                .name("请假流程001")
                /*加载文件*/
                .addClasspathResource("HelloWorld.bpmn")
                .addClasspathResource("HelloWorld.png")
                /*完成部署*/
                .deploy();
        System.out.println("部署ID：" + deploy.getId());
    }
```



1. 先获取流程引擎对象：在创建时回自动加载classpath下的activiti.cfg.xml

2. 首先获得默认的流程引擎，通过流程引擎获取了一个RepositoryService对象(仓库对象)。

   ```java
   ProcessEngine processEngine= ProcessEngines.getDefaultProcessEngine();
   ```

3. 仓库的服务对象产生一个部署对象配置对象，用来封装部署操作的相关配置。

4. 这是一个链式编程，在部署配置对象中设置显示名，上传流程定义规则文件。

5. 像数据库表中存放流程定义规则信息。

6. 这一步在数据库中将操作三张表

   1. act_re_deployment(部署对象表)

      存放流程定义的显示名和部署时间，每部署一次增加一条记录

   2. act_re_procdef(流程定义表)

      存放流程定义的属性信息，部署每个新的流程定义都会在这张表中添加一条记录。

      注意：当流程定义的key相同的情况下，使用的是版本升级

   3. act_ge_bytearray(资源文件表)

      存储流程定义相关的部署信息。即流程定义文档的存放地。每部署一次就回增加两条记录，一条是关于bpmn规则文件，一条是图片(如果部署只指定了bpmn一个文件，activiti回在部署时解析bpmn文件内容自动生成流程图)。两个文件不是很大，都是以二进制形式存储在数据库中。

### 3.部署流程定义(ZIP部署)

把resources里面的HelloWorld.bpmn和HelloWorld.png打包压缩，放到项目的resources，后缀必须是zip。

![](D:\images\2020-1-28\2020128171431.PNG)

```java
    @Test
    public void deployProcess_ZIP() {
        /*不加 / 代表从当前包里找文件，加 / 代表从classpath的根目录里面找文件*/
        InputStream resourceAsStream = getClass().getResourceAsStream("/HelloWorld.zip");
        /*与流程定义和部署对象相关的Service*/
        RepositoryService repositoryService = this.processEngine.getRepositoryService();
        ZipInputStream zipInputStream = new ZipInputStream(resourceAsStream);
        /*创建一个部署对象*/
        Deployment deploy = repositoryService.createDeployment()
                /*添加部署的名称*/
                .name("请假流程001")
                /*加载文件，添加流程图的流*/
                .addZipInputStream(zipInputStream)
                /*完成部署*/
                .deploy();
        System.out.println("部署ID：" + deploy.getId());
    }
```

### 4.查询流程部署

```java
    @Test
    public void queryProcessDeploy() {
        RepositoryService repositoryService = this.processEngine.getRepositoryService();
        /*创建部署信息的查询*/
        List<Deployment> list = repositoryService.createDeploymentQuery()
            
                /*.deploymentId()//根据部署ID去查询
                .deploymentName()//根据部署名称查询
                .deploymentNameLike();//根据部署名称模糊查询
                .orderByDeploymentId().asc()//根据部署ID排序
                .orderByDeploymenTime().desc()//根据部署时间降序
                .orderByDeploymentName()//根据部署名称升序
                .count()//总数
                .listPage()//分页查询返回list集合
                .singleResult()//返回单个对象*/
            
                .list();//查询返回list集合
        list.forEach(deployment -> {
            System.out.println("部署ID:" + deployment.getId());
            System.out.println("部署名称:" + deployment.getName());
            System.out.println("部署时间:" + deployment.getDeploymentTime());
        });
    }
```

### 5.查询流程定义

```java
    @Test
    public void queryProcessDef() {
        RepositoryService repositoryService = this.processEngine.getRepositoryService();
        List<ProcessDefinition> list = repositoryService.createProcessDefinitionQuery()

                /* .deploymentId() //部署对象ID查询
                 .deploymentIds()//根据部署ID的集合查询Set<String> ids
                 .processDefinitionId()//流程定义的ID查询  HelloWorld:1:4
                 .processDefinitionIds()//流程定义的IDS查询
                 .processDefinitionKey()//流程定义的key查询
                 .processDefinitionKeyLike()//流程定义的key模糊查询
                 .processDefinitionName()//流程定义的名称查询
                 .processDefinitionNameLike()//流程定义的名称模糊查询
                 .orderByProcessDefinitionName()//流程名称排序
                 .orderByProcessDefinitionVersion()//版本排序
                 .processDefinitionResourceName()//流程图的BPMN文件名查询
                 .processDefinitionResourceNameLike()//流程图的BPMN文件名模糊查询
                 .processDefinitionVersionGreaterThan()//version>num
                 .processDefinitionVersionGreaterThanOrEquals()//version>=num
                 .processDefinitionVersionLowerThan()//version<num
                 .processDefinitionVersionLowerThanOrEquals()//version<=num
                 .listPage()//分页查询
                 .count()//结果集数量
                 .singleResult()//唯一结果集*/
                
                .list();//集合列表
        list.forEach(processDefinition -> {
            System.out.println("流程定义ID:" + processDefinition.getId());
            System.out.println("流程部署ID:" + processDefinition.getDeploymentId());
            System.out.println("流程定义KEY:" + processDefinition.getKey());
            System.out.println("流程定义的名称:" + processDefinition.getName());
            System.out.println("流程定义的bpmn文件名:" + processDefinition.getResourceName());//bpmn的name
            System.out.println("流程图片名:" + processDefinition.getDiagramResourceName());//png的name
            System.out.println("流程定义的版本号:" + processDefinition.getVersion());
        });
    }
```

### 6.启动流程

```java
    @Test
    public void startProcess() {
        RuntimeService runtimeService = this.processEngine.getRuntimeService();
        runtimeService.startProcessInstanceByKey("HelloWorld");
        System.out.println("流程启动成功");
    }
```

### 7.删除流程定义

```java
	@Test
    public void delProcessDef() {
        RepositoryService repositoryService = this.processEngine.getRepositoryService();
        /*根据流程部署ID删除流程定义 如果当前ID的流程在执行，会报错
        repositoryService.deleteDeployment("2501");*/
        
        /*根据流程部署ID删除流程定义 如果当前ID的流程正在执行，会把正在执行的流程数据删除 act_ru_*和act_hi_*表里面的数据*/
        repositoryService.deleteDeployment("2501", true);
    }
```

### 8.修改流程定义信息

修改流程图之后重新部署只要key不变，它的版本号就会+1

### 9.查询流程图

```java
    @Test
    public void viewProcessImagesById() {
        RepositoryService repositoryService = this.processEngine.getRepositoryService();
        /*根据流程部署ID查询流程定义对象*/
        ProcessDefinition processDefinition = repositoryService.createProcessDefinitionQuery().deploymentId("1").singleResult();
        /*从流程定义对象查询出流程订阅ID*/
        String id = processDefinition.getId();

        InputStream processDiagram = repositoryService.getProcessDiagram(id);
        File file = new File(String.format("d:/%S", processDefinition.getDiagramResourceName()));
        try {
            BufferedOutputStream outputStream = new BufferedOutputStream(new FileOutputStream(file));
            int length = 0;
            byte[] bytes = new byte[1024];
            while ((length = processDiagram.read(bytes)) != -1) {
                outputStream.write(bytes, 0, length);
                outputStream.flush();
            }
            outputStream.close();
            processDiagram.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

### 10.查询最新版本的流程定义

```java
    @Test
    public void queryNewProcessDef() {
        Map<String, ProcessDefinition> processDefinitionHashMap = new HashMap<>();
        /*查询所有的流程定义根据版本号升序*/
        RepositoryService repositoryService = this.processEngine.getRepositoryService();
        List<ProcessDefinition> list = repositoryService.createProcessDefinitionQuery().orderByProcessDefinitionVersion().asc().list();
        /*循环查找的流程定义*/
        list.forEach(processDefinition -> {
            /*添加map集合*/
            processDefinitionHashMap.put(processDefinition.getKey(), processDefinition);
        });
        /*循环map集合*/
        Collection<ProcessDefinition> collection = processDefinitionHashMap.values();
        collection.forEach(processDefinition -> {
            System.out.println("流程定义ID:" + processDefinition.getId());
            System.out.println("流程部署ID:" + processDefinition.getDeploymentId());
            System.out.println("流程定义KEY:" + processDefinition.getKey());
            System.out.println("流程定义的名称:" + processDefinition.getName());
            System.out.println("流程定义的bpmn文件名:" + processDefinition.getResourceName());// bpmn的name
            System.out.println("流程图片名:" + processDefinition.getDiagramResourceName());// png的name
            System.out.println("流程定义的版本号:" + processDefinition.getVersion());
        });
    }
```

### 11.删除流程定义(删除key相同的所有不同版本的流程定义)

```java
@Test
    public void delAllSameVersion() {
        RepositoryService repositoryService = this.processEngine.getRepositoryService();
        /*根据流程定义的Key查询流程集合*/
        List<ProcessDefinition> list = repositoryService.createProcessDefinitionQuery().processDefinitionKey("HelloWorld").list();
        list.forEach(processDefinition -> {
            repositoryService.deleteDeployment(processDefinition.getDeploymentId(), true);
        });
    }
```

## 流程实例，任务的执行

### 1.流程图

![](D:\images\2020-1-28\20200128123444.png)

### 2.部署流程定义

```java
private ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();

    @Test
    public void deployProcess_Classpath() {
        /*与流程定义和部署对象相关的Service*/
        RepositoryService repositoryService = this.processEngine.getRepositoryService();
                            /*创建一个部署对象*/
        Deployment deploy = repositoryService.createDeployment()
                /*添加部署的名称*/
                .name("请假流程001")
                /*加载文件*/
                .addClasspathResource("HelloWorld.bpmn")
                .addClasspathResource("HelloWorld.png")
                /*完成部署*/
                .deploy();
        System.out.println("部署ID：" + deploy.getId());
    }
```

### 3.启动流程实例

```java
    @Test
    public void startProcess() {
        RuntimeService runtimeService = processEngine.getRuntimeService();
        /*
         * 参数1：流程定义ID
         * 参数2：Map<String,Object> 流程变量
         * */
        //runtimeService.startProcessInstanceById(processDefinitionId,variables);

        /*
         * 参数1：流程定义ID
         * 参数2：String 业务ID 把业务ID和流程执行实例进行绑定
         * */
        //runtimeService.startProcessInstanceById(processDefinitionId,businessKey);

        /*
         * 参数1：流程定义ID
         * 参数2：String 业务ID 把业务ID和流程执行实例进行绑定
         * 参数3：Mao<String,Object>流程变量
         * */
        //runtimeService.startProcessInstanceById(processDefinitionId,businessKey,variables);

        /*实例开发中使用的*/

        //runtimeService.startProcessInstanceByKey(processDefinitionId,businessKey,variables);
        //runtimeService.startProcessInstanceByKey(processDefinitionId,businessKey);

        ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("HelloWorld");
        System.out.println(processInstance.getId());
        System.out.println("启动成功");
    }
```

- 操作数据库的act_ru_execution表，如果是用户任务节点，同时也会在act_ru_task添加一条记录

### 4.查询我的个人任务

​	数据表：act_ru_task

```java
    @Test
    public void queryProcess() {
        TaskService taskService = this.processEngine.getTaskService();
        List<Task> list = taskService.createTaskQuery()
                .taskAssignee("张三")//任务办理人查询任务

                /*   .deploymentId()//部署ID查询
                     .deploymentIdIn()//根据部署ID集合查询 where id in (1,2,3,4)
                     .executionId()//执行实例ID
                     .processDefinitionId()//流程定义ID
                     .processDefinitionKey()//流程定义的Key
                     .processDefinitionKeyIn()
                     .processDefinitionKeyLike()
                     .processDefinitionName()
                     .processDefinitionNameLike()
                     .processInstanceBusinessKey()
                     .orderByTaskCreateTime().desc()
                     .listPage()
                     .count()
                     .singleResult()*/

                .list();
        list.forEach(task -> {
            System.out.println("任务ID:" + task.getId());
            System.out.println("任务办理人:" + task.getAssignee());
            System.out.println("执行实例ID:" + task.getExecutionId());
            System.out.println("任务名称:" + task.getName());
            System.out.println("流程定义ID:" + task.getProcessDefinitionId());
            System.out.println("流程实例ID:" + task.getProcessInstanceId());
            System.out.println("任务创建时间:" + task.getCreateTime());
        });
    }
```

说明：

1. ​	因为是任务查询，所以从processEngine中应该得到TaskService。
2. ​	使用TaskService获取到任务查询对象TaskQuery。
3. ​	为查询对象添加查询过滤条件，使用taskAssignee指定任务的办理者(即查询指定用户的代办任务)，同时可以添加分页排序等过滤条件。
4. ​	调用list方法执行查询，返回办理者为指定用户的任务列表。
5. ​	任务ID，名称，办理人，创建时间可以从act_ru_task表中查到。
6. ​	在这种情况下，ProcessInstance相当于Execution。
7. ​	如果assignee属性为部门经理，结果为空。因为现在流程只到了"填写请假申请"阶段，后面的任务还没有执行，即在数据库中没有部门经理可以办理的任务，所以查询不到。
8. ​	一个Task节点和Execution节点是1对1的情况下，在task对象中使用Execution_来表示他们之间的关系。
9. ​	任务ID在数据库表act_ru_task中对应"ID_"列。

附加：

​	在activiti任务中，主要分为两大类查询任务(个人任务和组任务)

1. 确切值定理办理者的任务，这个任务将成为指定者的私有任务，即个人任务。
2. 无法指定具体的某一个人来办理的任务，可以把任务分配给几个人或者一道多个小组，让这个范围内的用户可以选择性(如有空余时间时)来办理这类任务，即组任务。

### 5.办理任务

```java
    @Test
    public void completeTask() {
        TaskService taskService = this.processEngine.getTaskService();
        /*根据任务ID去完成*/
        taskService.complete("2504");
        /*根据任务ID去完成任务并制定流程变量*/
        //taskService.complete(taskId,variables);
    }
```

说明：

1. 是办理任务，所以从ProcessEngine得到的是TaksService。
2. 当执行完这段代码，再以员工的身份去执行查询的时候，会发现这个时候已经没有数据了，因为正在执行的任务中没有数据。
3. 对于执行完的任务，activiti将从act_ru_task表中删除该任务，下一个任务会被插入进来。
4. 以"部门经理"的身份进行查询，可以查到结果。因为流程执行到部门经理审批这个节点了。
5. 在执行办理任务代码，执行完以后以"部门经理"身份进行查询，没有结果。
6. 重复第3步和第四步直到流程执行完。

### 6.判断流程是否结束

```java
    @Test
    public void isComplete() {
        RuntimeService runtimeService = this.processEngine.getRuntimeService();
        ProcessInstance processInstance = runtimeService.createProcessInstanceQuery().processInstanceId("2501").singleResult();
        /*if (null != processInstance) {
            System.out.println("流程未结束");
        } else {
            System.out.println("流程已结束");
        }*/

        /*已知任务ID 5002*/
        String taskId = "5002";

        /*根据任务ID查询任务实例对象*/
        TaskService taskService = this.processEngine.getTaskService();
        Task task = taskService.createTaskQuery().taskId(taskId).singleResult();
        /*任务实例里面取出流程实例ID*/
        String processInstanceId = task.getProcessInstanceId();
        /*使用流程实例ID去流程实例表查询有没有数据*/
        ProcessInstanceQuery processInstanceQuery = runtimeService.createProcessInstanceQuery().processInstanceId(processInstanceId);
        if (null != processInstanceQuery) {
            System.out.println("流程未结束");
        } else {
            System.out.println("流程已结束");
        }
    }
```

在流程执行的过程中，创建的流程实例ID在整个过程中都不会变，当流程结束后，流程实例将会正在执行的执行对象中(act_ru_execution)被删除

说明：

1. 因为是查询流程实例，所以先获取RuntimeService。
2. 创建流程实例查询对象，设置实例ID过滤参数。
3. 由于一个流程实例ID只对应一个实例，使用singleResult执行查询返回一个唯一的结果，如果结果数量大于1，抛出异常。
4. 判断指定ID的实例是否存在，如果结果为空，则代表流程结束，实例在正在执行的执行对象表中已被删除，转换成历史数据。

### 7.查询正在执行的流程实例(act_ru_execution)

```java
	@Test
    public void queryProcessInstance() {
        RuntimeService runtimeService = this.processEngine.getRuntimeService();
        List<ProcessInstance> list = runtimeService.createProcessInstanceQuery().list();
        list.forEach(processInstance -> {
            System.out.println("执行实例ID：" + processInstance.getId());
            System.out.println("流程定义ID：" + processInstance.getProcessDefinitionId());
            System.out.println("流程实例ID：" + processInstance.getProcessInstanceId());
        });
    }
```

### 8.查询历史任务

```java
    @Test
    public void queryHistoryTask() {
        HistoryService historyService = this.processEngine.getHistoryService();
        List<HistoricTaskInstance> list = historyService.createHistoricTaskInstanceQuery().list();
        list.forEach(historicTaskInstance -> {
            System.out.println("任务ID:" + historicTaskInstance.getId());
            System.out.println("任务办理人:" + historicTaskInstance.getAssignee());
            System.out.println("执行实例ID:" + historicTaskInstance.getExecutionId());
            System.out.println("任务名称:" + historicTaskInstance.getName());
            System.out.println("流程定义ID:" + historicTaskInstance.getProcessDefinitionId());
            System.out.println("流程实例ID:" + historicTaskInstance.getProcessInstanceId());
            System.out.println("任务创建时间:" + historicTaskInstance.getCreateTime());
            System.out.println("任务结束时间:" + historicTaskInstance.getEndTime());
            System.out.println("任务持续时间：" + historicTaskInstance.getDurationInMillis());
        });
    }
```

### 9.查询历史流程实例

```java
    @Test
    public void queryHistoryProcessInstance() {
        HistoryService historyService = this.processEngine.getHistoryService();
        List<HistoricProcessInstance> list = historyService.createHistoricProcessInstanceQuery().list();
        list.forEach(historicProcessInstance -> {
            System.out.println("执行实例ID：" + historicProcessInstance.getId());
            System.out.println("流程定义ID：" + historicProcessInstance.getProcessDefinitionId());
            System.out.println("流程启动时间：" + historicProcessInstance.getStartTime());
        });
    }
```

### 10.总结

Execution 执行对象

按流程定义的规则执行一次的过程

对应的表：

- ​		act_ru_execution：正在执行的信息
- ​		act_hi_procinst：已经执行完的历史流程实例信息
- ​		act_hi_actinst：存放历史所有完成的活动

ProcessInstance 流程实例

特指流程从开始到结束的那个最大的执行分支，一个执行的流程中，流程实例只有1个。

注意：

1. 如果是单列流程，执行对象ID就是流程实例ID
2. 如果一个流程有分支和聚合，那么执行对象ID和流程实例ID就不相同
3. 一个流程中，流程实例只有1个，执行对象可以存在多个。

Taks 任务

执行到某任务环节时生成的任务信息。

对应的表：

- ​	act_ru_task：正在执行的任务信息
- ​	act_hi_taskinst：已经执行完的历史任务信息

相关ID总结

- 部署ID——act_re_deployment id
- 流程定义ID——act_re_procdef id
- 流程实例ID——act_ru_execution id
- 执行实例ID——act_ru_task execution id
- 任务ID——act_run_task id

## 流程变量

### 1.流程图

![](D:\images\2020-1-28\20200128123444.png)

流程变量在整个工作流中扮演很重要的作用。例如：请假流程中有请假天数，请假原因等一写参数都为流程变量的范围。流程变量的作用域范围是只对应一个流程实例。也就是说各个流程实例变量是不相互邮箱的。流程实例结束完成以后流程变量还保存在数据库中(存放到流程变量的历史表中)。

### 2.部署流程定义

```java
private ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();

    @Test
    public void deployProcess_Classpath() {
        /*与流程定义和部署对象相关的Service*/
        RepositoryService repositoryService = this.processEngine.getRepositoryService();
                            /*创建一个部署对象*/
        Deployment deploy = repositoryService.createDeployment()
                /*添加部署的名称*/
                .name("请假流程001")
                /*加载文件*/
                .addClasspathResource("HelloWorld.bpmn")
                .addClasspathResource("HelloWorld.png")
                /*完成部署*/
                .deploy();
        System.out.println("部署ID：" + deploy.getId());
    }
```

### 3.启动流程实例

```java
    @Test
    public void startProcess() {
        RuntimeService runtimeService = this.processEngine.getRuntimeService();
		/*创建流程变量对象*/
 	    Map<String, Object> map = new HashMap<>();
        map.put("请假天数", 5);
        map.put("请假原因", "出去玩");
        map.put("请假时间", new Date());
        ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("HelloWorld", map);
        System.out.println(processInstance.getId() + " " + processInstance.getProcessDefinitionId());
    }
```

### 4.设置流程变量

RuntimeService：

​		数据表：act_ru_execution

```java
    @Test
    public void setVariable() {
        RuntimeService runtimeService = this.processEngine.getRuntimeService();
        String executionId = "2501";
        //runtimeService.setVariable(executionId, "请假人", "小明");
        Map<String, Object> map = new HashMap<>();
        map.put("请假天数", 1);
        map.put("请假原因", "出去玩2");
        map.put("请假时间", new Date());
        map.put("object", new User(1, "张三"));
        runtimeService.setVariables(executionId, map);
        System.out.println("流程设置成功");
    }
```

TaskService：

​		数据表：act_ru_task

```java
    @Test
    public void setVariable() {
        TaskService taskService = this.processEngine.getTaskService();
        String taskId = "2509";
        Map<String, Object> map = new HashMap<>();
        map.put("任务ID", 9);
        taskService.setVariables(taskId, map);
        System.out.println("流程设置成功");
    }
```



说明：

1. 流程变量的作用域就是流程实例，只要设置就行了，不用管在那个阶段设置
2. 基本类型设置流程变量，在taskService中使用任务ID，定义流程变量的名称，设置流程变量的值。
3. JavaBean类型设置流程变量，需要这个JavaBean实现了Serializable接口
4. 设置流程变量的时候，向act_ru_variable这个表添加数据

### 5.获取流程变量

​		数据表：act_ru_variable

```java
    @Test
    public void getVariable() {
        RuntimeService runtimeService = this.processEngine.getRuntimeService();
        String executionId = "2501";
        Date date = (Date) runtimeService.getVariable(executionId, "请假时间");
        String simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd").format(date);
        System.out.println(simpleDateFormat);
        Object variable = runtimeService.getVariable(executionId, "请假天数");
        User user = (User) runtimeService.getVariable(executionId, "object");
        System.out.println(variable);
        System.out.println(user);
    }
```

说明：

1. ​	流程变量的获取针对流程实例(即一个流程)，每个流程实例获取的流程变量时不同的。
2. ​	使用基本类型获取流程变量，在taskService中使用任务ID，流程变量的名称，获取流程变量的值。
3. ​	JavaBean类型设置获取流程变量，除了需要这个JavaBean实现了Serializable接口外，还要求流程变量对象的属性不能发生变化，否则抛出异常。要固定化序列化的ID。

```java
private static final long serialVersionUID = 1L;
```

### 6.查询历史的流程变量

​	数据表：act_hi_varinst

```java
    @Test
    public void getHistoryVariable() {
        HistoryService historyService = this.processEngine.getHistoryService();

      /*  HistoricVariableInstance historicVariableInstance = historyService.createHistoricVariableInstanceQuery().id("2504").singleResult();
        System.out.println(historicVariableInstance.getId());
        System.out.println(historicVariableInstance.getProcessInstanceId());
        System.out.println(historicVariableInstance.getVariableName());
        System.out.println(historicVariableInstance.getValue());*/

        String processInstanceId = "2501";
        List<HistoricVariableInstance> list = historyService.createHistoricVariableInstanceQuery().processInstanceId(processInstanceId).list();
        list.forEach(historicVariableInstance -> {
            System.out.println("ID：" + historicVariableInstance.getId());
            System.out.println("流程ID：" + historicVariableInstance.getProcessInstanceId());
            System.out.println("变量名：" + historicVariableInstance.getVariableName());
            System.out.println("变量类型" + historicVariableInstance.getVariableTypeName());
            System.out.println("变量值：" + historicVariableInstance.getValue());
            System.out.println();
        });
    }
```

### 7.流程变量支持的类型

![](D:\images\2020-1-28\2020129124303.png)

### 8.总结

#### 1.流程变量

在流程执行或者任务执行的过程中，用于设置和获取变量，使用流程变量在流程传递的过程中传递业务参数。

对应的表：

1. act_ru_avriable：正在执行的流程变量表
2. act_hi_varinst：流程变量历史表

#### 2.扩展知识：setVariable 和 setVariableLocal 的区别

setVariable：设置流程变量的时候，流程变量名称相同的时候，后一次的值替换前一次的值，而且可以看到TASK_ID的字段不会存放任务的ID的值

setVariableLocal：

1. 设置流程变量的时候，针对当前活动的节点设置流程变量，如果一个流程中存在2个活动节点，对每个活动节点都设置流程变量，即使流程变量的名称相同，后一次的版本的值也不会替换前一次版本的值，它会使用不同的任务ID作为标志，存放2个流程变量值，而且可以看到TASK_ID的字段会存放任务ID的值，例如act_hi_varinst表的数据：不同的任务节点，即使流程变量名称相同，存放的值也是不同的。
2. 还用使用serVariableLocal说明流程变量绑定了当前的任务，当流程继续执行时，下个任务获取不到这个流程变量(因为正在执行的流程变量重没有这个数据)，所有查询正在执行的任务时不能查询到我们需要的数据，此时需要查询历史的流程变量。

## 流程执行历史记录

### 1.查询历史流程实例

​		查找按照某个流程定义的规则一共执行了多少次流程

查询所有
		数据表：act_hi_procinst

```java
    @Test
    public void historyProcess() {
        HistoryService historyService = processEngine.getHistoryService();
        List<HistoricProcessInstance> list = historyService.createHistoricProcessInstanceQuery()

                /* .processDefinitionId()
                 .processDefinitionKey()
                 .processDefinitionKeyIn()
                 .processDefinitionName()
                 .processDefinitionVersion()
                 .processInstanceBusinessKey()
                 .processInstanceId()
                 .processInstanceIds()
                 .orderByProcessDefinitionId()
                 .orderByProcessInstanceBusinessKey()*/

                .list();
        list.forEach(historicProcessInstance -> {
            System.out.println("历史流程实例ID：" + historicProcessInstance.getId());
            System.out.println("流程定义ID：" + historicProcessInstance.getProcessDefinitionId());
            System.out.println("历史流程实例业务ID：" + historicProcessInstance.getBusinessKey());
            System.out.println("流程部署ID：" + historicProcessInstance.getDeploymentId());
            System.out.println("流程订阅Key：" + historicProcessInstance.getProcessDefinitionKey());
            System.out.println("开始活动ID：" + historicProcessInstance.getStartActivityId());
            System.out.println("结束活动ID：" + historicProcessInstance.getEndActivityId());
        });
    }
```

查询单个

```java
    @Test
    public void historyProcessInstanceSingle() {
        String processInstanceId = "2501";
        HistoricProcessInstance historicProcessInstance = this.historyService.createHistoricProcessInstanceQuery()
                .processInstanceId(processInstanceId)
                .orderByProcessInstanceStartTime().asc()
                .singleResult();

        System.out.println("历史流程实例ID：" + historicProcessInstance.getId());
        System.out.println("流程定义ID：" + historicProcessInstance.getProcessDefinitionId());
        System.out.println("历史流程实例业务ID：" + historicProcessInstance.getBusinessKey());
        System.out.println("流程部署ID：" + historicProcessInstance.getDeploymentId());
        System.out.println("流程订阅Key：" + historicProcessInstance.getProcessDefinitionKey());
        System.out.println("开始活动ID：" + historicProcessInstance.getStartActivityId());
        System.out.println("结束活动ID：" + historicProcessInstance.getEndActivityId());
    }
```



### 2.查询历史活动

​	数据表：act_hi_actinst

```java
    @Test
    public void queryHistoryAct() {
        List<HistoricActivityInstance> list = this.historyService.createHistoricActivityInstanceQuery()
                /* .activityId()
                 .activityInstanceId()
                 .activityName()
                 .orderByActivityId()
                 .orderByActivityName()*/
                .list();

        list.forEach(historicActivityInstance -> {
            System.out.println("ID：" + historicActivityInstance.getId());
            System.out.println("流程定义ID：" + historicActivityInstance.getProcessDefinitionId());
            System.out.println("流程实例ID：" + historicActivityInstance.getProcessInstanceId());
            System.out.println("执行实例ID：" + historicActivityInstance.getExecutionId());
            System.out.println("活动ID：" + historicActivityInstance.getActivityId());
            System.out.println("任务ID：" + historicActivityInstance.getTaskId());
            System.out.println("活动名称：" + historicActivityInstance.getActivityName());
            System.out.println("活动类型：" + historicActivityInstance.getActivityType());
            System.out.println("任务办理人：" + historicActivityInstance.getAssignee());
            System.out.println("开始时间：" + historicActivityInstance.getStartTime());
            System.out.println("结束时间：" + historicActivityInstance.getEndTime());
            System.out.println("持续时间：" + historicActivityInstance.getDurationInMillis());
        });
    }
```

### 3.查询历史任务

某一次流程的执行一共经历了多少个任务

​	数据表：act_hi_taskinst

```java
    @Test
    public void queryHistoryTask() {
        List<HistoricTaskInstance> list = this.historyService.createHistoricTaskInstanceQuery()

                /*.deploymentId()
                .deploymentId()
                .executionId()
                .processDefinitionId()
                .processDefinitionKey()
                .processDefinitionKeyIn()
                .processDefinitionKeyLike() //processDefinitionKeyLike="%HelloWorld"
                .processDefinitionName()
                .processDefinitionNameLike()
                .orderByTaskDefinitionKey()
                .singleResult()
                .listPage()
                .count()*/

                .list();
        list.forEach(historicTaskInstance -> {
            System.out.println("任务ID:" + historicTaskInstance.getId());
            System.out.println("任务办理人:" + historicTaskInstance.getAssignee());
            System.out.println("执行实例ID:" + historicTaskInstance.getExecutionId());
            System.out.println("任务名称:" + historicTaskInstance.getName());
            System.out.println("流程定义ID:" + historicTaskInstance.getProcessDefinitionId());
            System.out.println("流程实例ID:" + historicTaskInstance.getProcessInstanceId());
            System.out.println("任务创建时间:" + historicTaskInstance.getCreateTime());
        });
    }
```

### 4.查询历史流程变量

某一次流程的执行一共设置的流程变量

​	数据表：act_hi_varinst

```java
    @Test
    public void historyProcessVariables() {
        String processInstanceId = "2501";
        List<HistoricVariableInstance> list = this.historyService.createHistoricVariableInstanceQuery()
                .processInstanceId(processInstanceId)
                .list();

        list.forEach(historicVariableInstance -> {
            System.out.println("V-ID" + historicVariableInstance.getId());
            System.out.println("流程实例ID" + historicVariableInstance.getProcessInstanceId());
            System.out.println("变量名称" + historicVariableInstance.getVariableName());
            System.out.println("变量类型名称" + historicVariableInstance.getVariableTypeName());
            System.out.println("变量值" + historicVariableInstance.getValue());
            System.out.println();
        });
    }
```

### 5.总结

由于数据库重保存着历史信息以及正在运行的流程实例信息，在实际项目重对已完成任务的查看频率远不及对代办和可接任务的查看，所以在activiti采用分开管理，把正在运行的交给RuntimeService。

TaskService管理，而历史数据交给HistoryService来管理。这样做的好处在于，加快流程执行的速度，因为正在执行的流程的表重数据不会很大。

## 连线

使用流程变量去控制流程的走向

### 1.流程图

![](D:\images\2020-1-28\2020129155443.PNG)

提交申请节点

![](D:\images\2020-1-28\20200129160628.png)	

部门经理审批节点

![](D:\images\2020-1-28\20200129160739.png)

总经理审批节点

![](D:\images\2020-1-28\20200129160846.png)

![](D:\images\2020-1-28\20200129162917.PNG)

![](D:\images\2020-1-28\20200129163004.PNG)

### 2.部署流程定义+启动流程实例

```java
    private ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();

    /**
     * 部署流程定义
     */
    @Test
    public void deployProcess() {
        RepositoryService repositoryService = processEngine.getRepositoryService();
        Deployment deploy = repositoryService.createDeployment().name("报销流程")
                .addClasspathResource("SequenceFlow.bpmn")
                .addClasspathResource("SequenceFlow.png").deploy();

        System.out.println("部署ID：" + deploy.getId());
    }

    /**
     * 启动流程实例
     */
    @Test
    public void startProcess() {
        RuntimeService runtimeService = this.processEngine.getRuntimeService();
        ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("sequenceflow");
        System.out.println("启动ID：" + processInstance.getId() + " 启动名称：" + processInstance.getName());
    }
```

### 3.查看我的个人任务

```java
    @Test
    public void queryTask() {
        String assignee = "孙七";
        List<Task> list = this.processEngine.getTaskService()
                .createTaskQuery().taskAssignee(assignee).list();
        list.forEach(task -> {
            System.out.println("任务ID:" + task.getId());
            System.out.println("任务名称:" + task.getName());
            System.out.println("任务的创建时间:" + task.getCreateTime());
            System.out.println("任务的办理人:" + task.getAssignee());
            System.out.println("流程实例ID：" + task.getProcessInstanceId());
            System.out.println("执行对象ID:" + task.getExecutionId());
            System.out.println("流程定义ID:" + task.getProcessDefinitionId());
        });
    }
```

### 4.办理任务

```java
    @Test
    public void completeTask(){
        TaskService taskService = this.processEngine.getTaskService();
        /*"2504"是act_ru_task表的主键*/
        taskService.complete("2504");
        System.out.println("办理成功");
    }
```

### 5.办理任务，使用流程变量指定流程走向

```java
    @Test
    public void completeTask2() {
        TaskService taskService = this.processEngine.getTaskService();
        String taskId = "5002";
        Map<String, Object> map = new HashMap<>();
        map.put("outcome", "不重要");
        taskService.complete(taskId, map);
        System.out.println("办理成功");
    }
```

说明：

使用流程变量，设置连线需要的流程变量的名称outcome，并设置流程变量的值对应

![](D:\images\2020-1-28\20200129163004.PNG)

![20200129162917](D:\images\2020-1-28\20200129162917.PNG)

流程会按照指定的连线完成任务。

### 6.总结

#### 1.一个活动重可以指定一个或多个SequenceFlow(Start重有一个，End中没有)。

- 开始活动中有一个SequenceFlow.
- 结束活动中没有SequenceFlow.
- 其他活动中有一条或者多条SequenceFlow.

#### 2.如果只有一个，则可以不使用流程变量设置condition的名称；

如果有多个，则需要使用流程变量设置codition的名称。outcome表示流程变量的名称，"不重要"表示流程变量的值，${}中间的内容要使用boolean类型的表达式，用来判断应该执行的连线。

## 排它网关(ExclusiveGateWay)

### 1.流程图

![](D:\images\2020-1-28\20200129184408.PNG)

![](D:\images\2020-1-28\20200129184649.PNG)

![20200129184635](D:\images\2020-1-28\20200129184635.PNG)

![20200129184622](D:\images\2020-1-28\20200129184622.PNG)

![20200129184552](D:\images\2020-1-28\20200129184552.PNG)

![20200129184535](D:\images\2020-1-28\20200129184535.PNG)

![20200129184521](D:\images\2020-1-28\20200129184521.PNG)

![20200129184503](D:\images\2020-1-28\20200129184503.PNG)

### 2.部署流程定义+启动流程实例

```java
    @Test
    public void deployProcess() {
        RepositoryService repositoryService = processEngine.getRepositoryService();
        Deployment deploy = repositoryService.createDeployment()
                .name("报销流程")
                .addClasspathResource("ExclusiveGateWay.bpmn")
                .addClasspathResource("ExclusiveGateWay.png")
                .deploy();
        System.out.println("部署ID：" + deploy.getId());
    }

    @Test
    public void startProcess(){
        RuntimeService runtimeService = processEngine.getRuntimeService();
        ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("myProcess");
        System.out.println("启动ID：" + processInstance.getId() + " 启动名称：" + processInstance.getName());
    }
```

### 3.查看我的个人任务

```java
    @Test
    public void queryTask() {
        TaskService taskService = processEngine.getTaskService();
        List<Task> list = taskService.createTaskQuery().taskAssignee("张三").list();
        list.forEach(task -> {
            System.out.println("任务ID:" + task.getId());
            System.out.println("任务名称:" + task.getName());
            System.out.println("任务的创建时间:" + task.getCreateTime());
            System.out.println("任务的办理人:" + task.getAssignee());
            System.out.println("流程实例ID：" + task.getProcessInstanceId());
            System.out.println("执行对象ID:" + task.getExecutionId());
            System.out.println("流程定义ID:" + task.getProcessDefinitionId());
        });
    }
```

### 4.办理任务

```java
    @Test
    public void completeTask(){
        TaskService taskService = this.processEngine.getTaskService();
        /*"2504"是act_ru_task表的主键*/
        taskService.complete("2504");
        System.out.println("办理成功");
    }
```



### 5.办理任务(使用流程变量指定流程走向)

```java
    @Test
    public void complete() {
        TaskService taskService = processEngine.getTaskService();
        String taskId = "2504";
        Map<String, Object> map = new HashMap<>();
        map.put("money", 400);
        taskService.complete(taskId, map);
        System.out.println("办理成功");
    }
```

### 6.说明

1. 一个排它网关对应一个以上的顺序流
2. 由排它网关流出的顺序流都有个conditionExpression元素，在内部维护返回boolean类型的决策结果。
3. 决策网关指挥返回一条结果。当流程执行到排它网关时，流程引擎回自动检索网关出口，从上到下检索如果发现第一条决策结果为true或者没有设置条件的(默认为成立)，则流出。
4. 如果没有任何一个出口符合条件，则排出异常。
5. 使用流程变量，设置连线的条件，并按照连线的条件执行工作流，如果没有条件符合的条件，则以默认的连接离开。

## 并行网关(parallelGateWay)

### 1.流程图

![](D:\images\2020-1-28\20200129220950.PNG)

![](D:\images\2020-1-28\20200129221421.PNG)

![20200129221402](D:\images\2020-1-28\20200129221402.PNG)

以上流程图中，分两个用户 [商家和买家]

卖家付款 商家发货 商家收款 买家收货

### 2.部署流程定义+启动流程实例

```java
    private ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();

    @Test
    public void deployProcess() {
        RepositoryService repositoryService = processEngine.getRepositoryService();
        Deployment deploy = repositoryService.createDeployment()
                .name("淘宝流程")
                .addClasspathResource("ParallelGateWay.bpmn")
                .addClasspathResource("ParallelGateWay.png")
                .deploy();
        System.out.println("部署ID：" + deploy.getId());
    }

    @Test
    public void test() {
        RuntimeService runtimeService = processEngine.getRuntimeService();
        ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("myProcess_1");
  System.out.println("流程实例ID：" + processInstance.getId() + " 启动名称：" + processInstance.getName());
    }
```

### 3.查看我的个人任务

```java
    @Test
    public void queryTask() {
        TaskService taskService = processEngine.getTaskService();
        List<Task> list = taskService.createTaskQuery().taskAssignee("买家").list();
        list.forEach(task -> {
            System.out.println("任务ID:" + task.getId());
            System.out.println("任务名称:" + task.getName());
            System.out.println("任务的创建时间:" + task.getCreateTime());
            System.out.println("任务的办理人:" + task.getAssignee());
            System.out.println("流程实例ID：" + task.getProcessInstanceId());
            System.out.println("执行对象ID:" + task.getExecutionId());
            System.out.println("流程定义ID:" + task.getProcessDefinitionId());
        });
    }
```

### 4.办理任务

```java
    @Test
    public void completeTask(){
        TaskService taskService = this.processEngine.getTaskService();
       	String taskId = "2504";
        taskService.complete(taskId);
        System.out.println("办理成功");
    }
```

### 5.说明

1. 一个流程中流程实例只有一个，执行对象有多个。
2. 并行网关的功能是基于进入和外出的顺序流的，分支(fork)，并行后的所有外出顺序流，为每个顺序流都创建一个并发分支。汇聚(join)，所有到达并行网关，在此等待的进入分支，直到所有进入顺序流的分支都到达以后，流程就会同归汇聚网关。
3. 并行网关的进入和外出都是使用相同节点标识。
4. 如果同一个并行网关有多个进入和多个外出顺序流，它就同时具有分支和汇聚功能。这时，网关会线汇聚所有进入的顺序流，然后再切分成多个并行分支。
5. 并行网关不会解析条件。即使顺序中定义了条件，也会被忽略。
6. 并行网关不需要是"平衡的"(比如，对应并行网关的进入和外出的节点数目不一定相等)。

## 接收任务(receiveTask，即等待活动)

接受活动(receive Task，即等待活动)

接受任务是一个简单的任务，它会等待对应消息的到达。当前官方只实现了这个任务的java语义。当流程达到接受任务，流程状态会保存到数据库中。

在任务创建后，意味这流程会进入等待状态，知道引擎接受了一个特定的消息，这回触发流程穿过接受任务继续执行。

### 1.流程图

![](D:\images\2020-1-28\20200130110734.PNG)

### 2.部署流程定义

```java
    @Test
    public void deployProcess() {
        RepositoryService repositoryService = processEngine.getRepositoryService();
        Deployment deploy = repositoryService.createDeployment()
                .name("汇总流程")
                .addClasspathResource("ReceiveTask.bpmn")
                .addClasspathResource("ReceiveTask.png")
                .deploy();
        System.out.println("部署ID：" + deploy.getId());
    }
```

### 3.启动流程实例

```java
    @Test
    public void startProcess() {
        RuntimeService runtimeService = processEngine.getRuntimeService();
        ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("myProcess");
        System.out.println("流程实例ID：" + processInstance.getId() + " 启动名称：" + processInstance.getName());
    }
```

### 4.汇总金额

```java
    @Test
    public void executionTask() {
        RuntimeService runtimeService = processEngine.getRuntimeService();
        String processInstanceId = "2501";
        /*查询执行对象ID*/
        Execution execution = runtimeService
                .createExecutionQuery() // 创建执行对象查询
                .processInstanceId(processInstanceId)//使用流程实例ID查询
                .activityId("_7")   //当前活动的id，对应receiveTask.bpmn文件中的活动节点id的属性值
                .singleResult();

        System.out.println("执行实例ID：" + execution.getId());
        System.out.println("流程实例ID：" + execution.getProcessInstanceId());
        System.out.println("活动ID：" + execution.getActivityId());


        /*使用流程变量设置当前销售额，用来传递业务参数*/
        /*去查询数据库，进行汇总，耗时操作*/
        int value = 10000;
        runtimeService.setVariable(execution.getId(), "当前的销售额", value);
        /*向后执行一步，如果流程处于等待状态，使得流程继续执行*/
        runtimeService.signal(execution.getId());
    }
```

### 5.发送消息

```java
    @Test
    public void sendMsg() {
        RuntimeService runtimeService = processEngine.getRuntimeService();
        String executionId = "2501";
        /*从流程变量中获取汇总当日销售额的值*/
        Integer value = (Integer) runtimeService.getVariable(executionId, "当前的销售额");
        System.out.println(value);
        System.out.println("发送短信");
        boolean flag = false;
        int num = 0;
        do {
            flag = true;
            num++;
            if (num == 10) {
                System.out.println("尝试10次发送，全部失败，终止发送");
                break;
            }
        } while (!flag);

        /*向后执行一步，如果流程处于等待状态，使得流程继续执行*/
        runtimeService.signal(executionId);
        System.out.println("流程执行完成");
    }
```



### 6.启动流程实例(启动流程+汇总金额，发送消息)

```java
    @Test
    public void doTask() {
        RuntimeService runtimeService = processEngine.getRuntimeService();

        String processDefinitionKey = "myProcess";
        ProcessInstance processInstance = runtimeService.startProcessInstanceByKey(processDefinitionKey);
        System.out.println("流程启动成功，流程实例ID：" + processInstance.getId() + " " + processInstance.getProcessDefinitionId());

        Execution execution = runtimeService
                .createExecutionQuery() // 创建执行对象查询
                .processInstanceId(processInstance.getId())//使用流程实例ID查询
                .activityId("_7")   //当前活动的id，对应receiveTask.bpmn文件中的活动节点id的属性值
                .singleResult();

        System.out.println("执行实例ID：" + execution.getId());
        System.out.println("流程实例ID：" + execution.getProcessInstanceId());
        System.out.println("活动ID：" + execution.getActivityId());

        /*汇总当日的销售金额*/
        /*使用流程变量设置当日销售额，用来传递有业务参数*/
        int money = 0;
        int num = 0;
        while (true) {
            num++;
            try {
                /*查询数据库，进行汇总*/
                money = summaryMoney();
                break;
            } catch (Exception e) {
                e.printStackTrace();
                if (num == 10) {
                    System.out.println("尝试10次发送，全部失败，终止发送");
                    break;
                }
            }
        }

        /*设置流程变量*/
        runtimeService.setVariable(processInstance.getId(), "当前的销售金额", money);
        /*向后执行一步，如果流程处于等待状态，是的流程继续执行*/
        runtimeService.signal(execution.getId());

        /*获取当日的销售金额*/
        Integer values = (Integer) runtimeService.getVariable(processInstance.getId(), "当前的销售金额");
        /*发送短信*/
        System.out.println("发送短信：金额是：" + values);
        /*向后执行一步，如果流程处于等待状态，是的流程继续执行*/
        runtimeService.signal(execution.getId());
    }

    public Integer summaryMoney() {
        /*查询数据库*/
        System.out.println("金额正在汇总中，请稍等");
        try {
            Thread.sleep(2000);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return 10000;
    }
```

说明：

当前任务：(一般指机器自动完成，但需要耗费一定时间的工作)完成后，向后推移流程，可以调用runtimeService.signal(executionId)，传递接受执行对象的Id。

## 个人任务

### 1.流程图

![](D:\images\2020-1-28\20200128123444.png)

### 2.分配个人任务方式一(直接指定办理人)

![](D:\images\2020-1-28\20200128123710.png)

缺点：

1. 办理人固定了，但是实际开发中，办理人是不固定的。(张三是个人任务的办理人)
2. 但是这样分配任务的办理人不够灵活，因为项目开发中任务的办理人不要放置文件中。

### 3.分配个人任务方式二(使用流程变量)

#### 1.流程图

![](D:\images\2020-1-28\20200130130312.PNG)

#### 2.部署流程定义

```java
    @Test
    public void deployProcess() {
        RepositoryService repositoryService = processEngine.getRepositoryService();
        Deployment deploy = repositoryService.createDeployment()
                .name("请假流程001")
                .addClasspathResource("HelloWorld.bpmn")
                .addClasspathResource("HelloWorld.png")
                .deploy();
        System.out.println("部署ID：" + deploy.getId());
    }
```

#### 3.启动流程实例

```java
    /**
     * 启动流程并指定下一个任务的办理人
     */
    @Test
    public void startProcess() {
        RuntimeService runtimeService = processEngine.getRuntimeService();
        Map<String, Object> map = new HashMap<>();
        map.put("username", "张三");
        ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("HelloWorld", map);
        System.out.println("流程启动成功，流程实例ID：" + processInstance.getId() + " " + processInstance.getProcessDefinitionId());
    }
```

#### 4.查询我的个人任务

```java
    @Test
    public void queryTask() {
        TaskService taskService = processEngine.getTaskService();
        List<Task> list = taskService.createTaskQuery().taskAssignee("张三").list();
        list.forEach(task -> {
            System.out.println("任务ID:" + task.getId());
            System.out.println("任务办理人:" + task.getAssignee());
            System.out.println("执行实例ID:" + task.getExecutionId());
            System.out.println("任务名称:" + task.getName());
            System.out.println("流程定义ID:" + task.getProcessDefinitionId());
            System.out.println("流程实例ID:" + task.getProcessInstanceId());
            System.out.println("任务创建时间:" + task.getCreateTime());
        });
    }
```

#### 5.办理任务

```java
    @Test
    public void complete() {
        TaskService taskService = processEngine.getTaskService();
        String taskId = "5002";
        Map<String, Object> map = new HashMap<>();
        map.put("username", "李四");//办理任务指定办理人
        taskService.complete(taskId, map);//总经理在办的任务不用加username了,map除掉即可
        System.out.println("办理成功");
    }
```

说明：

1. 张三是个人任务的办理人。
2. 在开发中，可以在页面中指定下一个任务的办理人，通过流程变量设置下一个任务的办理人。

### 4.分配个人任务方式三(使用类)

#### 1.创建一个监听类

```java
/**
 * 监听器
 * @author baerwang
 * @date 2020/1/30 13:53
 */
public class TaskListenerImpl implements TaskListener {

    @Override
    public void notify(DelegateTask delegateTask) {
        System.out.println("Welcome");
        /*指定办理人*/
        String assignee = "李四";
        delegateTask.setAssignee(assignee);
    }
}
```

#### 2.添加流程监听器(流程图)

![](D:\images\2020-1-28\20200130140952.PNG)

![](D:\images\2020-1-28\20200130181441.PNG)

#### 3.部署流程定义

```java
      private ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();

	@Test
    public void deployProcess() {
        RepositoryService repositoryService = processEngine.getRepositoryService();
        Deployment deploy = repositoryService.createDeployment()
                .name("请假流程001")
                .addClasspathResource("HelloWorld.bpmn")
                .addClasspathResource("HelloWorld.png")
                .deploy();
        System.out.println("部署ID：" + deploy.getId());
    }
```

#### 4.启动流程实例

```java
    @Test
    public void startProcess() {
        RuntimeService runtimeService = processEngine.getRuntimeService();
        Map<String, Object> map = new HashMap<>();
        map.put("username", "张三");//第一次启动申请人${username},没有使用类
        ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("myProcess", map);
        System.out.println("流程启动成功，流程实例ID：" + processInstance.getId() + " " + processInstance.getProcessDefinitionId());
    }
```

#### 5.查看我的个人任务

```java
    @Test
    public void queryTask() {
        TaskService taskService = processEngine.getTaskService();
        List<Task> list = taskService.createTaskQuery().taskAssignee("张三").list();
        list.forEach(task -> {
            System.out.println("任务ID:" + task.getId());
            System.out.println("任务办理人:" + task.getAssignee());
            System.out.println("执行实例ID:" + task.getExecutionId());
            System.out.println("任务名称:" + task.getName());
            System.out.println("流程定义ID:" + task.getProcessDefinitionId());
            System.out.println("流程实例ID:" + task.getProcessInstanceId());
            System.out.println("任务创建时间:" + task.getCreateTime());
        });
    }
```

#### 6.办理任务

```java
    @Test
    public void complete() {
        TaskService taskService = processEngine.getTaskService();
        String taskId = "2505";
        taskService.complete(taskId);
        System.out.println("办理成功");
    }
```

### 5.总结

个人任务及三种分配方式：

在taskProcess.bpmn中直接写assignee="张三"。

在taskProcess.bpmn中写assignee="#{userId}"，变量的值是String。使用流程变量指定办理人。

使用TaskListener接口，要使类实现该接口，在类中定义，

```java
delegateTask.setAssignee(assignee);//指定个人任务的办理人.
```

使用任务ID和办理人重新指定办理人：

```java
processEngine.getTaskService().setAssignee(taskId,userId)
```

## 用户组任务

### 1.流程图

![](D:\images\2020-1-28\20200130160527.PNG)

### 2.分配个人任务方式一(直接指定办理人)

#### 1.流程图中任务节点的配置

![](D:\images\2020-1-28\20200130161822.PNG)

#### 2.部署流程定义

```java
    @Test
    public void deployProcess() {
        RepositoryService repositoryService = processEngine.getRepositoryService();
        Deployment deploy = repositoryService.createDeployment()
                .name("请假流程001")
                .addClasspathResource("Dept.bpmn")
                .addClasspathResource("Dept.png")
                .deploy();
        System.out.println("部署ID：" + deploy.getId());
    }
```

#### 3.启动流程实例

```java
    @Test
    public void startProcess() {
        RuntimeService runtimeService = processEngine.getRuntimeService();
        ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("myProcess");
        System.out.println("流程启动成功，流程实例ID：" + processInstance.getId() + " " + processInstance.getProcessDefinitionId());
    }
```

#### 4.查询组任务成员列表

​	数据表：act_ru_task
​			"2504" 是这个表的ID

```java
    @Test
    public void queryGroupUser() {
        List<IdentityLink> identityLinksForTask = processEngine.getTaskService().getIdentityLinksForTask("2504");
        identityLinksForTask.forEach(identityLink -> {
            System.out.println("userId：" + identityLink.getUserId());
            System.out.println("taskId：" + identityLink.getTaskId());
            System.out.println("type：" + identityLink.getType());
            System.out.println();
        });
    }
```

#### 5.查询任务

```java
    @Test
    public void queryTask() {
        TaskService taskService = processEngine.getTaskService();
        List<Task> list = taskService.createTaskQuery().taskAssignee("小A").list();
        list.forEach(task -> {
            System.out.println("任务ID:" + task.getId());
            System.out.println("任务办理人:" + task.getAssignee());
            System.out.println("执行实例ID:" + task.getExecutionId());
            System.out.println("任务名称:" + task.getName());
            System.out.println("流程定义ID:" + task.getProcessDefinitionId());
            System.out.println("流程实例ID:" + task.getProcessInstanceId());
            System.out.println("任务创建时间:" + task.getCreateTime());
        });
    }
```

#### 6.查询组任务

```java
    @Test
    public void queryGroupTask() {
        TaskService taskService = processEngine.getTaskService();
        List<Task> list = taskService.createTaskQuery().taskCandidateUser("小B").list();
        list.forEach(task -> {
            System.out.println("任务ID:" + task.getId());
            System.out.println("任务办理人:" + task.getAssignee());
            System.out.println("执行实例ID:" + task.getExecutionId());
            System.out.println("任务名称:" + task.getName());
            System.out.println("流程定义ID:" + task.getProcessDefinitionId());
            System.out.println("流程实例ID:" + task.getProcessInstanceId());
            System.out.println("任务创建时间:" + task.getCreateTime());
        });
    }
```

#### 7.拾取任务

```java
    @Test
    public void claim() {
        TaskService taskService = processEngine.getTaskService();
        taskService.claim("2504", "小A");
        System.out.println("任务拾取成功");
    }
```

#### 8.任务拾取回退

```java
    @Test
    public void claimBack() {
        TaskService taskService = processEngine.getTaskService();
        taskService.setAssignee("2504", null);
        System.out.println("任务回退成功");
    }
```

#### 9.办理任务

```java
    @Test
    public void complete() {
        TaskService taskService = processEngine.getTaskService();
        taskService.complete("2504");
        System.out.println("办理成功");
    }
```

#### 10.说明

1. 小A，小B，小C，小D时组任务的办理人
2. 但是这样分配组任务的办理人不够灵活，因为项目开发中的办理人不要放置XML文件中。
3. act_ru_identitylink表存放的任务的办理人，包括个人任务和组任务，表示正在执行的任务。
4. act_hi_identitylink表存放任务的办理人，包括个人任务和组任务，表示历史任务，区别在于，如果是个人任务type的类型表示participant(参与者)，如果是组任务type的类型表示candidate(候选者)和participant(参与者)。

### 3.分配个人任务方式二(使用流程变量)

#### 1.流程图中任务节点的配置

![](D:\images\2020-1-28\20200130170141.PNG)

#### 2.部署流程定义

```java
      private ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();

	@Test
    public void deployProcess() {
        RepositoryService repositoryService = processEngine.getRepositoryService();
        Deployment deploy = repositoryService.createDeployment()
                .name("请假流程001")
                .addClasspathResource("Dept.bpmn")
                .addClasspathResource("Dept.png")
                .deploy();
        System.out.println("部署ID：" + deploy.getId());
    }
```

#### 3.启动流程实例

```java
@Test
public void startProcess() {
    RuntimeService runtimeService = processEngine.getRuntimeService();
    Map<String, Object> map = new HashMap<>();
    map.put("userNames", "小A,小B,小C,小D");
    ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("myProcess",map);
    System.out.println("流程启动成功，流程实例ID：" + processInstance.getId() + " " + processInstance.getProcessDefinitionId());
}
```

#### 4.查询组任务成员列表

```java
    @Test
    public void queryGroupUser() {
        List<IdentityLink> identityLinksForTask = processEngine.getTaskService().getIdentityLinksForTask("2505");
        identityLinksForTask.forEach(identityLink -> {
            System.out.println("userId：" + identityLink.getUserId());
            System.out.println("taskId：" + identityLink.getTaskId());
            System.out.println("type：" + identityLink.getType());
            System.out.println();
        });
    }
```

#### 5.查询任务

```java
    @Test
    public void queryTask() {
        TaskService taskService = processEngine.getTaskService();
        List<Task> list = taskService.createTaskQuery().taskAssignee("小A").list();
        list.forEach(task -> {
            System.out.println("任务ID:" + task.getId());
            System.out.println("任务办理人:" + task.getAssignee());
            System.out.println("执行实例ID:" + task.getExecutionId());
            System.out.println("任务名称:" + task.getName());
            System.out.println("流程定义ID:" + task.getProcessDefinitionId());
            System.out.println("流程实例ID:" + task.getProcessInstanceId());
            System.out.println("任务创建时间:" + task.getCreateTime());
        });
    }
```

#### 6.查询组任务

```java
    @Test
    public void queryGroupTask() {
        TaskService taskService = processEngine.getTaskService();
        List<Task> list = taskService.createTaskQuery().taskCandidateUser("小A").list();
        list.forEach(task -> {
            System.out.println("任务ID:" + task.getId());
            System.out.println("任务办理人:" + task.getAssignee());
            System.out.println("执行实例ID:" + task.getExecutionId());
            System.out.println("任务名称:" + task.getName());
            System.out.println("流程定义ID:" + task.getProcessDefinitionId());
            System.out.println("流程实例ID:" + task.getProcessInstanceId());
            System.out.println("任务创建时间:" + task.getCreateTime());
        });
    }
```

#### 7.任务拾取

```java
    @Test
    public void claim() {
        TaskService taskService = processEngine.getTaskService();
        taskService.claim("2505", "小A");
        System.out.println("任务拾取成功");
    }
```

#### 8.任务回退

```java
    @Test
    public void claimBack() {
        TaskService taskService = processEngine.getTaskService();
        taskService.setAssignee("2505", null);
        System.out.println("任务回退成功");
    }
```

#### 9.组任务添加成员

```java
    @Test
    public void addGroupUser() {
        TaskService taskService = processEngine.getTaskService();
        taskService.addCandidateUser("2505", "小E");
        System.out.println("添加成功");
    }
```

#### 10.组任务删除成员

```java
    @Test
    public void deleteGroupUser() {
        TaskService taskService = processEngine.getTaskService();
        taskService.deleteCandidateUser("2505", "小E");
        System.out.println("删除成功");
    }
```

#### 11.任务办理

```java
    @Test
    public void complete() {
        TaskService taskService = processEngine.getTaskService();
        taskService.complete("2505");
        System.out.println("办理成功");
    }
```

#### 12.说明

1. 小A，小B，小C，小E是组任务的办理人
2. 在开发中，可以在页面中指定下一组任务的办理人，通过流程变量设置下一个任务的办理人

### 4.分配个人任务方式三(使用类)

#### 1.创建监听器

```java
/**
 * 监听器
 * @author baerwang
 * @date 2020/1/30 13:53
 */
public class TaskListenerImpl implements TaskListener {

    @Override
    public void notify(DelegateTask delegateTask) {
        System.out.println("Welcome");
        delegateTask.addCandidateUser("A");
        delegateTask.addCandidateUser("B");
        delegateTask.addCandidateUser("C");
        delegateTask.addCandidateUser("D");
    }
}
```

#### 2.添加流程监听器(流程图)

![](D:\images\2020-1-28\20200130175558.PNG)

![](D:\images\2020-1-28\20200130181203.PNG)

#### 3.部署流程定义

```java
    private ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    private TaskService taskService = processEngine.getTaskService();

    @Test
    public void deployProcess() {
        RepositoryService repositoryService = processEngine.getRepositoryService();
        Deployment deploy = repositoryService.createDeployment()
                .name("请假流程001")
                .addClasspathResource("Dept.bpmn")
                .addClasspathResource("Dept.png")
                .deploy();
        System.out.println("部署ID：" + deploy.getId());
    }
```

#### 4.启动流程实例

```java
    @Test
    public void startProcess() {
        RuntimeService runtimeService = processEngine.getRuntimeService();
        ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("myProcess");
        System.out.println("流程启动成功，流程实例ID：" + processInstance.getId() + " " + processInstance.getProcessDefinitionId());
    }
```

#### 5.查询组任务成员列表

```java
    @Test
    public void queryGroupUser() {
        List<IdentityLink> identityLinksForTask = processEngine.getTaskService().getIdentityLinksForTask("2504");
        identityLinksForTask.forEach(identityLink -> {
            System.out.println("userId：" + identityLink.getUserId());
            System.out.println("taskId：" + identityLink.getTaskId());
            System.out.println("type：" + identityLink.getType());
            System.out.println();
        });
    }
```

#### 6.查询任务

```java
    @Test
    public void queryTask() {
        List<Task> list = taskService.createTaskQuery().taskAssignee("A").list();
        list.forEach(task -> {
            System.out.println("任务ID:" + task.getId());
            System.out.println("任务办理人:" + task.getAssignee());
            System.out.println("执行实例ID:" + task.getExecutionId());
            System.out.println("任务名称:" + task.getName());
            System.out.println("流程定义ID:" + task.getProcessDefinitionId());
            System.out.println("流程实例ID:" + task.getProcessInstanceId());
            System.out.println("任务创建时间:" + task.getCreateTime());
        });
    }
```

#### 7.查询组任务

```java
    @Test
    public void queryGroupTask() {
        List<Task> list = taskService.createTaskQuery().taskCandidateUser("A").list();
        list.forEach(task -> {
            System.out.println("任务ID:" + task.getId());
            System.out.println("任务办理人:" + task.getAssignee());
            System.out.println("执行实例ID:" + task.getExecutionId());
            System.out.println("任务名称:" + task.getName());
            System.out.println("流程定义ID:" + task.getProcessDefinitionId());
            System.out.println("流程实例ID:" + task.getProcessInstanceId());
            System.out.println("任务创建时间:" + task.getCreateTime());
        });
    }
```

#### 8.任务拾取

```java
    @Test
    public void claim() {
        taskService.claim("2504", "A");
        System.out.println("任务拾取成功");
    }
```

#### 9.任务回退

```java
    @Test
    public void claimBack() {
        taskService.setAssignee("2504", null);
        System.out.println("任务回退成功");
    }
```

#### 10.组任务添加成员

```java
    @Test
    public void addGroupUser() {
        taskService.addCandidateUser("2504", "E");
        System.out.println("添加成功");
    }
```

#### 11.组任务删除成员

```java
    @Test
    public void deleteGroupUser() {
        taskService.deleteCandidateUser("2504", "E");
        System.out.println("删除成功");
    }
```

#### 12.任务办理

```java
    @Test
    public void complete() {
        taskService.complete("2504");
        System.out.println("办理成功");
    }
```

#### 13.说明

1. 在类中使用delegateTask.addCandidateUser(userId)，的方式分配组任务的办理人，此时A，B，C，D是下个任务的办理人。
2. 通过processEngine.getTaskService.claim(taskId,userId)，将组任务分配给个人任务，也叫认领任务，即指定某个人去办理这个任务，此时由如来去办理任务。
3. 注意：认领任务的时候，可以是组任务成员中的人，也可以部署组任务成员的人，此时通过typ的类型为participant来指定任务的办理人
4. addCandidateUser()即向组任务添加成员，deleteCandidateUser()即删除组任务的成员。
5. 在开发中，可以将每一个任务的办理人规定好，例如张三的领导是李四和王五，这样张三提交任务，由李四或者王五去查询组任务，可以看到对应张三的申请，李四或者王五再通过认领任务(claim)的方式，由某个人去完成这个任务。

### 总结
​	组任务及三种分配方式：

1. 1. 在taskProcess.bpmn中总结写candidate-users="小A,小B,小C,小D

   2. 在taskProcess.bpmn中写candidate-users="${userNames}"，变量的值要是String的。

      使用流程变量指定办理人

      ```java
      Map<String,Object> map = new HashMap<>();
      map.put("userNames","A,B,C,D")
      ```

2. 使用TaskListener接口，使用类实现该接口，在类中定义

 ```java
   /*添加组任务的用户*/
   delegateTask.addCandidateUser(userId1);
   delegateTask.addCandidateUser(userId2);
 ```

3. 组任务分配给个人任务(认领任务)

 ```java
    processEngine.getTaskService().claim(taskId, userId);
 ```

4. 个人任务分配给组任务

 ```java
    processEngine.getTaskService(). setAssignee(taskId, null);
 ```

5. 向组任务添加人员

 ```java
      processEngine.getTaskService().addCandidateUser(taskId, userId);
 ```

6. 向组删除人员

 ```java
    processEngine.getTaskService().deleteCandidateUser(taskId, userId);
 ```

7. 个人任务和组任务存放办理人对应的表

8. act_ru_identitylink表存放任务的办理人，包括个人任务和组任务，表示正在执行的任务。

9. act_hi_identitylink表存放任务的办理人，包括个人任务和组任务，表示历史任务，区别在于，如果是个人任务type的类型表示participant(参与者)，如果是组任务type的类型表示candidate(候选者)和participant(参与者)。

## 工作流定义的角色组

### 1.角色数据表

```sql
SELECT * FROM act_id_group 		#角色
SELECT * FROM act_id_membership	#用户和角色之间的关系
SELECT * FROM act_id_info		#用户的详细信息
SELECT * FROM act_id_user		#用户表
```

### 2.创建用户和用户组

```java
    private ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();

    @Test
    public void createUserAndGroup() {

        IdentityService identityService = processEngine.getIdentityService();

        /*保存act_id_group*/
        GroupEntity groupEntity = new GroupEntity("1");
        groupEntity.setName("总经理");
        identityService.saveGroup(groupEntity);
        GroupEntity groupEntity2 = new GroupEntity("2");
        groupEntity2.setName("部门经理");
        identityService.saveGroup(groupEntity2);

        /*保存act_id_user*/
        UserEntity userEntityA = new UserEntity("1");
        userEntityA.setFirstName("xiao");
        userEntityA.setLastName("A");
        userEntityA.setPassword("AA");
        userEntityA.setEmail("A@xx.com");
        identityService.saveUser(userEntityA);

        UserEntity userEntityB = new UserEntity("2");
        userEntityB.setFirstName("xiao");
        userEntityB.setLastName("B");
        userEntityB.setPassword("BB");
        userEntityB.setEmail("B@xx.com");
        identityService.saveUser(userEntityB);

        UserEntity userEntityC = new UserEntity("3");
        userEntityC.setFirstName("xiao");
        userEntityC.setLastName("C");
        userEntityC.setPassword("CC");
        userEntityC.setEmail("C@xx.com");

        identityService.saveUser(userEntityC);

        /*建立用户和组的关系 act_id_membership*/
        identityService.createMembership("1", "2");
        identityService.createMembership("2", "1");
        identityService.createMembership("3", "2");

        System.out.println("保存成功");
    }
```

## 总结

| RepositoryService | 管理流程定义                                 |
| ----------------- | -------------------------------------------- |
| RuntimeService    | 执行管理，包括启动，推进，删除流程实例等操作 |
| TaskService       | 任务管理                                     |
| HistoryService    | 历史管理(执行完的数据的管理)                 |
| IdentityService   | 组织机构管理                                 |
| FormService       | 一个可选服务，任务表单管理                   |
| ManagerService    | 管理器服务                                   |

![](D:\images\2020-1-28\2020131161048.png)