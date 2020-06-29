

[TOC]

## 引言

工作流引擎，应用于解决流程审批和流程编排方面等问题，有效的提供了扩展性的支撑。而目前来说，工作流领域也有了相对通行化的标准规范，也就是BPMN2.0。支持这个规范的开源引擎主要有：Activiti，flowable，Jbpm4等。

## Activiti架构

网上找的一些分层架构图



![img](/Users/fanxudong/IdeaProjects/blog/4 Java/assert/bfc57552-9d60-3483-9577-3503688de2d4.png)

![img](/Users/fanxudong/IdeaProjects/blog/11 activiti/asset/20190214113258.png)

大致包括：

- 核心接口层，被PVM接口定义。
- 核心实现层，基于PVM的思想和接口，定义了一些关键实体。
  - ActivityImpl（该类抽象了节点实现）
  - FlowElementBehavior实现（该类抽象了节点指令动作）
  - ExecutionImpl（流程执行实体类）
- 命令层，Activiti在编码模式上直接限定整体风格为命令模式。也就是将业务逻辑封装为一个个的Command接口实现类。这样新增一个业务功能时只需要新增一个Command实现即可。这里需要特别提到的是，命令本身需要运行在命令上下文中，也就是CommandContext类对象。
- 命令拦截层，采用责任链模式，通过责任链模式的拦截器层，为命令的执行创造条件。诸如开启事务，创建CommandContext上下文，记录日志等
- 业务接口层，面向业务，提供了各种接口。这部分的接口就不再面向框架开发者了，而是面向框架的使用者。
- 部署层，严格来说，这个与上面说到的并不是一个完整的分层体系。但是为了突出重要性，单独拿出来说。流程运转的前提是流程定义。而流程定义解析就是一切的开始。从领域语言解析为Java的POJO对象依靠的就是部署层。
- 流程引擎，所有接口的总入口。上面提到的业务接口层，部署层都可以从流程引擎类中得到。因此这里的流程引擎接口其实类似门面模式，只作为提供入口。

### ProcessEngine（流程引擎）

1. ProcessEngine是所有Activiti引擎功能的中心入口。
2. 定义Activiti流程引擎的配置文件，利用Spring的解析和依赖注入功能来构建**引擎**。
3. 配置文件必须包含一个id为**processEngineConfiguration**的bean，用于构建ProcessEngine，**org.activiti.spring.SpringProcessEngineConfiguration**为在Spring环境下使用的流程引擎，用于定义processEngineConfiguration。
4. SpringProcessEngineConfiguration实现了ApplicationContextAware接口，在唯一的接口方法setApplicationContext中，获取spring容器。
5. 当把数据源（DataSource）传递给 SpringProcessEngineConfiguration （使用"dataSource"属性）之后，Activiti内部使用了一个org.springframework.jdbc.datasource.**TransactionAwareDataSourceProxy**代理来封装传递进来的数据源（DataSource）。 这样做是为了确保从数据源（DataSource）获取的SQL连接能够与Spring的事物结合在一起发挥得更出色。这意味它不再需要在你的Spring配置中**代理数据源**（dataSource）了。

#### 流程引擎初始化

org.activiti.engine.impl.cfg.ProcessEngineConfigurationImpl类的init()，Activiti工作流引擎的初始化。 

