.. NOTE(stephenfin): If making changes to this file, ensure that the line
   numbers found in 'Documentation/intro/what-is-ovs' are kept up-to-date.

============
Open vSwitch
============

.. image:: https://travis-ci.org/openvswitch/ovs.png
    :target: https://travis-ci.org/openvswitch/ovs

什么事 Open vSwitch？
---------------------

Open vSwitch 是一个基于Apache 2 license的多层软件交换机。
我们的目标是实现一个支持标准管理接口的，开放转发功能以支持编程扩展和控制的交换机平台。

Open vSwitch 非常适合在VM环境中用作虚拟交换机。
除了将标准控制和可视化接口暴露给虚拟网络层外，它还旨在支持跨越多个物理服务器的分布式系统。
Open vSwitch 支持多种基于Linux的虚拟化技术，包括 Xen/XenServer, KVM, 及 VirtualBox。

大部分代码是平台无关的C编码，可以轻松地移植到其他环境中。
当前版本的 Open vSwitch 支持以下功能：

- 具有trunk和access口的标准 802.1Q VLAN 功能
- 上游交换机上连接或不链接LACP的NIC
- NetFlow, sFlow(R), 及镜像，以提高可视化
- QoS 配置，加上策列
- Geneve, GRE, VXLAN, STT, 及 LISP 隧道
- 802.1ag 连接故障管理
- OpenFlow 1.0 加大量扩展
- 具有C和Python绑定的事物配置数据库
- 使用Linux内核模块的高性能转发

Linux 内核模块支持 Linux 3.10及以上版本。

Open vSwitch 也可以完全在用户空间运行，无需内核模块协助。
这个用户空间实现应该比基于内核的交换机更容易进行移植。
用户空间中的OVS可以访问Linux或DPDK设备。

.. note::
    
	使用用户空间实现，且没有用DPDK做加速处理的OVS被认为是测试性的，具有性能成本。

这是什么？
------------

OVS的主要组成成分如下：

- ovs-vswitchd, 实现交换机的守护进程，与Linux内核模块共同实现基于流的报文交换。
- ovsdb-server, 一个轻量级的数据库，ovs-vswitchd 查询以获取其配置信息。
- ovs-dpctl, 用于配置交换机内核模块的工具。
- Citrix XenServer and Red Hat Enterprise Linux中用于构建RPM包的脚本和规范。
  XenServer RPMs 允许将Open vSwitch安装在Citrix XenServer主机上，作为替代其交换机的附加功能。
- ovs-vsctl, 用于查询和更新 ovs-vswitchd 的配置的实用程序。
- ovs-appctl, 一个向Open vSwitch守护进程发送命令的程序。

Open vSwitch 还提供了一些工具：

- ovs-ofctl, 用于查询和控制OpenFlow交换机和控制器的应用程序。
- ovs-pki, 用于创建和管理OpenFlow交换机的公钥设施的程序。
- ovs-testcontroller, 一个简单的OpenFlow控制器，对于测试非常有用(尽管不能用于生产)。
- tcpdump的补丁，使其能够解析OpenFlow消息。

还有那些可用的文件？
----------------------

.. TODO(stephenfin): Update with a link to the hosting site of the docs, once
   we know where that is

在常规Linux或FreeBSD上安装Open vSwitch请参阅 `installation guide <Documentation/intro/install/general.rst>`__ 。
特定平台的安装请参阅 `其他安装指南 <Documentation/intro/install/index.rst>`__ 。

常见问题及解答，请参阅 `FAQ <Documentation/faq>`__.

了解 Open vSwitch 的一些高级特性，请参阅 `教程 <Documentation/tutorials/ovs-advanced.rst>`__.

每个Open vSwitch用户空间程序都附带一个联机帮助页。许多帮助页是自己定义的，作为构建过程的一部分，我们建议您在构建帮助页面之前阅读。

Contact
-------

bugs@openvswitch.org
