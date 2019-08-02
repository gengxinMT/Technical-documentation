

## Consensus protocols:2PC&3PC

[TOC]

首先，2PC和3PC都是用来解决分布式系统中的一致性问题的协议。

## 2PC

### what is 2PC

#### without fault

两阶段提交的定义非常简单，也符合我们生活中达成共识所采用的方法：

1.  first *proposal* phase：Contact every participant, suggest a value and gather their responses
2. second *commit-or-abort* phase ：If everyone agrees, contact every participant again to let them know. Otherwise, contact every participant to *abort* the consensus.

示意图如下：

![2PC_phase_one](http://ww4.sinaimg.cn/large/006tNc79ly1g5d5vczz01j30wk0kcagy.jpg)

![2PC_phase_two](http://ww3.sinaimg.cn/large/006tNc79ly1g5d5yoqsc7j30n20m0jw1.jpg)

[Consensus Protocols: Two-Phase Commit](https://www.the-paper-trail.org/post/2008-11-27-consensus-protocols-two-phase-commit/) 中指出，在不考虑错误的情况下，2PC确实解决了共识问题。从共识问题的3个特性上去审视：agreement、validity、termination。（文中对于 validaty 的讲解还挺明确的）

#### Crashes and failure

然而在现实中，节点可能随时挂掉或出现各种错误，因此我们必须从理论上先考察该协议会出现什么样的问题。这里作者首先给出了错误的 3 种模型：Fail-stop、Fail-recover(the most commonly used model of failure)、Byzantine failure。并且明确指出接下去主要讨论的是 Fail-recover 这种错误类型，这也是一种最常见的错误类型。

1、在第一个阶段协调者挂掉的情况

![2PC_cordinator_failure_phase1](http://ww3.sinaimg.cn/large/006tNc79ly1g5d5flbr1mj30zc0k8jzt.jpg)

2、在第二个阶段协调者挂掉的情况

![2PC_cordinator_failure_phase2](http://ww4.sinaimg.cn/large/006tNc79ly1g5ixvx9gznj30mk0lmte8.jpg)

3、在第二个阶段出现协调者挂掉，部分参与者挂掉的情况

可能会出现不一致性：假设第二个阶段协调者在挂掉之前发出的是 abort 请求，一个参与者完成了 abort 操作后也挂掉了，那么剩余的参与者如果执行 commit 就会产生不一致性。

#### 2PC 优点

+ 复杂度低

2PC is still a very popular consensus protocol, because it has a low message complexity. 

#### 2PC 存在的问题

+ 阻塞
  2PC协议中，参与者一直是事务阻塞的，因此在事务进行的过程中，系统不能响应第三方节点的访问。这是偏于保守的，牺牲了一部分的可用性。

  阻塞另一个隐患来自于协调者可能的故障。由于协调者对于参与者有timeout机制，但是参与者对协调者没有timeout机制，如果协调者宕机，那么所有的参与者会跟着阻塞下去。

+ 第二阶段会出现不一致性
  假设协调者宕机了，并且部分接受到GLOBAL_COMMIT请求的参与者也宕机/分区了，那么剩余的参与者就不知道该 commit 还是 abort 了。

+ 网络分区
假设协调者发出了GLOBAL_COMMIT请求时发生了网络分区，此时有一部分节点收到消息正常commit，但另一部分节点未收到，还处于阻塞状态。

## 3PC

### what is 3PC

三阶段提交协议致力于解决 2PC 的阻塞问题以及缓解一部分不一致性的可能。为此它引入了 **参与者超时机制** 和一个额外的 **PreCommit阶段**。

![3PC](http://ww1.sinaimg.cn/large/006tNc79ly1g5d6ty3uvaj30zu0kwn16.jpg)

wiki 的示意图中并没有把 abort的情况画出来。

下图列出了每个阶段协调者和参与者的可能的状态。

![3PC](http://ww2.sinaimg.cn/large/006tNc79ly1g5ibe37f5cj31jg0u0gox.jpg)



#### 3PC 优点

+ 解决了单点阻塞的问题

The fact that 3PC will not block on single node failures makes it much more appealing for services where high availability is more important than low latencies.

+ 降低了不一致性的可能

三阶段的过程中在第三个阶段出现 failure 的可能性要比两阶段中第二阶段出现 faliure 的概率低很多，这也是为什么在第三个阶段时对于参与者的超时机制处理是让其 commit 而不是 abort 的原因，当然也是存在小概率是例外的。

#### 3PC 存在的问题

+ 网络分区
+ 小概率的不一致性



## 参考资料

1、[Consensus Protocols: Two-Phase Commit](https://www.the-paper-trail.org/post/2008-11-27-consensus-protocols-two-phase-commit/) 

2、[Consensus Protocols: Three-phase Commit](https://www.the-paper-trail.org/post/2008-11-29-consensus-protocols-three-phase-commit/)

3、http://www.calvinneo.com/2019/03/12/2pc-3pc/

4、3 阶段的理解 on wiki
https://en.wikipedia.org/wiki/Three-phase_commit_protocol

5、各种情况的理解
https://matt33.com/2018/07/08/distribute-system-consistency-protocol/

