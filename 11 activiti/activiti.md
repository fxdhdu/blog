1. Deployment：流程部署对象，部署一个流程时创建。
2. ProcessDefinitions：流程定义，部署成功后自动创建。
3. ProcessInstances：流程实例，启动流程时创建。 
4. Task：任务，在Activiti中的Task仅指有角色参与的任务，即定义中的UserTask。 
5. Execution：执行计划，流程实例和流程执行中的所有节点都是Execution，如UserTask、ServiceTask等。





Activiti使用命令模式作为基础开发模式

自己写Command！然后扔给CommandExecutor()了事！





activiti定义了一个拦截器链，链上的每个拦截器都有个next，会一直next执行下去。以CompleteTaskCmd为例，拦截器链为：

logger拦截器-->spring事务拦截器-->CommandContext拦截器-->CommandInvoker拦截器

其中CommandContext拦截器的工作主要是设置Context：





activiti的数据是在CommandContext关闭时，持久化到数据库里的。



TaskEntity实现了3个接口：Task，DelegateTask和PersistentObject。其中PersistentObject是一个**声明接口**，表明TaskEntity需要持久化。接口是一种角色的声明，是一份职责的描述而TaskEntity就是这个角色的具体扮演者，因此TaskEntity必须承担如complete,delegate等职责。





actityBehavior实现了当流转到此节点时对应的处理逻辑.



在activiti里一个动作都将对应一种行为,启动流程的动作将执行PROCESS_START的原子操作

如果是异步的就在调度线程池中拿到线程执行，否则就在当前线程内执行



## 表关系

在创建流程后会在流程数据库中插入一条ExecutionEntity数据，在act_ru_execution表中可以看到这个execution的id和proc_inst_id的值是一样的



1）一个产品或者一个项目，从技术上必须有一个明确的、唯一的开发模型或者叫开发样式（真不知道怎么说恰当），我们常说希望一个团队的所有人写出的代码都有统一的风格，都像是一个人写出来的，很理想化，但做到很难，往往我们都是通过“规范”去约束大家这样做，而规范毕竟是程序之外的东西，主观性很强，不遵守规范的情况屡屡发生。而如果架构师给出了明确的开发模型，并使用一些基础组件加以强化，把程序员要走的路规定清楚，那你想不遵守规范都会很难，因为那意味着你写的东西没发工作。就像Activiti做的这样，明确以Command作为基本开发模型，辅之以Event-Listener，这样编程风格的整体性得到了保证。

2）使用命令模式的好处，我这里体会最深的就是 职责分离，解耦。有了Command，各个Service从角色上说只是一些协调者或者控制者，他不需要知道具体怎么做，他只是把任务交给了各个命令。直接的好处是臃肿的、万能的大类没有了。而这往往是我们平时开发中最深恶痛绝的地方。