```java
public abstract class ProcessEngineConfigurationImpl extends ProcessEngineConfiguration {

  @Override
  public ProcessEngine buildProcessEngine() {
    init();
    ProcessEngineImpl processEngine = new ProcessEngineImpl(this); 
    postProcessEngineInitialisation();

    return processEngine;
  }  
  
	public void init() {
    initConfigurators();
    configuratorsBeforeInit();
    initHistoryLevel();
    initExpressionManager();

    if (usingRelationalDatabase) {
      initDataSource();
    }

    initAgendaFactory();
    initHelpers();
    initVariableTypes();
    initBeans();
    initScriptingEngines();
    initClock();
    initBusinessCalendarManager();
    initCommandContextFactory();
    initTransactionContextFactory();
    initCommandExecutors(); //
    initServices();
    initIdGenerator();
    initBehaviorFactory();
    initListenerFactory();
    initBpmnParser();
    initProcessDefinitionCache();
    initProcessDefinitionInfoCache();
    initKnowledgeBaseCache();
    initJobHandlers();
    initJobManager();
    initAsyncExecutor(); // 

    initTransactionFactory();

    if (usingRelationalDatabase) {
      initSqlSessionFactory();
    }

    initSessionFactories();
    initDataManagers();
    initEntityManagers();
    initHistoryManager();
    initJpa();
    initDeployers();
    initDelegateInterceptor();
    initEventHandlers();
    initFailedJobCommandFactory();
    initEventDispatcher();
    initProcessValidator();
    initDatabaseEventLogging();
    configuratorsAfterInit();
  }
  
  public ProcessEngineImpl(ProcessEngineConfigurationImpl processEngineConfiguration) {
    this.processEngineConfiguration = processEngineConfiguration;
    this.name = processEngineConfiguration.getProcessEngineName();
    this.repositoryService = processEngineConfiguration.getRepositoryService();
    this.runtimeService = processEngineConfiguration.getRuntimeService();
    this.historicDataService = processEngineConfiguration.getHistoryService();
    this.taskService = processEngineConfiguration.getTaskService();
    this.managementService = processEngineConfiguration.getManagementService();
    this.dynamicBpmnService = processEngineConfiguration.getDynamicBpmnService();
    this.asyncExecutor = processEngineConfiguration.getAsyncExecutor();
    this.commandExecutor = processEngineConfiguration.getCommandExecutor();
    this.sessionFactories = processEngineConfiguration.getSessionFactories();
    this.transactionContextFactory = processEngineConfiguration.getTransactionContextFactory();

    if (processEngineConfiguration.isUsingRelationalDatabase() && processEngineConfiguration.getDatabaseSchemaUpdate() != null) {
      commandExecutor.execute(processEngineConfiguration.getSchemaCommandConfig(), new SchemaOperationsProcessEngineBuild());
    }

    if (name == null) {
      log.info("default activiti ProcessEngine created");
    } else {
      log.info("ProcessEngine {} created", name);
    }

    ProcessEngines.registerProcessEngine(this);

    if (asyncExecutor != null && asyncExecutor.isAutoActivate()) {
      asyncExecutor.start(); // 启动异步任务
    }

    if (processEngineConfiguration.getProcessEngineLifecycleListener() != null) {
      processEngineConfiguration.getProcessEngineLifecycleListener().onProcessEngineBuilt(this);
    }

    processEngineConfiguration.getEventDispatcher().dispatchEvent(ActivitiEventBuilder.createGlobalEvent(ActivitiEventType.ENGINE_CREATED));
  }  
}
```



- AsyncExecutor在构建ProcessEngine时启动（start）。若通过xml配置方式创建ProcessEngine，需要在SpringProcessEngineConfiguration的配置中加上

  ```xml
  <property name="asyncExecutorActivate" value="true" />。
  ```

  

- AsyncExecutor在start的时候，初始化AcquireTimerJobsRunnable、**AcquireAsyncJobsDueRunnable**、DefaultExecuteAsyncRunnableFactory，并使用Thread的start函数启动。

```java
  // 监听新的需要执行的定时作业，交给JobManager处理；
  protected AcquireTimerJobsRunnable timerJobRunnable; 

  // 监听新的需要执行的异步作业，交给JobManager处理；
  protected AcquireAsyncJobsDueRunnable asyncJobsDueRunnable;

  // 监听过期作业，重置过期作业；
  protected ResetExpiredJobsRunnable resetExpiredJobsRunnable;
```



- AcquireAsyncJobsDueRunnable中会去数据库中查询成功锁定的JobEntity任务。并放到线程池中执行任务。

### 业务接口层

#### RuntimeService

**启动一个流程实例**

把流程定义发布到Activiti引擎后，我们可以基于它发起新流程实例。 对每个流程定义，都可以有很多流程实例。 流程定义是“蓝图”，流程实例是它的一个运行的执行。

所有与流程运行状态相关的东西都可以通过**RuntimeService**获得。 有很多方法可以启动一个新流程实例。在下面的代码中，我们使用定义在流程定义xml 中的key来启动流程实例。 我们也可以在流程实例启动时添加一些流程变量，因为第一个用户任务的表达式需要这些变量。 流程变量经常会被用到，因为它们赋予来自同一个流程定义的不同流程实例的特别含义。 简单来说，流程变量是区分流程实例的关键。

```java
Map<String, Object> variables = new HashMap<String, Object>();
variables.put("employeeName", "Kermit");
variables.put("numberOfDays", new Integer(4));
variables.put("vacationMotivation", "I'm really tired!");
RuntimeService runtimeService = processEngine.getRuntimeService();
ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("vacationRequest", variables);
// Verify that we started a new process instance
Log.info("Number of process instances: " + runtimeService.createProcessInstanceQuery().count());

```

**RuntimeService**负责启动一个流程定义的新实例。

如上所述，流程定义定义了流程各个节点的结构和行为。 流程实例就是这样一个流程定义的实例。对每个流程定义来说，同一时间会有很多实例在执行。

