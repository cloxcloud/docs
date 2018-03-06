.. include:: ../vars.rst
.. _pine64_the_not_so_short_road_node_installation:

==========================================
Node Installation
==========================================

This guide intends to describe the process of creating a compute node based on |distro| on a |SBC_model|.  You will need a microSD card with at least 4GB and preferably class 10 or better.

.. warning::
    Please note that there are images ready to be dumped on a microSD card that will save you the trouble of this guide. This guide is only intended to show what is inside those images and for those who want to contribute and improve this process.

.. note::
    * Read **Notes** sections attached to some steps, before using the shell
    * Commands prefixed by "**#**" are meant to be run as root. Commands prefixed by "**$**" must be run as a normal user.
    * ``(...)`` in code snippets means that could be code before/after the modified lines. That portion of code need to stay unmodified.


1. Install |distro| on |SBC_model|
=========================================

Download **xenial-minimal-pine64** from `<https://github.com/ayufan-pine64/linux-build/releases/>`_. Dump it to your microSD card and perform a **dist-upgrade**.

2. Install the required software
=====================================

2.1. Install required packages
------------------------------------------------

.. prompt:: bash # auto

    # apt install lxd lxd-tools python-pylxd/xenial-updates python-ws4py python-pip bridge-utils git ruby genisoimage nfs-common libvncserver1 libjpeg62 qemu-utils
    # pip install idna==2.5
    # pip install setuptools
    # pip install isoparser


3. Initial Configurations
============================

3.1. Configure network bridge
---------------------------------------------

Now let us configure the bridge that will be used by all containers deployed. Configure it in
**/etc/network/interfaces**. Your configuration should look something like this:

.. code-block:: bash

    auto lo
    iface lo inet loopback

    #auto eth0
    #iface eth0 inet dhcp
    
    auto br0
    iface br0 inet static
            address 10.8.2.38
            netmask 255.255.255.0
            gateway 10.8.2.1
            bridge_ports eth0
            bridge_fd 0
            bridge_maxwait 0

Reboot to apply changes:

.. prompt:: bash # auto

    # reboot

3.2. Initialize LXD
------------------------------

First, start and enable LXD's service:

.. prompt:: bash # auto

    # systemctl start lxd
    # systemctl enable lxd

Provide initial configuration for LXD:

.. prompt:: bash # auto

    # lxd init

Say **yes** to **Create a new datastore** and set **type** to **dir**. Don't allow remote connections to LXD. On the question, "**Do you want to configure the LXD bridge?**", answer "**no**". We won't use that bridge, instead we will let OpenNebula to do this job.

3.2.1. Modify Network inside LXD default profile
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Containers inherit properties from a profile. The default profile contains a network device, we'll remove this one as
it's not managed by OpenNebula.

.. prompt:: bash # auto

    # lxc profile device remove default eth0


4. Configure the |SBC_model| as a node for OpenNebula
==================================================

4.1. Install OpenNebula node packages and dependencies
-----------------------------------------------------------------------------------

Download  `opennebula-common package <http://downloads.opennebula.org/repo/5.4/Ubuntu/16.04/pool/opennebula/opennebula-common_5.4.1-1_all.deb>`_
and install it:

.. prompt:: bash # auto

    # dpkg -i opennebula-common_5.4.1-1_all.deb


4.2. Configure oneadmin user to correctly use lxd
--------------------------------------------------------------------------

Assign sudo permission to oneadmin and add it to lxd group

.. prompt:: bash # auto

    # echo "oneadmin ALL= NOPASSWD: ALL" >> /etc/sudoers
    # usermod -a -G lxd oneadmin

4.3. Loopback devices
-------------------------------------------------------------
Every file system image used by LXDoNe will require one loop device. The default kernel's limit for loop devices is 8. To increase the loopback device:

.. prompt:: bash # auto

    # echo "options loop max_loop=128" >> /etc/modprobe.d/local-loop.conf
    # echo "loop" >> /etc/modules-load.d/modules.conf
    # depmod

4.5. Install VNC server
-------------------------------------

We compiled and provided it for |distro| |distro_version| in our releases. Download `svncterm`_ for this architecture, install dependencies and install svncterm.

.. prompt:: bash # auto
    
    # apt install libjpeg62 libvncserver1
    # dpkg -i svncterm_1.2-1_arm64.deb

4.5. Configure Passwordless SSH
-------------------------------------------------------------

OpenNebula Front-end connects to the hypervisor Hosts using SSH. You must distribute the public key of ``oneadmin`` user
from all machines to the file ``/var/lib/one/.ssh/authorized_keys`` in all the machines. There are many methods to
achieve the distribution of the SSH keys, ultimately the administrator should choose a method (the recommendation is to
use a configuration management system). In this guide we are going to manually scp the SSH keys.

When the package was installed in the Front-end, an SSH key was generated and the ``authorized_keys`` populated.
We will sync the ``id_rsa``, ``id_rsa.pub`` and ``authorized_keys`` from the Front-end to the nodes. Additionally we
need to create a ``known_hosts`` file and sync it as well to the nodes. To create the ``known_hosts`` file, we have to
execute this command as user ``oneadmin`` in the Front-end with all the node names and the Front-end name as parameters:

.. prompt:: bash $ auto

    $ ssh-keyscan <frontend> <node1> <node2> <node3> ... >> /var/lib/one/.ssh/known_hosts

Now we need to copy the directory ``/var/lib/one/.ssh`` to all the nodes. The easiest way is to set a temporary password
to ``oneadmin`` in all the hosts and copy the directory from the Front-end:

.. prompt:: bash $ auto

    $ scp -rp /var/lib/one/.ssh <node1>:/var/lib/one/
    $ scp -rp /var/lib/one/.ssh <node2>:/var/lib/one/
    $ scp -rp /var/lib/one/.ssh <node3>:/var/lib/one/
    $ ...

You should verify that connecting from the Front-end, as user ``oneadmin``, to the nodes and the Front-end itself, and
from the nodes to the Front-end, does not ask password:

.. prompt:: bash $ auto

    $ ssh <frontend>
    $ exit

    $ ssh <node1>
    $ ssh <frontend>
    $ exit
    $ exit

    $ ssh <node2>
    $ ssh <frontend>
    $ exit
    $ exit

    $ ssh <node3>
    $ ssh <frontend>
    $ exit
    $ exit

5. Connect
==============

Now you are done and ready to connect the new node to your cloud. I know, I know, it's easier to download a pre-made
image from clox.org but it always feels better to do it yourself. Either that or you are feeling really annoyed knowing
all the time you could have saved yourself with the pre-made image :)
