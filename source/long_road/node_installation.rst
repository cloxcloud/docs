.. _the_not_so_short_road_node_installation:

==========================================
Node Installation
==========================================

This guide intends to describe the process of creating a compute node based on Raspbian on a Raspberry Pi 3.  You will need a microSD card with at least 4GB and preferably class 10 or better.

.. warning::
    Please note that there are images ready to be dumped on a microSD card that will save you the trouble of this guide. This guide is only intended to show what is inside those images and for those who want to contribute and improve this process.

.. note::
    * Read **Notes** sections attached to some steps, before using the shell
    * Commands prefixed by "**#**" are meant to be run as root. Commands prefixed by "**$**" must be run as a normal user.
    * ``(...)`` in code snippets means that could be code before/after the modified lines. That portion of code need to stay unmodified.


1. Install Raspbian on the Raspberry Pi
=========================================

You can check the steps here `<https://www.raspberrypi.org/documentation/installation/installing-images/>`__.

1.1. Upgrade to stretch
--------------------------------------
Change repositories and upgrade system:

.. prompt:: bash # auto

    # echo "deb http://archive.raspbian.org/raspbian/ stretch main contrib non-free rpi" > /etc/apt/sources.list
    # echo "deb http://archive.raspberrypi.org/debian/ stretch main ui" > /etc/apt/sources.list.d/raspi.list
    # apt update
    # apt upgrade

2. Install the required software
=====================================

2.1. Install required packages
------------------------------------------------

.. prompt:: bash # auto

    # apt install snapd bridge-utils python-pip python-ws4py git ruby genisoimage
    # pip install idna==2.5
    # pip install pylxd==2.0.5
    # pip install isoparser

2.2. Disable ``apparmor`` service
----------------------------------------------------

Apparmor can cause problems when installing packages via snaps. You can stop it and disable it from automatic start by
executing:

.. prompt:: bash # auto

    # systemctl stop apparmor.service
    # systemctl disable apparmor.service

2.3. Install LXD as Snap
-----------------------------------------

.. prompt:: bash # auto

    # snap install lxd --channel=2.0/stable

Some apps like **pylxd** and **LXDoNe** will assume lxd is installed on **/var/lib/lxd**. Because we are using LXD through
a snap, the installation folder is in another location, so we will create a symbolic link for compatibility with
**pylxd** and **LXDoNe**.

.. prompt:: bash # auto

    # ln -s /var/snap/lxd/common/lxd/ /var/lib/lxd

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

If you never used *snapd* before, you’ll need to update your **PATH** for using lxd as ``root`` user:

.. prompt:: bash # auto

    # echo "PATH=\"/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin\"" >> /etc/environment
    # source /etc/environment && export PATH


And for use it with *sudo* modify **/etc/sudoers** replacing:

.. code-block:: bash

    (...)
    Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    (...)

by:

.. code-block:: bash

    (...)
    Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/games:/usr/games:/snap/bin"
    (...)

Provide initial configuration for LXD:

.. prompt:: bash # auto

    # lxd init

Press enter on the first two questions to select default value. On the third question, "**Do you want to configure the LXD bridge?**", answer "**no**". We won't use that bridge, instead we will let OpenNebula to do this job.

3.2.1. Modify Network inside LXD default profile
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Containers inherit properties from a profile. The default profile contains a network device, we'll remove this one as
it's not managed by OpenNebula.

.. prompt:: bash # auto

    # lxc profile device remove default eth0


4. Configure the RPi as a node for OpenNebula
==================================================

4.1. Install OpenNebula node packages and dependencies
-----------------------------------------------------------------------------------

Download  `opennebula-common package <http://downloads.opennebula.org/repo/5.2/Debian/8/pool/opennebula/opennebula-common_5.2.1-1_all.deb>`_
and install it:

.. prompt:: bash # auto

    # dpkg -i opennebula-common_5.2.1-1_all.deb


4.2. Configure oneadmin user to correctly use lxd
--------------------------------------------------------------------------

Assign sudo permission to oneadmin and add it to lxd group

.. prompt:: bash # auto

    # echo "oneadmin ALL= NOPASSWD: ALL" >> /etc/sudoers
    # usermod -a -G lxd oneadmin

To use *lxd* as sudo with *oneadmin* user, in **/etc/sudoers.d/opennebula** replace

.. code-block:: bash

    Defaults:oneadmin secure_path = /sbin:/bin:/usr/sbin:/usr/bin

by:

.. code-block:: bash

    Defaults:oneadmin secure_path = /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/games:/usr/games:/snap/bin

4.3. Configure Passwordless SSH
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

4.4. Install VNC server
-------------------------------------

We compiled and provided it for Raspbian Stretch in our releases. Download it from the latest release and install the
required dependencies from repositories.

.. prompt:: bash # auto
    
    sudo dpkg -i <path_to>/svncterm_1.2-1_armhf.deb


5. Connect
==============

Now you are done and ready to connect the new node to your cloud. I know, I know, it's easier to download a pre-made
image from clox.org but it always feels better to do it yourself. Either that or you are feeling really annoyed knowing
all the time you could have saved yourself with the pre-made image :)