RuntimeService也可以用来**获取和保存流程变量**。 这些数据是特定于某个流程实例的，并会被很多流程中的节点使用 （比如，一个排他网关常常使用流程变量来决定选择哪条路径继续流程）。

Runtimeservice也能**查询流程实例和执行**。 执行对应BPMN 2.0中的'token'。基本上执行指向流程实例当前在哪里。 

RuntimeService可以在流程实例等待外部触发时使用，这时可以用来继续流程实例。 流程实例可以有很多暂停状态，而服务提供了多种方法来'触发'实例， 接受外部触发后，流程实例就会继续向下执行。

```java
runtimeService.suspendProcessInstanceById(instance.getProcessInstanceId());
```

#### TaskService

任务是**由系统中真实人员执行**的，它是Activiti这类BPMN引擎的核心功能之一。 

所有与任务有关的功能都包含在**TaskService**中：

\* **查询**分配给用户或组的任务

\* 创建**独立运行任务**。这些任务与流程实例无关。

\* 手工设置任务的**执行者**，或者这些用户通过何种方式与任务关联。

\* 认领并完成一个任务。认领意味着一个人期望成为任务的执行者， 即这个用户会完成这个任务。完成意味着“做这个任务要求的事情”。 通常来说会有很多种处理形式。

#### RepositoryService

**RepositoryService**负责静态信息（比如，不会改变的数据，至少是不怎么改变的），提供了管理和控制**发布包**和**流程定义**的操作。 

**流程定义**是BPMN 2.0流程的java实现。 它包含了一个流程每个环节的结构和行为。 

**发布包**是Activiti引擎的打包单位。一个发布包可以包含多个BPMN 2.0 xml文件和其他资源。 开发者可以自由选择把任意资源包含到发布包中。 既可以把一个单独的BPMN 2.0 xml文件放到发布包里，也可以把整个流程和相关资源都放在一起。 （比如，'hr-processes'实例可以包含hr流程相关的任何资源）。 

可以通过RepositoryService来**部署**这种发布包。 发布一个发布包，意味着把它上传到引擎中，所有流程都会在保存进数据库之前分析解析好。 从这点来说，系统知道这个发布包的存在，发布包中包含的流程就已经可以启动了。

除此之外，服务可以

\* **查询**引擎中的发布包和流程定义。

\* **暂停**或**激活**发布包，对应全部和特定流程定义。 暂停意味着它们不能再执行任何操作了，激活是对应的反向操作。

\* 获得多种资源，像是包含在发布包里的文件， 或引擎自动生成的流程图。

\* 获得流程定义的pojo版本， 可以用来通过java解析流程，而不必通过xml。

**repositoryService**.**createProcessDefinitionQuery**().processDefinitionKey(flowKey).list().isEmpty()

#### historyService

#### managementService

### 领域模型（持久层）

在org.activiti.engine.impl.db.DbSqlSession中封装了org.apache.ibatis.session.SqlSession（mybatis）进行数据库操作



1. Deployment：流程部署对象，部署一个流程时创建。
2. ProcessDefinitions：流程定义，部署成功后自动创建。
3. ProcessInstances：流程实例，启动流程时创建。 
4. Task：任务，在Activiti中的Task仅指有角色参与的任务，即定义中的UserTask。 
5. Execution：执行计划，流程实例和流程执行中的所有节点都是Execution，如UserTask、ServiceTask等。



#### 表关系

在创建流程后会在流程数据库中插入一条ExecutionEntity数据，在act_ru_execution表中可以看到这个execution的id和proc_inst_id的值是一样的

![D27F86F5-3D32-49FC-8EA0-96ED4030CACA](/Users/fanxudong/IdeaProjects/blog/4 Java/assert/D27F86F5-3D32-49FC-8EA0-96ED4030CACA.png)




### 命令模式

> 1）一个产品或者一个项目，从技术上必须有一个明确的、唯一的开发模型或者叫开发样式（真不知道怎么说恰当），我们常说希望一个团队的所有人写出的代码都有统一的风格，都像是一个人写出来的，很理想化，但做到很难，往往我们都是通过“规范”去约束大家这样做，而规范毕竟是程序之外的东西，主观性很强，不遵守规范的情况屡屡发生。而如果架构师给出了明确的开发模型，并使用一些基础组件加以强化，把程序员要走的路规定清楚，那你想不遵守规范都会很难，因为那意味着你写的东西没发工作。就像Activiti做的这样，明确以Command作为基本开发模型，辅之以Event-Listener，这样编程风格的整体性得到了保证。
>
> 2）使用命令模式的好处，我这里体会最深的就是 职责分离，解耦。有了Command，各个Service从角色上说只是一些协调者或者控制者，他不需要知道具体怎么做，他只是把任务交给了各个命令。直接的好处是臃肿的、万能的大类没有了。而这往往是我们平时开发中最深恶痛绝的地方。

