# GPU

Kubernetes是一个开源平台，用于自动化部署、扩展和管理集装箱化应用程序。Kubernetes软件包括对GPU的支持和Kubernetes软件的增强，因此用户可以轻松配置和使用GPU资源来加速工作负载，如深度学习。本文描述了使用NVIDIA支持的组件(如驱动程序和运行时)安装上游Kubernetes的两种分步方法，以及使用NVIDIA GPUs的两种方法——一种使用DeepOps的方法和一种使用Kubeadm的方法。

要在集群中设置编排和调度，强烈建议您使用DeepOps。DeepOps是一个模块化的ansible脚本集合，可以在您的节点上自动部署Kubernetes、Slurm或两者的混合组合。它还安装了必要的图形处理器驱动程序、图形处理器容器运行时(nvidia-docker2)以及图形处理器加速工作的各种其他依赖项。封装了英伟达GPU的最佳实践，可以根据需要定制或作为单独的组件运行。

Kubernetes可以通过不同的机制部署。使用DeepOps来自动部署，尤其是对于许多工作节点的集群。使用以下步骤使用DeepOps安装Kubernetes:

1. 选择要从中部署的资源调配节点。
   这是DeepOps Ansible脚本运行的地方，通常是一台与目标集群有连接的开发笔记本电脑。在此资源调配节点上，使用以下命令克隆DeepOps仓库。

   `$ git clone https://github.com/nvidia/deepops.git`

2. 或者，使用以下命令签出最近的发行标签 release tag。如果您没有明确使用发布 release tag，那么将使用最新的开发代码，而不是正式发布的代码。
3. 