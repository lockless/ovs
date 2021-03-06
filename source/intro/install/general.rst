..
      Licensed under the Apache License, Version 2.0 (the "License"); you may
      not use this file except in compliance with the License. You may obtain
      a copy of the License at

          http://www.apache.org/licenses/LICENSE-2.0

      Unless required by applicable law or agreed to in writing, software
      distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
      WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
      License for the specific language governing permissions and limitations
      under the License.

      Convention for heading levels in Open vSwitch documentation:

      =======  Heading 0 (reserved for the title in a document)
      -------  Heading 1
      ~~~~~~~  Heading 2
      +++++++  Heading 3
      '''''''  Heading 4

      Avoid deeper levels because they do not render well.

==================================================
Open vSwitch 安装于 Linux 系统--FreeBSD 和 NetBSD
==================================================

本文档介绍如何在通用的 Linux，FreeBSD 或 NetBSD 主机上构建和安装 Open vSwitch。
其他特定平台上的安装信息，请参阅  :doc:`index` 中列出的其他安装指南。

获取 Open vSwitch 源码
------------------------

Open vSwitch 源码可以在其 Git 库中下载，使用如下命令::

    $ git clone https://github.com/openvswitch/ovs.git

请从 "master" 分支中开出，这是当前正在开发的最新代码。
当然，如果你需要一个已经发布的版本，也可以参照如下命令，下载指定版本的代码::

    $ git checkout v2.7.0

代码库还为每个版本系列分配一个分支。
例如，要获取 Open vSwitch 2.7.x 发布版本的最新修订程序，这些修订可能包含在最新的发布版本中未修订的bug，
你可以使用如下命令::

    $ git checkout origin/branch-2.7

如果不想使用 Git，你可以通过 http://openvswitch.org/download/ 获取Open vSwitch发布版本的压缩包，
或者从 https://github.com/openvswitch/ovs 的Web界面下载。

.. _general-build-reqs:

构建依赖
----------

要在 Open vSwitch 发行版本环境中编译用户空间程序，你可能需要如下软件：

- GNU make

- C 编译器，如：

  - GCC 4.6 及以上版本

  - Clang 3.4 及以上版本

  - MSVC 2013，参考 :doc:`windows` 获取更多 Windows 环境构建说明。

  .. note::

    虽然OVS可能与其他编译器兼容，但是对原子操作的最佳支持可能丢失，从而使得OVS非常慢（参考 ``lib/ovs-atomic.h`` ）.

- libssl，来自OpenSSL，可选库，但是如果打算将 Open vSwitch 连接到 OpenFlow 控制器，则建议使用。
  这种情况需要使用 libssl 在 Open vSwitch 和 OpenFlow 控制器的连接中建立机密性和真实性。
  如果安装了 libssl，那么 Open vSwitch 将自动构建并支持它。

- libcap-ng，由 Steve Grubb 编写，是可选的，但是建议使用。
  需要以具有非root权限的用户运行OVS守护进程，
  如果安装了 libcap-ng，那么 Open vSwitch 将自动构建并支持它。

- Python 2.7，还必须使用 Python 库1.4.0或更高的版本。


在 Linux 中，你可以选择编译 Open vSwitch 发布版本中附带的内核模块，或者使用Linux内核(3.3或更高版本)中内置的内核模块。
参阅 :doc:`/faq/index` 中描述的问题 "Open vSwitch 内核数据路径中哪些功能不是Linux内核原生的一部分?"获取更多的信息。
你还可以仅使用用户空间实现，当然，需要以功能或性能上的某些消耗代价。参阅 :doc:`userspace` 获取更多信息。

要在Linux上编译内核模块，你还需要安装如下内容：

- 支持的Linux内核版本

  不论是内置还是作为独立模块，为了支持ingress policing，必须使能内核配置选项 ``NET_CLS_BASIC`` 、 ``NET_SCH_INGRESS`` 和 ``NET_ACT_POLICE`` 。
  ``NET_CLS_POLICE`` 已经过时了，现在已经不再使用。

  在内核3.11以前的版本， ``ip_gre`` 模块，用于支持 GRE tunnels over IP
  (``NET_IPGRE``)，不能加载或编译。

  要使用Open vSwitch配置 HTB 或 HFSC 服务质量，必须使能相应的配置选项。

  要为TAP设备开启Open vSwitch支持，需要配置 ``CONFIG_TUN`` 。

