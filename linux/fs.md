# 文件系统

第二层称为虚拟文件系统(VFS)层。VFS层有两个重要功能:

1. 它通过定义一个干净的VFS接口将文件系统通用操作与其实现分开。VFS接口的几种实现可以在同一台机器上共存，允许对本地安装的不同类型的文件系统进行透明访问。
2. 它提供了一种在整个网络中唯一表示文件的机制。VFS基于一种称为vnode的文件表示结构，它包含网络范围内唯一文件的数字指示符。(UNIX信息节点仅在单个文件系统中是唯一的。)这种网络范围的唯一性是支持网络文件系统所必需的。内核为每个活动节点(文件或目录)维护一个vnode结构。

因此，VFS区分本地文件和远程文件，本地文件根据它们的文件系统类型进一步区分。
VFS根据文件系统类型激活特定于文件系统的操作来处理本地请求，并为远程请求调用NFS协议过程(或其他网络文件系统的其他协议过程)。
文件句柄是由相关的vnodes构造的，并作为参数传递给这些过程。实现文件系统类型或远程文件系统协议的层是架构的第三层。
让我们简要考察一下Linux中的VFS架构。Linux VFS定义的四种主要对象类型是: