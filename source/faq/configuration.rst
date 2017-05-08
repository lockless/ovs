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

===================
Basic Configuration
===================

Q: How do I configure a port as an access port?

    A. Add ``tag=VLAN`` to your ``ovs-vsctl add-port`` command. For example,
    the following commands configure br0 with eth0 as a trunk port (the
    default) and tap0 as an access port for VLAN 9:

    ::

        $ ovs-vsctl add-br br0
        $ ovs-vsctl add-port br0 eth0
        $ ovs-vsctl add-port br0 tap0 tag=9

    If you want to configure an already added port as an access port, use
    ``ovs-vsctl set``, e.g.:

    ::

        $ ovs-vsctl set port tap0 tag=9

Q: How do I configure a port as a SPAN port, that is, enable mirroring of all
traffic to that port?

    A. The following commands configure br0 with eth0 and tap0 as trunk ports.
    All traffic coming in or going out on eth0 or tap0 is also mirrored to
    tap1; any traffic arriving on tap1 is dropped:

    ::

        $ ovs-vsctl add-br br0
        $ ovs-vsctl add-port br0 eth0
        $ ovs-vsctl add-port br0 tap0
        $ ovs-vsctl add-port br0 tap1 \
            -- --id=@p get port tap1 \
            -- --id=@m create mirror name=m0 select-all=true output-port=@p \
            -- set bridge br0 mirrors=@m

    To later disable mirroring, run:

    ::

        $ ovs-vsctl clear bridge br0 mirrors

Q: Does Open vSwitch support configuring a port in promiscuous mode?

    A: Yes.  How you configure it depends on what you mean by "promiscuous
    mode":

    - Conventionally, "promiscuous mode" is a feature of a network interface
      card.  Ordinarily, a NIC passes to the CPU only the packets actually
      destined to its host machine.  It discards the rest to avoid wasting
      memory and CPU cycles.  When promiscuous mode is enabled, however, it
      passes every packet to the CPU.  On an old-style shared-media or
      hub-based network, this allows the host to spy on all packets on the
      network.  But in the switched networks that are almost everywhere these
      days, promiscuous mode doesn't have much effect, because few packets not
      destined to a host are delivered to the host's NIC.

      This form of promiscuous mode is configured in the guest OS of the VMs on
      your bridge, e.g. with "ifconfig".

    - The VMware vSwitch uses a different definition of "promiscuous mode".
      When you configure promiscuous mode on a VMware vNIC, the vSwitch sends a
      copy of every packet received by the vSwitch to that vNIC.  That has a
      much bigger effect than just enabling promiscuous mode in a guest OS.
      Rather than getting a few stray packets for which the switch does not yet
      know the correct destination, the vNIC gets every packet.  The effect is
      similar to replacing the vSwitch by a virtual hub.

      This "promiscuous mode" is what switches normally call "port mirroring"
      or "SPAN".  For information on how to configure SPAN, see "How do I
      configure a port as a SPAN port, that is, enable mirroring of all traffic
      to that port?"

Q: How do I configure a DPDK port as an access port?

    A: Firstly, you must have a DPDK-enabled version of Open vSwitch.

    If your version is DPDK-enabled it will support the other-config:dpdk-init
    configuration in the database and will display lines with "EAL:..." during
    startup when other_config:dpdk-init is set to 'true'.

    Secondly, when adding a DPDK port, unlike a system port, the type for the
    interface and valid dpdk-devargs must be specified. For example::

        $ ovs-vsctl add-br br0
        $ ovs-vsctl add-port br0 myportname -- set Interface myportname \
            type=dpdk options:dpdk-devargs=0000:06:00.0

    Refer to :doc:`/intro/install/dpdk` for more information on enabling and
    using DPDK with Open vSwitch.

Q: How do I configure a VLAN as an RSPAN VLAN, that is, enable mirroring of all
traffic to that VLAN?

    A: The following commands configure br0 with eth0 as a trunk port and tap0
    as an access port for VLAN 10.  All traffic coming in or going out on tap0,
    as well as traffic coming in or going out on eth0 in VLAN 10, is also
    mirrored to VLAN 15 on eth0.  The original tag for VLAN 10, in cases where
    one is present, is dropped as part of mirroring:

    ::

        $ ovs-vsctl add-br br0
        $ ovs-vsctl add-port br0 eth0
        $ ovs-vsctl add-port br0 tap0 tag=10
        $ ovs-vsctl \
            -- --id=@m create mirror name=m0 select-all=true select-vlan=10 \
               output-vlan=15 \
            -- set bridge br0 mirrors=@m

    To later disable mirroring, run:

    ::

        $ ovs-vsctl clear bridge br0 mirrors

    Mirroring to a VLAN can disrupt a network that contains unmanaged switches.
    See ovs-vswitchd.conf.db(5) for details. Mirroring to a GRE tunnel has
    fewer caveats than mirroring to a VLAN and should generally be preferred.

Q: Can I mirror more than one input VLAN to an RSPAN VLAN?

    A: Yes, but mirroring to a VLAN strips the original VLAN tag in favor of
    the specified output-vlan.  This loss of information may make the mirrored
    traffic too hard to interpret.

    To mirror multiple VLANs, use the commands above, but specify a
    comma-separated list of VLANs as the value for select-vlan.  To mirror
    every VLAN, use the commands above, but omit select-vlan and its value
    entirely.

    When a packet arrives on a VLAN that is used as a mirror output VLAN, the
    mirror is disregarded.  Instead, in standalone mode, OVS floods the packet
    across all the ports for which the mirror output VLAN is configured.  (If
    an OpenFlow controller is in use, then it can override this behavior
    through the flow table.)  If OVS is used as an intermediate switch, rather
    than an edge switch, this ensures that the RSPAN traffic is distributed
    through the network.

    Mirroring to a VLAN can disrupt a network that contains unmanaged switches.
    See ovs-vswitchd.conf.db(5) for details.  Mirroring to a GRE tunnel has
    fewer caveats than mirroring to a VLAN and should generally be preferred.

Q: How do I configure mirroring of all traffic to a GRE tunnel?

    A: The following commands configure br0 with eth0 and tap0 as trunk ports.
    All traffic coming in or going out on eth0 or tap0 is also mirrored to
    gre0, a GRE tunnel to the remote host 192.168.1.10; any traffic arriving on
    gre0 is dropped::

        $ ovs-vsctl add-br br0
        $ ovs-vsctl add-port br0 eth0
        $ ovs-vsctl add-port br0 tap0
        $ ovs-vsctl add-port br0 gre0 \
             -- set interface gre0 type=gre options:remote_ip=192.168.1.10 \
             -- --id=@p get port gre0 \
             -- --id=@m create mirror name=m0 select-all=true output-port=@p \
             -- set bridge br0 mirrors=@m

    To later disable mirroring and destroy the GRE tunnel::

        $ ovs-vsctl clear bridge br0 mirrors
        $ ovs-vsctl del-port br0 gre0

Q: Does Open vSwitch support ERSPAN?

    A: No.  As an alternative, Open vSwitch supports mirroring to a GRE tunnel
    (see above).

Q: How do I connect two bridges?

    A: First, why do you want to do this?  Two connected bridges are not much
    different from a single bridge, so you might as well just have a single
    bridge with all your ports on it.

    If you still want to connect two bridges, you can use a pair of patch
    ports.  The following example creates bridges br0 and br1, adds eth0 and
    tap0 to br0, adds tap1 to br1, and then connects br0 and br1 with a pair of
    patch ports.

    ::

        $ ovs-vsctl add-br br0
        $ ovs-vsctl add-port br0 eth0
        $ ovs-vsctl add-port br0 tap0
        $ ovs-vsctl add-br br1
        $ ovs-vsctl add-port br1 tap1
        $ ovs-vsctl \
            -- add-port br0 patch0 \
            -- set interface patch0 type=patch options:peer=patch1 \
            -- add-port br1 patch1 \
            -- set interface patch1 type=patch options:peer=patch0

    Bridges connected with patch ports are much like a single bridge. For
    instance, if the example above also added eth1 to br1, and both eth0 and
    eth1 happened to be connected to the same next-hop switch, then you could
    loop your network just as you would if you added eth0 and eth1 to the same
    bridge (see the "Configuration Problems" section below for more
    information).

    If you are using Open vSwitch 1.9 or an earlier version, then you need to
    be using the kernel module bundled with Open vSwitch rather than the one
    that is integrated into Linux 3.3 and later, because Open vSwitch 1.9 and
    earlier versions need kernel support for patch ports. This also means that
    in Open vSwitch 1.9 and earlier, patch ports will not work with the
    userspace datapath, only with the kernel module.

Q: How do I configure a bridge without an OpenFlow local port?  (Local port in
the sense of OFPP_LOCAL)

    A: Open vSwitch does not support such a configuration.  Bridges always have
    their local ports.
