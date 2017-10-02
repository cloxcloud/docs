.. _Box0 for Raspberry Pi3: https://mega.nz/#!BiAyFQra!9N4Dq0TKQmOdPdfs9MwvwmhVVlCqLVhkcHrh1AlE0n8


.. _the_short_road_first_node:

******************************************
First node deployment
******************************************

This guide intends to describe the process of deploying your first node. We will call it "**box0**". This node is nothing else than a basic compute node plus an LXD container running OpenNebula. A ready to be deployed image have been created, check :ref:`the_short_road_first_node`.

You will need a microSD card with at least 4GB and preferably class 10 or better.

.. note::
    * Read **Notes** sections attached to some steps, before using the shell
    * Commands prefixed by "**#**" are meant to be run as root. Commands prefixed by "**$**" must be run as a normal user.
    * ``(...)`` in code snippets means that could be code before/after the modified lines. That portion of code need to stay unmodified.

1. Get and install your first node
==========================================
Please, download the image and follow this short guide.

1.1. Download image
-------------------------------------------------------------
Download `Box0 for Raspberry Pi3`_.


1.2. Dump image on a microSD card
-------------------------------------------------------------

1.2.1. If you use Linux
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
I will assume **/dev/mmcblk0** is your microSD card. Please check and be sure. If you make a mistake and erase all of your
girlfriend's pictures you are on your own so good luck.

.. prompt:: bash # auto

    # dd if=./image_name.img of=/dev/mmcblk0 && sync

1.2.2. If you use Windows
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Sorry, no idea. Check the Internet. Never tried before. Luckily never will. 
`This link <https://www.raspberrypi.org/documentation/installation/installing-images/>`_ might help.

1.3. Turn it on
-------------------------------------------------------------
Connect the microSD to your Raspberry and turn it on.

1.4. Expand filesystem
-------------------------------------------------------------
Now you should expand the filesystem so it will use all the space from your microSD card. Log in  the new node either through ssh or directly with a keyboard and a monitor. Default user and password is "**cloud**". Enter the "**Raspberry Pi Software Configuration Tool**":

.. prompt:: bash $ auto

    $ sudo raspi-config

Go to "**Advanced Options**" and hit "**Expand Filesystem**"

"**READY**", that is all. Easy, right?

.. note::
    Now you have your first node ready. As explained before on the overview we recommend at least two nodes and a NFS drive for production, but if you just want to try "**Clox**" you can use this Raspberry. Everything is already configured. Open the following URL: `<http://192.168.0.9:9869/>`_. The default user is "**oneadmin**" and default password "**oneadmin**" also. 


.. warning::
    The default OpenNebula's datastore has been maintained, with the default configuration. This datastore uses the ssh drivers, which means the image will be copied before deploying the container. Because of this the container will start slowly and will not run as smoothly as it should. That is why we recommend an NFS datastore. Still, you can now go to the frontend with this URL: `<http://192.168.0.9:9869/>`_ but only with testing purposes. The default user is "**oneadmin**" and default password "**oneadmin**" also. Click on the "**Keep me logged in**" checkbox if the session closes unexpectedly.

Now, you should create the NFS datastore and add new compute nodes. Please, go to the next guide.
