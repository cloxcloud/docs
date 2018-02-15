.. include:: ../vars.rst

.. _pine64_the_short_road_overview:

================================================================================
Overview
================================================================================

This guide will allow you to deploy CloX on a |SBC_model| SBC.

For testing Clox you will only need a |SBC_model| and deploy the first node, called "**box0**", see  :ref:`pine64_the_short_road_first_node`. For production environments you will need two or more |SBC_model|, depending on how many services you will deploy. You will also need an external hard drive disk that supports NFS, check :ref:`pine64_the_short_road_nfs_datastore`. Finally, you need one or more switches (depending on how many SBCs and HDDs you are using) to interconnect everything.

The first node will be called "**box0**" and it's IP address will be "**192.168.0.10**". This node is nothing else than a basic compute node plus an LXD container running OpenNebula. A ready to be deployed image have been created, check :ref:`pine64_the_short_road_first_node`.

Then, you will need to add more compute nodes that will be called "**box1**", "**box2**" ... "**boxN**". Their IP addresses will be "**192.168.0.11**" "**192.168.0.12**" "**192.168.0.(10+N)**", check :ref:`pine64_the_short_road_new_node`

.. note::
	Default username and password for all images is "**cloud**". You are allowed to execute root commands with sudo.