Activiti使用命令模式作为基础开发模式。将命令进行封装，发出命令（命令执行内容的实现）和执行命令（真正执行命令）分离。增加CommandExecutor来执行命令，同时使用职责链模式对命令进行记录、事务处理等。

重点需要关注的类

- `Command`命令接口，所有的具体命令都需要实现该类，最终业务就是执行该类的execute方法。
- `CommandContext`命令上下文，为具体命令的执行提供上下文支撑。该上下文的生成是依靠命令拦截器中的上下文拦截器`org.activiti.engine.impl.interceptor.CommandContextInterceptor`来生成的。该拦截器会判断是复用当前的上下文还是生成新的上下文。
- CommandExecutor：执行命令。开发activiti功能时，自己写Command，然后扔给CommandExecutor了事。

### 责任链模式

activiti定义了一个拦截器链，链上的每个拦截器都有个next，会一直next执行下去。以CompleteTaskCmd为例，拦截器链为：

![img](/Users/fanxudong/IdeaProjects/blog/11 activiti/asset/20191221235530.png)

#### customPreCommandInterceptors（自定义前置拦截器）

实现自定义的命令拦截器AbstractCommandInterceptor

#### LogInterceptor（日志拦截器）

使用slf4j

#### TransactionInterceptor（事务拦截器）

事务拦截器是否提供取决于`org.activiti.engine.impl.cfg.ProcessEngineConfigurationImpl`的子类对方法`createTransactionInterceptor`的实现。独立使用时的`org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration`该方法返回为空。也就是不提供事务拦截器。此时，命令的运行就无法通过事务拦截器来提供事务环境了。

项目中使用SpringTransactionInterceptor事务拦截器（使用Spring事务模板TransactionTemplate执行之后的操作，设置事务传播属性）

#### CommandContextInterceptor（命令上下文拦截器）

创建命令上下文Context，或则复用当前上下文（设置ThreadLocal变量）。

CommandContext关闭时，持久化数据，即执行context.close()方法时，调用session.flush()，真正执行完成数据库操作（提交事务）。

CommandContext包含了本次操作中涉及到所有的数据对象。

#### customPostCommandInterceptors（自定义后置拦截器）



#### CommandInvoker：执行命令







TaskEntity实现了3个接口：Task，DelegateTask和PersistentObject。其中PersistentObject是一个**声明接口**，表明TaskEntity需要持久化。接口是一种角色的声明，是一份职责的描述而TaskEntity就是这个角色的具体扮演者，因此TaskEntity必须承担如complete,delegate等职责。



actityBehavior实现了当流转到此节点时对应的处理逻辑.



在activiti里一个动作都将对应一种行为,启动流程的动作将执行PROCESS_START的原子操作

如果是异步的就在调度线程池中拿到线程执行，否则就在当前线程内执行


## bpmn模型

- BpmnModel
  - Process
    - StartEvent
    - EndEvent
    - UserTask
    - ServiceTask
    - SequenceFlow
    - ExculsiveGateway

## 流程生命周期

部署 -> 启动 -> 调度 -〉 结束

UserTask这么启动起来

## 流程操作

启动

结束

暂停

继续

重试

跳过



## 遗留问题

- 集群中如何保证一个job只有一个机器执行
- job在处理过程中失败了(其他异常或server重启),这个时候如何处理这些failed job,重新跑起来

```
当我们将serviceTask 设置 async = "true" (关于 isExclusive 后续会提到) 的时候，流程引擎采用JobExecutor 来异步执行，执行顺序为引擎首先会将该任务实例化一条job记录，插act_ru_job表，然后JobExecutor 扫描该表并加锁执行该job，
```

- 事务的可见性、事务传播导致的问题。

- 流程中启动流程，ThreadLocal导致的问题。

  

## 参考：

[Activiti源码分析（框架、核心类。。。）](https://blog.csdn.net/bluejoe2000/article/details/41790771)

[csdn白乔的博客](https://blog.csdn.net/bluejoe2000)

[Activiti 5.16 用户手册](http://www.mossle.com/docs/activiti/index.html)

[activiti源码分析（一）设计模式](https://www.cnblogs.com/lighten/p/5863102.html)

[activiti学习笔记3--20170327-Job Executor and Async Executor](https://blog.csdn.net/silent_zqy/article/details/67073439)

[activiti乐观锁与死锁](https://my.oschina.net/xiehui/blog/140704)

[Activiti架构分析及源码详解](https://www.cnblogs.com/jfire/p/12078847.html)

