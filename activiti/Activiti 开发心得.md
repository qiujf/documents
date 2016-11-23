# Activiti 开发 

## 写在前面

几年前我在当时的公司曾经JBMP作为工作进行工作流应用的开发。当时用JBMP做开发陆续踩过一些坑，也积累了一些总结，可惜这些经验没有很好地记录下来。现在，我所在的部门需要开发一些OA工具，用于提高研发效率，由于这些OA工具牵涉到流程控制，自然而然地使用流程引擎也就列入了计划之中。不过这次不打算用JBMP，而是用Activiti.



关于Activiti的开发，首先有两份文档，一份是[官方的开发指南](http://www.activiti.org/userguide/),另一份是电子书**Activiti in Action.pdf**



### E1: 使用Activiti Explorer UI还是定制开发UI

Activiti套件中提供了完整的用户表单开发功能，使用FormService就可以设计出基本的用户表单并在Activiti Explorer中运行，这样开发者就省去了开发UI的开发代价。

对于要求不高的内部系统，Activiti 自带的UI功能可以满足基本的需求，但是如果期望设计出具有更好体验的UI，这个时候就只能自己开发用户界面了。其实在大多数的流程开发项目中，为了满足各种复杂的业务需求，直接使用Activiti Explorer的情况并不多。



### E2: 正确使用Activiti Engine API接口

Activiti Engine API提供了以下的接口

- FormService 
- HistoryService
- IdentityService
- ManagementService
- RepositoryService
- RuntimeService
- TaskService

对于Activiti的流程、数据地操作，要通过以上几个接口来实现。这是因为Activiti的运行数据是存储在数据库中的，包括流程（process instance)状态、任务（task)的状态、历史记录等。Activiti流程引擎在执行流程是从数据库中读取数据，处理，存储；因此要获取准确的数据必须通过API接口调用。

举个例子

``` java
        RuntimeService runtimeService = processEngine.getRuntimeService();
        ProcessInstance processInstance =  runtimeService.startProcessInstanceById();
        processInstance.getProcessVariables().put("key","value");
```

这段代码貌似向processInstance设置了一个变量，但是实际上操作是一个内存对象，变量值并不会被保存至数据库，正确的写法是

``` java
      ProcessInstance processInstance =  runtimeService.startProcessInstanceById();
      processInstance.getProcessVariables().put("key","value")
```



### E3: 对流程进行封装

尽管Activiti的API可以直接对流程进行操作，但是从协作编程的角度考虑，我们需要对流程进行封装（ProcessWrapper)。一般来说提供以下几类接口较为合适

*  流程的启动（Input：启动参数；output:流程ID）
*  执行Human Task接口（Input: 流程ID、任务参数；output:执行结果）
*  状态接口（Input:流程ID; output:流程的状态。

定义了这些接口后，流程的使用者就可以并行使用这些接口定义进行开发了，而流程开发可以专注在内部流程的设计、调试、优化。



### E4: 使用尽可能少的流程变量

我们曾经试图在流程中存储许多数据，这些数据中的很多跟流程的运行并没有决定关系，即不是决定流程走向的变量，也不是Human Task需要的变量，也不是调用外部服务时需要的变量。只是想利用Activiti的变量存储机制来存储一些数据。这么做的一个结果就是让流程的执行速度变慢，一般的我只会考虑以下变量

* 决定流程走向的变量，比如在分支流程线上起作用的变量。
* 调用外部服务（External Service) 需要的变量，但是也可以保存一个简单的主键数据，然后在调用外部服务时计算生成外部服务需要的参数变量。
* Human Task的参数，保存这些数据的原因是这些数据会被存入历史数据以备审核。
* 在脚本任务(Script Task)中使用到的变量。



### E5: 使用回调方法来执行长时任务 

对于需要较长执行时间的任务，可以考虑回调的方法。即设置一个Human task 或 Event Message Trigger来触发回调事件继续流程的执行。



### E6: 流程之间的交互, Human Task不仅仅是Human Task

流程间的交互可以通过传变量，调用Human task的方法来交互。Activiti流程中的Human Task在现实中不一定必须由真实的用户来触发，很多时候系统也可以去调用complete Human Task, 这也给我们实现流程的自动交互提供了一种方式。



### E7: Signal, Message 也可以用于流程交互

流程之间的通信，还有一个方便的作法是利用Singal  或 Message的机制来实现流程。



### E8: 调用子流程SubProcess

调用子流程的过程可以使用Call Activity，这种方法下子流程的定义是独立于父流程的，或者使用内嵌子流程，也就是子流程定义在父流程的内部。



### E9: 获取流程的执行状态

* 通过获取Human Task的方式，来确定流程的位置；
* 通过查询Active Activities节点的方式，来确定流程的位置；
* 通过利用事件监听器或脚本任务（Script Task)在流程运行中设置代表流程状态的变量，并用它来确定流程的状态。



















##  