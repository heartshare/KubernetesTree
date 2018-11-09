# KubernetesTree
Kubernetes容器技术研究

<pre>
Kubernetes简介

Kubernetes 是一个全新的基于容器技术的分布式架构领先方案，它是谷歌十几年以来大规模应用容器
技术的经验积累和升华的重要成果，确切的说，Kubernetes是谷歌严格保密十几年的秘密武器--Borg
的开源版本，Borg是谷歌的一个久负盛名的内部使用的大规模集群管理系统，它基于容器技术，目的是
实现资源管理的自动化，以及跨多个数据中心的资源利用率的最大化。

Kubernetes是一个开放的平台，不局限于任何一种语言。
Kubernetes是一个完备的分布式系统支撑平台。Kubernetes具有完备的集群管理能力，包括多层次的
安全防护和准入机制，多组合应用支撑能力，透明的服务注册和服务发现机制，内建智能负载均衡器，
强大的故障发现和自我修复能力，服务滚动升级和在线扩容能力，可扩展的资源自动调度机制，以及
多粒度的资源配额管理能力，同时Kubernetes提供了完善的管理工具，这些工具覆盖了包括开发，
部署测试，运维监控在内的各个环节，因此Kubernetes是一个全新的基于容器技术的分布式架构解
决方案，并且是一个一站式的完备的分布式系统开发和支撑平台。

在Kubernetes中，Service是分布式集群的核心，一个Service对象拥有如下关键特征。
    1）拥有一个唯一指定的名字
    2）拥有一个虚拟IP
    3）能够提供某种远程服务能力
    5）被映射到了提供这种服务能力的一组容器应用之上。

    Service的服务进程目前都基于Socket通信方式对外提供服务，比如Redis, Memcache, Mysql, Web Server或者实现了某个具体业务的一个特定TCP Server进程。

    虽然一个Service通常由多个相关的服务进程提供服务，每个服务进程都有一个独立的
    Endpoint(IP + Port)访问点，但Kubernetes能够让我们通过Service（虚拟IP + 服务端口）
    连接到指定的Service上，有了K8s内建的透明负载均衡和故障恢复机制，不管后端有多少服务进
    程，也不管某个服务进程是否会由于发生故障而重新部署到其他机器，都不会影响到我们队服务的
    正常调用。

    容器提供了强大的隔离功能，所有有必要把为Service提供服务的这组进程放入容器中进行隔离，
    为此K8s设计了Pod对象，将每个服务进程包装到相应的Pod中，使其成为Pod中运行的一个容
    器(Container)，为了建立Service和Pod间的映射关系，K8s首先给每个Pod贴上一个标签，
    然后给相应的Service定义标签选择器（Label Selector），比如Mysql Service的标签选
    择器的选择条件为name = mysql，意为该Service要作用于所有包含name = mysql Label的
    Pod上，这样巧妙的解决了Service与Pod的关联问题。

    Pod运行在一个我们称之为节点的环境中，这个节点既可以是物理的，也可以是私有云或者公有
    云中的一个虚拟机，通常在一个节点上运行几百个Pod;其次每个Pod里运行着一个特殊的被称之
    为Pause的容器，其他则为业务容器，这些业务容器共享Pause容器的网络栈和Volume挂载卷，
    因此它们之间的通信和数据交换更为高效，在设计时，我们可以充分利用这一特性将一组密切相
    关的服务进程放入同一个Pod中；最后，需要注意的是，并不是每个Pod和它里面运行的容器都
    能“映射到”一个Service上，只有那些提供服务的一组Pod才会被映射成一个服务。


    在集群管理方面，K8s将集群中的机器划分为一个Master节点和一群工作节点(Node)，其中，在Master节点上运行着集群管理相关的一组进程 kube-apiserver, kube-controller-manager和kube-scheduler，这些进程实现了整个集群的资源管理，Pod调度，弹性伸缩，安全控制，系统监控，和纠错等管理功能，并且全都是自动完成的，Node作为集群中的工作节点，运行真正的应用程序，在Node上，K8s管理的最小单元是Pod,Node上运行着K8s的kubelet,kube-proxy服务进程，这些服务进程负责Pod的创建，启动，监控，重启，销毁，以及实现软件模式的负载均衡。

    在K8s集群中，你只需为需要扩容的Service关联的Pod创建一个Replication Controller（简称RC），则该Service的扩容以至于后来的Service升级等头疼问题都将迎刃而解。在一个RC定义文件中包括以下3个关键信息
         1）目标Pod的定义
         2）目标Pod需要运行的副本数量
         3) 要监控的目标Pod的标签
         在创建好RC(系统自动创建Pod)后，K8s会通过RC中定义的Label筛选出对应的Pod实例并实时监控器状态和数量，如果实例数量少于定义的副本数量，则会根据RC中定义的Pod模板来创建一个新的Pod,然后将此Pod调度到合适的Node上启动运行，直到Pod实例的数量达到预定目标，这个过程完全是自动化的，无需人工干预。有了RC，服务的扩容就编程了一个纯粹的数字游戏了，只要修改RC中副本的数量即可，后续的Service升级也将通过RC来自动完成。
</pre>

<pre>
为什么使用Kubernetes
   K8s作为当前唯一被业界广泛任何和看好的Docker分布式系统解决方案，K8s有什么好处呢？
   1）轻装上阵
   2）K8s全面拥抱微服务架构。微服务架构的核心是将一个巨大的单体应用拆分为很多小的互相连接的微服务，一个微服务背后可能有多个实例副本在支撑，副本的数量可能会随着系统的负荷变化而进行调整，内嵌的负载均衡器在这里发挥了重要作用，微服务架构使得每个服务都可以由专门的开发团队来开发，开发者可以自由选择开发 技术，这对于大规模团队来说很有价值，另外每个微服务独立开发，升级，扩容，因此系统具备很高的稳定和快速迭代进化能力，
   3）业务系统可以随时随地的整体搬迁到公有云上，K8s最初的设计目标就是运行在Google自家的公有云中，未来会支持更多的公有云及基于OpenStack的私有云，同时在K8s的架构方案中，底层网络的细节被完全屏蔽，基于服务的ClusterIP甚至都无须我们改变运行期的配置文件，就能将系统从屋里环境无缝迁移到公有云平台中，或者在服务高峰期将部分服务对应的Pod副本放入公有云中以提升系统的吞吐量，不仅节省了公司的硬件投入，还大大改善了客户体验。
   5）K8s系统架构具备了超强的横向扩容能力
</pre>
