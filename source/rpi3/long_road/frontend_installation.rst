.. include:: ../vars.rst
.. _the_not_so_short_road_frontend_installation:

******************************************
Front-end installation
******************************************

This guide intends to describe the process of installing OpenNebula on a Ubuntu container running |architecture| architecture. You can download a basic container from `<http://linuxcontainers.org/>`_ image repositories or Ubuntu `<http://ubuntu.org/>`_ images or you can even use the base image provided in `<http://clox.cloud/>`_ and follow the steps provided on this guide.

.. warning::
    Please note there is a ready to be deployed container that will save you the trouble of this guide, see :ref:`pine64_the_short_road_first_node`. This guide is only intended to show what is inside that container and for those who want to contribute and improve this process.

.. note::
    * Read **Notes** sections attached to some steps, before using the shell
    * Commands prefixed by "**#**" are meant to be run as root. Commands prefixed by "**$**" must be run as a normal user.
    * ``(...)`` in code snippets means that could be code before/after the modified lines. That portion of code need to stay unmodified.

1. Install OpenNebula
=================================

1.1. Dependencies first
---------------------------------

You can find OpenNebula dependencies `here <http://docs.opennebula.org/5.2/integration/references/build_deps.html#build-deps>`_.
But if you are as lazy as me, just run the following command:

.. prompt:: bash # auto

    # apt install g++ scons libmysqlclient-dev bash-completion bison debhelper default-jdk flex javahelper
    libmysql++-dev libsqlite3-dev libssl-dev libws-commons-util-java libxml2-dev libxmlrpc3-client-java
    libxmlrpc3-common-java libxslt1-dev libcurl4-openssl-dev ruby scons libxmlrpc-c++8-dev nfs-common genisoimage

1.2. Prepare the source code
-----------------------------------

Get the sources and compile them

.. prompt:: bash # auto

    # wget http://downloads.opennebula.org/packages/opennebula-5.2.1/opennebula-5.2.1.tar.gz
    # tar -xvzf opennebula-5.2.1.tar.gz
    # cd opennebula-5.2.1
    # scons

Now, grab a cup of coffee, walk your dog and take a shower. Don't worry, it will still be compiling when you return.

1.3. oneadmin
---------------------------------

Create ``oneadmin`` user and group

.. prompt:: bash # auto

    # groupadd -g 9869 oneadmin
    # useradd -d /var/lib/one -g oneadmin -u 9869 oneadmin
    # ./install.sh -u oneadmin -g oneadmin

1.4. Ruby dependencies
---------------------------------

Install required ruby gems. Luckly OpenNebula provides us with a script for this

.. prompt:: bash # auto

    # /usr/share/one/install_gems

1.5. oneadmin password
----------------------------------

Create **oneauth** file with ``oneadmin`` password. This password will be used when entering Sunstone. Replace
***password*** with your password.

.. prompt:: bash # auto

    # mkdir /var/lib/one/.one
    # echo "oneadmin:password" > /var/lib/one/.one/one_auth
    # chown -R oneadmin:oneadmin /var/lib/one
    # chmod 600 /var/lib/one/.one/one_auth

1.6. Configure OpenNebula services's logrotate
----------------------------------------------------

Just copy inside **/etc/logrotate.d/opennebula** the following configuration:

.. code-block:: bash

    delaycompress
    dateext
    dateformat -%Y%m%d-%s

    compress
    weekly
    rotate 52

    /var/log/one/one_xmlrpc.log {
        missingok
        notifempty
        copytruncate
    }

    /var/log/one/oned.log {
        missingok
        notifempty
        copytruncate
    }

    /var/log/one/sched.log {
        missingok
        notifempty
        copytruncate
    }

Set appropriated permissions to the file:

.. prompt:: bash # auto

    # chown root:root /etc/logrotate.d/opennebula

1.7. OpenNebula systemd services
-----------------------------------

Download systemd units an untar it:

.. prompt:: bash $ auto

    $ wget opennebula.systemd.units
    $ tar xf opennebula.systemd.units