- 要构建一个内核模块，你需要用于构建该内核的GCC版本。

- 内核构建目录，对应于该模块要运行的Linux内核镜像。
  例如，在Debian 和 Ubuntu中，包含内核二进制程序的linux-image软件包都具有相应的所需构建基础架构的Linux头文件包。

如果使用 Git 目录开发（而不是代码压缩包），后者你需要修改 Open vSwitch 构建系统及数据库架构，那么还需要如下软件：

- Autoconf 2.63 及更新版本。

- Automake 1.10 及更新版本。

- libtool 2.4 及更新版本。(老版本或许也能正常工作)

如果要运行测试单元，需要：

- Perl. 版本 5.10.1 是已知的可以正常工作的版本。早期版本也可以正常工作。

用户空间及Linux内核数据路径测试还依赖于：

- pyftpdlib. 版本 1.2.0。早期版本可能也能正常工作。

- GNU wget. 版本 1.16。早期版本可能也能正常工作。

- netcat，常见的几种实现方式都可以使用。

- curl. 版本 7.47.0。早期的版本也可以工作。

- tftpy. 版本 0.6.2。早期的版本也可以工作。

ovs-vswitchd.conf.db 联机帮助页面将包含除文本之外的 E-R 图，只需要安装如下软件：

- dot 来自 graphviz (http://www.graphviz.org/)

- Perl. 版本 5.10.1。早期的版本也可以工作。

如果需要大量修改 Open vSwitch，请安装如下软件以获取更好的提示：

- "sparse" 0.4.4 或更新的版本(https://www.kernel.org/pub/software/devel/sparse/dist/)

- GNU make

- clang, 3.4 或更新的版本

- flake8 及 flake8 插件 (用于Python代码)。针对Python代码运行自动 flake8 检查启用了一些来自插件的警告选项。如果没有安装，只有在安装了 "hacking" 的系统上才会有警告提示。

你可能会发现 ``utilities/ovs-dev.py`` 中的ovs-dev脚本很有帮助。

.. _general-install-reqs:

安装依赖
----------

你用来构建 Open vSwitch 的设备可能并不是 Open vSwitch真正要运行的设备。
要简单的安装并运行 Open vSwitch，你需要如下软件：

- 与构建设备兼容的共享库。

- 在Linux上，如果要使用基于内核的datapath（这是最常见的用例），那么内核就要有一个兼容的模块。这可以是使用Open vSwitch构建的内核模块（例如上一步），也可以是Linux 3.3及更高版本所附带的内核模块。Open vSwitch 功能和性能可能会因模块和内核而异。有关更多信息，请参阅 :doc:`/faq/releases` 。

- For optional support of ingress policing on Linux, the "tc" program
  from iproute2 (part of all major distributions and available at
  https://wiki.linuxfoundation.org/networking/iproute2).

- Python 2.7，同时必须有 Python six library 1.4.0版本或更新版本。

在Linux上，必须保证 ``/dev/urandom`` 文件存在。为了支持 TAP 设备，必须保证 ``/dev/net/tun`` 文件存在。

.. _general-bootstrapping:

引导
------

假如你已经下载了发布版本压缩包，这一步可以省略。
如果是直接从 Open vSwitch Git 树中获取源代码，或者获得 Git 树快照，则需要在顶层目录中运行脚本 boot.sh 以构建 "configure" 脚本::

    $ ./boot.sh

.. _general-configuring:

配置
------

通过运行 configure 脚本来配置软件。
运行 configure 脚本通常可以不加任何参数。如::

    $ ./configure

默认情况下，所有文件都安装在 ``/usr/local`` 。
默认情况下，Open vSwitch 也期望在 ``/usr/local/etc/openvswitch`` 中存储其数据库。
如果需要将文件安装到其他目录，如 ``/usr`` 及 ``/var`` 来替换默认的 ``/usr/local`` 和 ``/usr/local/var`` ，
且期望将 ``/etc/openvswitch`` 作为默认的数据库目录，添加如下的配置选项::

    $ ./configure --prefix=/usr --localstatedir=/var --sysconfdir=/etc

.. note::

  Open vSwitch 使用上面的配置选项，通过安装包来安装，类似于 .rpm (如通过命令 ``yum install`` 或 ``rpm -ivh``) 和
  .deb (如通过命令 ``apt-get install`` 或 ``dpkg -i``)。

默认情况下，建立静态库并将其链接起来。
如果想要使用共享库::

    $ ./configure --enable-shared

如果要使用特定的 C 编译器来编译 Open vSwitch 用户程序，需要在 configure 命令中指定，如::

    $ ./configure CC=gcc-4.2

如果要使用 'clang' 编译器::

    $ ./configure CC=clang

要想 C 编译器提供特殊的标志，请添加 ``CFLAGS`` 到 configure 命令行中。
如果想要使用 CFLAGS，包括 ``-g`` 以创建调试符号及 ``-O2`` 以启用优化，必须自己在命令行中指定。
例如，用 CFLAGS 包括 ``-mssse3`` 来构建，需要运行如下命令::

    $ ./configure CFLAGS="-g -O2 -mssse3"

为了使用高效的哈希计算，可以提供特殊的 flags 来使用内建函数。
例如，在支持 SSE4.2 指令集的 X86_64 上，CRC32 函数可以通过传递 ``-msse4.2``::

    $ ./configure CFLAGS="-g -O2 -msse4.2"`

如果你使用不同的处理器，不知道要选择什么 flags，建议使用 ``-march=native`` 设置::

    $ ./configure CFLAGS="-g -O2 -march=native"

这样，GCC 将检测处理器并自动设置适当的标志。
如果您在目标计算机外编译 OVS，则不应使用此方法。

.. note::
  构建 Linux 内核时不使用 CFLAGS。内核模块使用的自定义 CFLAGS 通过 ``EXTRA_CFLAGS`` 变量提供。例如::

      $ make EXTRA_CFLAGS="-Wno-error=date-time"

构架 Linux 内核模块，以便可以运行基于内核的交换机。构建时将内核构建目录的位置传递到 ``--with-linux``。例如::

    $ ./configure --with-linux=/lib/modules/$(uname -r)/build

.. note::
  如果 ``--with-linux`` 请求构建与不支持的 Linux 版本时，那么 ``configure`` 会失败，并显示错误消息。
  具体信息请参考 :doc:`/faq/index` 。

如果希望构建其他体系的内核模块，则可以在调用 configure 脚本时使用 KARCH 变量指定内核体系结构字符串。例如要构建 MIPS 架构::

    $ ./configure --with-linux=/path/to/linux KARCH=mips

如果打算做许多 Open vSwitch 开发，可以使用 ``--enable-Werror`` 将 ``-Werror`` 选项添加到编译器命令行上，将警告转为错误。
这将不会错误构建产生的警告。例如::

    $ ./configure --enable-Werror

要使用 gcov 代码覆盖支持，添加 ``--enable-coverage``::

    $ ./configure --enable-coverage

configure 脚本接受一些其他选项，并赋予额外的环境变量。完整的列表，使用 ``--help`` 来查看::

    $ ./configure --help

也可以在单独的构建目录中运行 configure。
如果要从单个源代码目录中以多种方式构建 Open vSwitch 就会使用到这种方式。如，尝试 GCC 和 Clang 构建，或者为多个 Linux 版本构建内核模块。
例如::

    $ mkdir _gcc && (cd _gcc && ./configure CC=gcc)
    $ mkdir _clang && (cd _clang && ./configure CC=clang)

在某些情况下，使用 jemalloc 内存分配器替代 glibc 内存分配器时，ovsdb-server 及其他组件的性能会更好。
要链接到 jemalloc， 将其添加到LIBS中::

    $ ./configure LIBS=-ljemalloc

.. _general-building:

构建
-----

1. 在构建目录中运行 GNU make 命令::

       $ make

   如果 GNU make 安装为 "gmake" 则运行::

       $ gmake

   如果使用的是分离的构建目录，运行 make 或 gmake 以生成构建目录::

       $ make -C _gcc
       $ make -C _clang

   如果安装了 ``sparse`` (参考 "Prerequisites")，为了改进警告信息，添加 ``C=1`` 到命令行上。

   .. note::
     Clang 和 ccache 的某些版本不完全兼容。如果一起使用时看到警告信息，请考虑禁用 ccache。

2. 考虑运行testsuite时，请参考 :doc:`/topics/testing` 。

3. 运行 ``make install`` 将可执行文件和联机帮助安装到正在运行的系统中，默认情况下在 ``/usr/local``::

       $ make install

5. 如果构建内核模块，通过下面命令来安装::

       $ make modules_install

   It is possible that you already had a Open vSwitch kernel module installed
   on your machine that came from upstream Linux (in a different directory). To
   make sure that you load the Open vSwitch kernel module you built from this
   repository, you should create a ``depmod.d`` file that prefers your newly
   installed kernel modules over the kernel modules from upstream Linux. The
   following snippet of code achieves the same::

       $ config_file="/etc/depmod.d/openvswitch.conf"
       $ for module in datapath/linux/*.ko; do
         modname="$(basename ${module})"
         echo "override ${modname%.ko} * extra" >> "$config_file"
         echo "override ${modname%.ko} * weak-updates" >> "$config_file"
         done
       $ depmod -a

   Finally, load the kernel modules that you need. e.g.::

       $ /sbin/modprobe openvswitch

   To verify that the modules have been loaded, run ``/sbin/lsmod`` and check
   that openvswitch is listed::

       $ /sbin/lsmod | grep openvswitch

   .. note::
     If the ``modprobe`` operation fails, look at the last few kernel log
     messages (e.g. with ``dmesg | tail``). Generally, issues like this occur
     when Open vSwitch is built for a kernel different from the one into which
     you are trying to load it.  Run ``modinfo`` on ``openvswitch.ko`` and on a
     module built for the running kernel, e.g.::

         $ /sbin/modinfo openvswitch.ko
         $ /sbin/modinfo /lib/modules/$(uname -r)/kernel/net/bridge/bridge.ko

     Compare the "vermagic" lines output by the two commands.  If they differ,
     then Open vSwitch was built for the wrong kernel.

     If you decide to report a bug or ask a question related to module loading,
     include the output from the ``dmesg`` and ``modinfo`` commands mentioned
     above.

.. _general-starting:

Starting
--------

On Unix-alike systems, such as BSDs and Linux, starting the Open vSwitch
suite of daemons is a simple process.  Open vSwitch includes a shell script,
and helpers, called ovs-ctl which automates much of the tasks for starting
and stopping ovsdb-server, and ovs-vswitchd.  After installation, the daemons
can be started by using the ovs-ctl utility.  This will take care to setup
initial conditions, and start the daemons in the correct order.  The ovs-ctl
utility is located in '$(pkgdatadir)/scripts', and defaults to
'/usr/local/share/openvswitch/scripts'.  An example after install might be::

    $ export PATH=$PATH:/usr/local/share/openvswitch/scripts
    $ ovs-ctl start

Additionally, the ovs-ctl script allows starting / stopping the daemons
individually using specific options.  To start just the ovsdb-server::

    $ export PATH=$PATH:/usr/local/share/openvswitch/scripts
    $ ovs-ctl --no-ovs-vswitchd start

Likewise, to start just the ovs-vswitchd::

    $ export PATH=$PATH:/usr/local/share/openvswitch/scripts
    $ ovs-ctl --no-ovsdb-server start

Refer to ovs-ctl(8) for more information on ovs-ctl.

In addition to using the automated script to start Open vSwitch, you may
wish to manually start the various daemons. Before starting ovs-vswitchd
itself, you need to start its configuration database, ovsdb-server. Each
machine on which Open vSwitch is installed should run its own copy of
ovsdb-server. Before ovsdb-server itself can be started, configure a
database that it can use::

       $ mkdir -p /usr/local/etc/openvswitch
       $ ovsdb-tool create /usr/local/etc/openvswitch/conf.db \
           vswitchd/vswitch.ovsschema

Configure ovsdb-server to use database created above, to listen on a Unix
domain socket, to connect to any managers specified in the database itself, and
to use the SSL configuration in the database::

    $ mkdir -p /usr/local/var/run/openvswitch
    $ ovsdb-server --remote=punix:/usr/local/var/run/openvswitch/db.sock \
        --remote=db:Open_vSwitch,Open_vSwitch,manager_options \
        --private-key=db:Open_vSwitch,SSL,private_key \
        --certificate=db:Open_vSwitch,SSL,certificate \
        --bootstrap-ca-cert=db:Open_vSwitch,SSL,ca_cert \
        --pidfile --detach --log-file

.. note::
  If you built Open vSwitch without SSL support, then omit ``--private-key``,
  ``--certificate``, and ``--bootstrap-ca-cert``.)

Initialize the database using ovs-vsctl. This is only necessary the first time
after you create the database with ovsdb-tool, though running it at any time is
harmless::

    $ ovs-vsctl --no-wait init

Start the main Open vSwitch daemon, telling it to connect to the same Unix
domain socket::

    $ ovs-vswitchd --pidfile --detach --log-file

Validating
----------

At this point you can use ovs-vsctl to set up bridges and other Open vSwitch
features.  For example, to create a bridge named ``br0`` and add ports ``eth0``
and ``vif1.0`` to it::

    $ ovs-vsctl add-br br0
    $ ovs-vsctl add-port br0 eth0
    $ ovs-vsctl add-port br0 vif1.0

Refer to ovs-vsctl(8) for more details. You may also wish to refer to
:doc:`/topics/testing` for information on more generic testing of OVS.

Upgrading
---------

When you upgrade Open vSwitch from one version to another you should also
upgrade the database schema:

.. note::
   The following manual steps may also be accomplished by using ovs-ctl to
   stop and start the daemons after upgrade.  The ovs-ctl script will
   automatically upgrade the schema.

1. Stop the Open vSwitch daemons, e.g.::

       $ kill `cd /usr/local/var/run/openvswitch && cat ovsdb-server.pid ovs-vswitchd.pid`

2. Install the new Open vSwitch release by using the same configure options as
   was used for installing the previous version. If you do not use the same
   configure options, you can end up with two different versions of Open
   vSwitch executables installed in different locations.

3. Upgrade the database, in one of the following two ways:

   -  If there is no important data in your database, then you may delete the
      database file and recreate it with ovsdb-tool, following the instructions
      under "Building and Installing Open vSwitch for Linux, FreeBSD or NetBSD".

   -  If you want to preserve the contents of your database, back it up first,
      then use ``ovsdb-tool convert`` to upgrade it, e.g.::

          $ ovsdb-tool convert /usr/local/etc/openvswitch/conf.db \
              vswitchd/vswitch.ovsschema

4. Start the Open vSwitch daemons as described under `Starting`_ above.

Hot Upgrading
-------------

Upgrading Open vSwitch from one version to the next version with minimum
disruption of traffic going through the system that is using that Open vSwitch
needs some considerations:

1. If the upgrade only involves upgrading the userspace utilities and daemons
   of Open vSwitch, make sure that the new userspace version is compatible with
   the previously loaded kernel module.

2. An upgrade of userspace daemons means that they have to be restarted.
   Restarting the daemons means that the OpenFlow flows in the ovs-vswitchd
   daemon will be lost. One way to restore the flows is to let the controller
   re-populate it. Another way is to save the previous flows using a utility
   like ovs-ofctl and then re-add them after the restart. Restoring the old
   flows is accurate only if the new Open vSwitch interfaces retain the old
   'ofport' values.

3. When the new userspace daemons get restarted, they automatically flush the
   old flows setup in the kernel. This can be expensive if there are hundreds
   of new flows that are entering the kernel but userspace daemons are busy
   setting up new userspace flows from either the controller or an utility like
   ovs-ofctl. Open vSwitch database provides an option to solve this problem
   through the ``other_config:flow-restore-wait`` column of the
   ``Open_vSwitch`` table. Refer to the ovs-vswitchd.conf.db(5) manpage for
   details.

4. If the upgrade also involves upgrading the kernel module, the old kernel
   module needs to be unloaded and the new kernel module should be loaded. This
   means that the kernel network devices belonging to Open vSwitch is recreated
   and the kernel flows are lost. The downtime of the traffic can be reduced if
   the userspace daemons are restarted immediately and the userspace flows are
   restored as soon as possible.

The ovs-ctl utility's ``restart`` function only restarts the userspace daemons,
makes sure that the 'ofport' values remain consistent across restarts, restores
userspace flows using the ovs-ofctl utility and also uses the
``other_config:flow-restore-wait`` column to keep the traffic downtime to the
minimum. The ovs-ctl utility's ``force-reload-kmod`` function does all of the
above, but also replaces the old kernel module with the new one. Open vSwitch
startup scripts for Debian, XenServer and RHEL use ovs-ctl's functions and it
is recommended that these functions be used for other software platforms too.

Reporting Bugs
--------------

Report problems to bugs@openvswitch.org.
