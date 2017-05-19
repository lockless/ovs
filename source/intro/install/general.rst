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

- Python 2.7. You must also have the Python six library version 1.4.0
  or later.

On Linux you should ensure that ``/dev/urandom`` exists. To support TAP
devices, you must also ensure that ``/dev/net/tun`` exists.

.. _general-bootstrapping:

Bootstrapping
-------------

This step is not needed if you have downloaded a released tarball. If
you pulled the sources directly from an Open vSwitch Git tree or got a
Git tree snapshot, then run boot.sh in the top source directory to build
the "configure" script::

    $ ./boot.sh

.. _general-configuring:

Configuring
-----------

Configure the package by running the configure script. You can usually
invoke configure without any arguments. For example::

    $ ./configure

By default all files are installed under ``/usr/local``. Open vSwitch also
expects to find its database in ``/usr/local/etc/openvswitch`` by default. If
you want to install all files into, e.g., ``/usr`` and ``/var`` instead of
``/usr/local`` and ``/usr/local/var`` and expect to use ``/etc/openvswitch`` as
the default database directory, add options as shown here::

    $ ./configure --prefix=/usr --localstatedir=/var --sysconfdir=/etc

.. note::

  Open vSwitch installed with packages like .rpm (e.g. via ``yum install`` or
  ``rpm -ivh``) and .deb (e.g. via ``apt-get install`` or ``dpkg -i``) use the
  above configure options.

By default, static libraries are built and linked against. If you want to use
shared libraries instead::

    $ ./configure --enable-shared

To use a specific C compiler for compiling Open vSwitch user programs, also
specify it on the configure command line, like so::

    $ ./configure CC=gcc-4.2

To use 'clang' compiler::

    $ ./configure CC=clang

To supply special flags to the C compiler, specify them as ``CFLAGS`` on the
configure command line. If you want the default CFLAGS, which include ``-g`` to
build debug symbols and ``-O2`` to enable optimizations, you must include them
yourself. For example, to build with the default CFLAGS plus ``-mssse3``, you
might run configure as follows::

    $ ./configure CFLAGS="-g -O2 -mssse3"

For efficient hash computation special flags can be passed to leverage built-in
intrinsics. For example on X86_64 with SSE4.2 instruction set support, CRC32
intrinsics can be used by passing ``-msse4.2``::

    $ ./configure CFLAGS="-g -O2 -msse4.2"`

If you are on a different processor and don't know what flags to choose, it is
recommended to use ``-march=native`` settings::

    $ ./configure CFLAGS="-g -O2 -march=native"

With this, GCC will detect the processor and automatically set appropriate
flags for it. This should not be used if you are compiling OVS outside the
target machine.

.. note::
  CFLAGS are not applied when building the Linux kernel module. Custom CFLAGS
  for the kernel module are supplied using the ``EXTRA_CFLAGS`` variable when
  running make. For example::

      $ make EXTRA_CFLAGS="-Wno-error=date-time"

To build the Linux kernel module, so that you can run the kernel-based switch,
pass the location of the kernel build directory on ``--with-linux``. For
example, to build for a running instance of Linux::

    $ ./configure --with-linux=/lib/modules/$(uname -r)/build

.. note::
  If ``--with-linux`` requests building for an unsupported version of Linux,
  then ``configure`` will fail with an error message. Refer to the
  :doc:`/faq/index` for advice in that case.

If you wish to build the kernel module for an architecture other than the
architecture of the machine used for the build, you may specify the kernel
architecture string using the KARCH variable when invoking the configure
script. For example, to build for MIPS with Linux::

    $ ./configure --with-linux=/path/to/linux KARCH=mips

If you plan to do much Open vSwitch development, you might want to add
``--enable-Werror``, which adds the ``-Werror`` option to the compiler command
line, turning warnings into errors. That makes it impossible to miss warnings
generated by the build. For example::

    $ ./configure --enable-Werror

To build with gcov code coverage support, add ``--enable-coverage``::

    $ ./configure --enable-coverage

The configure script accepts a number of other options and honors additional
environment variables. For a full list, invoke configure with the ``--help``
option::

    $ ./configure --help

You can also run configure from a separate build directory. This is helpful if
you want to build Open vSwitch in more than one way from a single source
directory, e.g. to try out both GCC and Clang builds, or to build kernel
modules for more than one Linux version. For example::

    $ mkdir _gcc && (cd _gcc && ./configure CC=gcc)
    $ mkdir _clang && (cd _clang && ./configure CC=clang)

Under certains loads the ovsdb-server and other components perform better when
using the jemalloc memory allocator, instead of the glibc memory allocator. If
you wish to link with jemalloc add it to LIBS::

    $ ./configure LIBS=-ljemalloc

.. _general-building:

Building
--------

1. Run GNU make in the build directory, e.g.::

       $ make

   or if GNU make is installed as "gmake"::

       $ gmake

   If you used a separate build directory, run make or gmake from that
   directory, e.g.::

       $ make -C _gcc
       $ make -C _clang

   For improved warnings if you installed ``sparse`` (see "Prerequisites"), add
   ``C=1`` to the command line.

   .. note::
     Some versions of Clang and ccache are not completely compatible. If you
     see unusual warnings when you use both together, consider disabling
     ccache.

2. Consider running the testsuite. Refer to :doc:`/topics/testing` for
   instructions.

3. Run ``make install`` to install the executables and manpages into the
   running system, by default under ``/usr/local``::

       $ make install

5. If you built kernel modules, you may install them, e.g.::

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