Copy it to **/lib/systemd/system/** to be able to manage OpenNebula's services from systemd set the approppiate
permissions:

.. prompt:: bash # auto

    # cp -rpa opennebula.systemd.units/* /lib/systemd/system/
    # chown root:root /lib/systemd/system/opennebula*
    # chmod 644 /lib/systemd/system/opennebula*

1.7. SSH key
-----------------------------------
Generate an SSH authentication key for oneadmin. Then, authorize login with the generated key. Log in as oneadmin and execute this:

.. prompt:: bash # auto

    # su oneadmin

.. prompt:: bash $ auto

    $ ssh-keygen
    $ cp /var/lib/one/.ssh/id_rsa.pub /var/lib/one/.ssh/authorized_keys


2. LXDoNe
====================

This is the add-on that will allow OpenNebula to manage LXD containers.

2.1. Download and install driver
---------------------------------------

Download the `latest release <https://github.com/OpenNebula/addon-lxdone/releases/>`_ and untar it:

.. prompt:: bash $ auto

    $ tar -xf <lxdone-release>.tar.gz

Copy scripts to oneadmin drivers directory:

.. prompt:: bash $ auto

    $ cd addon-lxdone<lxdone-release>
    $ cp -rpa src/remotes/ /var/lib/one/


Set the appropriate permissions

.. prompt:: bash $ auto

    $ cd /var/lib/one/remotes/

.. prompt:: bash # auto

    # chown -R oneadmin:oneadmin vmm/lxd im/lxd*
    # chmod 755 -R vmm/lxd im/lxd*
    # chmod 644 im/lxd.d/collectd-client.rb

Go back to the addon's folder

.. prompt:: bash $ auto

    $ cd -

2.1.1. (Optional) Add support for 802.1Q driver (VLANs).
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Replace /var/lib/one/remotes/vnm.rb file for our modified version.

.. prompt:: bash $ auto

    $ cp -rpa src/one_wait/nic.rb /var/lib/one/remotes/vnm/nic.rb

.. prompt:: bash # auto

    # chown oneadmin:oneadmin /var/lib/one/remotes/vnm/nic.rb
    # chmod 755 /var/lib/one/remotes/vnm/nic.rb


2.2. Enable LXD
---------------------------------------

Modify **/etc/one/oned.conf**. Under **Information Driver Configuration** add this:

.. code-block:: bash

    #-------------------------------------------------------------------------------
    # lxd Information Driver Manager Configuration
    # -r number of retries when monitoring a host
    # -t number of threads, i.e. number of hosts monitored at the same time
    #-------------------------------------------------------------------------------
    IM_MAD = [ NAME = "lxd",
    EXECUTABLE = "one_im_ssh",
    ARGUMENTS = "-r 3 -t 15 lxd" ]
    #-------------------------------------------------------------------------------


Under **Virtualization Driver Configuration** add this:

.. code-block:: bash

    #-------------------------------------------------------------------------------
    # lxd Virtualization Driver Manager Configuration
    # -r number of retries when monitoring a host
    # -t number of threads, i.e. number of actions performed at the same time
    #-------------------------------------------------------------------------------
    VM_MAD = [ NAME = "lxd",
    EXECUTABLE = "one_vmm_exec",
    ARGUMENTS = "-t 15 -r 0 lxd",
    KEEP_SNAPSHOTS = "yes",
    TYPE = "xml",
    IMPORTED_VMS_ACTIONS = "migrate, live-migrate, terminate, terminate-hard, undeploy, undeploy-hard, hold, release, stop, suspend, resume, delete, delete-recreate, reboot, reboot-hard, resched, unresched, poweroff, poweroff-hard, disk-attach, disk-detach, nic-attach, nic-detach, snap-create, snap-delete"]
    #-------------------------------------------------------------------------------

2.3. Modify OpenNebula's information driver
-------------------------------------------------

ARM processors left out the PCI bus, so Opennebula´s pci.rb script will fail. Let's simply remove it.

.. prompt:: bash # auto

    # rm /var/lib/one/remotes/im/lxd-probes.d/pci.rb

2.4. Remove LXDoNe CPU limitation
------------------------------------------

Edit **/var/lib/one/remotes/vmm/lxd/deploy.py** and comment the following line:

.. code-block:: bash

    (...)
    profile['config'].append(lc.map_cpu(xqi('CPU', dicc)))  # cpu percentage
    (...)

Should look like this:

.. code-block:: bash

    (...)
    profile['config'].append(lc.map_ram(xqi('MEMORY', dicc)))
    # profile['config'].append(lc.map_cpu(xqi('CPU', dicc)))  # cpu percentage
    profile['config'].append(lc.map_vcpu(xqi('VCPU', dicc)))  # cpu core
    (...)

2.5. Configure systemd units
------------------------------------------

Reload the systemd manager configuration.

.. prompt:: bash # auto

    # systemctl daemon-reload

Enable required units so they will start automatically

.. prompt:: bash # auto

    # systemctl enable opennebula opennebula-sunstone opennebula-flow opennebula-gate

Now, start opennebula service to check everything is OK:

.. prompt:: bash # auto

    # systemctl start opennebula opennebula-sunstone

3. Add host names to **/etc/hosts**
=========================================
Make sure the frontend's hostname is listed in **/etc/hosts** after the loopback address (127.0.0.1)

Also, very node must be capable to reach the frontend container by his hostname. If you are using host names to add nodes in
the frontend, the frontend need to know the node's names too. You can configure a DNS server with an entry for every node
and the frontend or simply included his names in **/etc/hosts**.
