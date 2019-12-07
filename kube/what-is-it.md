# Kubernetes: What is it

本页是 Kubernetes 的概述。

- 回到过去
- 你为什么需要 Kubernetes ，它能做什么
-  Kubernetes 不是什么
- 下一步是什么

Kubernetes是一个可移植的、可扩展的开源平台，用于管理集装箱化的工作负载和服务，它促进了声明性配置和自动化。它有一个巨大的、快速增长的生态系统。Kubernetes服务、支持和工具随处可见。

 Kubernetes 这个名字源于希腊语，意思是舵手或飞行员。谷歌在2014年开源了 Kubernetes 项目。 Kubernetes 基于谷歌15年来在大规模运行生产工作负载方面的经验，并结合了来自社区的最佳想法和实践。

让我们回到过去，看看为什么Kubernetes如此有用。

 

## 部署演变

传统部署时代：早期，组织在物理服务器上运行应用程序。无法为物理服务器中的应用程序定义资源边界，这导致了资源分配问题。例如，如果一台物理服务器上运行多个应用程序，可能会出现一个应用程序占用大部分资源的情况，结果，其他应用程序的性能会下降。对此的解决方案是在不同的物理服务器上运行每个应用程序。但是，这并没有扩展，因为资源利用不足，而且组织维护许多物理服务器的成本很高。

虚拟化部署时代：作为一种解决方案，引入了虚拟化。它允许您在单个物理服务器的中央处理器上运行多个虚拟机。虚拟化允许应用程序在虚拟机之间隔离，并提供一定程度的安全性，因为一个应用程序的信息不能被另一个应用程序自由访问。

虚拟化允许更好地利用物理服务器中的资源，并允许更好的可扩展性，因为应用程序可以轻松添加或更新，降低硬件成本，等等。借助虚拟化，您可以将一组物理资源呈现为一组一次性虚拟机。

每个虚拟机都是一台完整的机器，在虚拟化硬件之上运行所有组件，包括其自己的操作系统。

容器部署时代：容器与虚拟机相似，但是它们具有宽松的隔离属性，以便在应用程序之间共享操作系统。因此，容器被认为是轻量级的。与虚拟机类似，容器有自己的文件系统、中央处理器、内存、进程空间等。由于它们与底层基础架构分离，因此可以跨云和操作系统分布进行移植。

容器变得流行是因为它们提供了额外的好处，例如：

- 敏捷的应用程序创建和部署：与虚拟机映像使用相比，容器映像创建更加容易和高效。
- 持续开发、集成和部署：提供可靠和频繁的容器映像构建和部署，并具有快速和简单的回滚(由于映像不变性)。
- 关注点的开发和操作分离：在构建/发布时而不是部署时创建应用程序容器映像，从而将应用程序从基础架构中分离出来。
- 可观察性不仅显示操作系统级别的信息和度量，还显示应用程序运行状况和其他信号。
- 开发、测试和生产过程中的环境一致性：在笔记本电脑上运行与在云中相同。
- 云和操作系统分发的可移植性：运行在Ubuntu、RHEL、核心操作系统、本地、谷歌 Kubernetes 引擎和其他任何地方。
- 以应用程序为中心的管理：将抽象级别从在虚拟硬件上运行操作系统提升到使用逻辑资源在操作系统上运行应用程序。
- 松散耦合、分布式、弹性、自由的微服务：应用程序被分解成更小、独立的部分，可以动态部署和管理——而不是运行在一台大型单用途机器上的单一堆栈。
- 资源隔离：可预测的应用程序性能。
- 资源利用：高效率和高密度。

 

## 你为什么需要Kubernetes，它能做什么

容器是打包和运行应用程序的好方法。在生产环境中，您需要管理运行应用程序的容器，并确保没有停机时间。例如，如果一个容器倒下，需要启动另一个容器。如果这种行为由一个系统来处理，不是更容易吗？

Kubernetes就是这样来营救的！Kubernetes为您提供了一个弹性运行分布式系统的框架。它负责应用程序的扩展和故障转移，提供部署模式等。例如，Kubernetes可以轻松地为您的系统管理金丝雀部署。

 Kubernetes 为您提供：

- 服务发现和负载平衡

Kubernetes 可以使用域名或自己的IP地址公开容器。如果到容器的流量很高，Kubernetes能够平衡负载并分配网络流量，从而使部署稳定。

- 存储编排

Kubernetes允许您自动安装您选择的存储系统，如本地存储、公共云提供商等。

- 自动展开和回滚

您可以使用Kubernetes描述您部署的容器的期望状态，并且它可以以受控的速率将实际状态更改为期望状态。例如，您可以自动化Kubernetes来为您的部署创建新的容器，移除现有的容器，并将它们的所有资源应用到新的容器中。

自动装箱

您为Kubernetes提供了一组节点，它可以使用这些节点来运行容器化任务。你告诉Kubernetes每个容器需要多少中央处理器和内存。Kubernetes可以将容器安装到您的节点上，以最大限度地利用您的资源。

- 自我康复

Kubernetes重新启动失败的容器，替换容器，杀死不响应用户定义的健康检查的容器，并且在它们准备好服务之前不会向客户端通告它们。

- Secrets和配置管理

Kubernetes允许您存储和管理敏感信息，例如密码、OAuth令牌和ssh密钥。您可以部署和更新机密和应用程序配置，而无需重建容器映像，也无需暴露堆栈配置中的机密。

 Kubernetes 不是什么

 

Kubernetes 不是一个传统的、包罗万象的平台即服务系统。由于Kubernetes在容器级别而不是硬件级别运行，它提供了一些PaaS产品通用的普遍适用的功能，例如部署、扩展、负载平衡、日志记录和监控。然而，Kubernetes不是单片的，这些默认解决方案是可选的和可插拔的。Kubernetes为构建开发平台提供了构建模块，但在重要的地方保留了用户选择和灵活性。

Kubernetes：

- 不限制支持的应用程序类型。

Kubernetes旨在支持极其多样的工作负载，包括无状态、有状态和数据处理工作负载。如果一个应用程序可以在容器中运行，它应该在Kubernetes上运行得很好。

- 不部署源代码，也不构建应用程序。

持续集成、交付和部署(配置项/光盘)工作流由组织文化和偏好以及技术要求决定。

- 不提供应用程序级服务，例如中间件(例如，消息总线)、数据处理框架(例如，Spark)、数据库(例如，mysql)、缓存或集群存储系统(例如，Ceph)作为内置服务。

这些组件可以在Kubernetes运行，和/或可以由Kubernetes运行的应用程序通过可移植机制（如开放服务代理（Open Service Broker））访问。

- 不规定日志记录、监控或警报解决方案。

它提供了一些集成作为概念的证明，以及收集和导出度量的机制。

- 既不提供也不要求配置语言/系统(例如jsonnet)。

它提供了一个声明性的API，可以被任意形式的声明性规范作为目标。

- 不提供也不采用任何全面的机器配置、维护、管理或自愈系统。

- 此外，Kubernetes不仅仅是一个编排系统。事实上，它消除了编排的需要。

编排的技术定义是执行已定义的工作流：先做A，然后做B，然后做c。相比之下，Kubernetes 包含一组独立的、可组合的控制过程，这些过程持续地将当前状态驱动到所提供的期望状态。从交流到交流不应该有什么关系，也不需要集中控制。这导致了一个更易于使用、更强大、更健壮、更有弹性和更可扩展的系统。

 