.. _the_short_road_overview:

================================================================================
Overview
================================================================================

Right now, Clox is only supported on Raspberry Pi3 although it should work just fine on Raspberry Pi2. Support for Pine64 should be coming soon.

If you only want to test Clox, one Raspberry Pi3 will be enough. We recommend at least two Raspberrys, a file system shared via NFS and a switch to interconnect all. This could be accomplished by a NAS drive or any computer serving the share. The reason to use NFS is that microSD cards are too slow to run the containers that will power this platform. With a descent hard drive exported via NFS you will have a much better performance. Please note  that a single NFS drive will be a bottleneck once the cluster starts getting big. In order to solve this you could add more NFS datastores or switch to a SAN. For the last case we recommend Ceph which is supported by OpenNebula and LXDoNe.

The first node will be called "**box0**" and it's IP address will be "**192.168.0.10**". This node is nothing else than a basic compute node plus an LXD container running OpenNebula. A ready to be deployed image have been created, check :ref:`the_short_road_first_node`.

Then, you will need to add more compute nodes that will be called "**box1**", "**box2**" ... "**boxN**". Their IP addresses will be "**192.168.0.11**" "**192.168.0.12**" "**192.168.0.(10+N)**", check :ref:`the_short_road_new_node`

.. note::
	Default username and password for all images is "**cloud**". You are allowed to execute root commands with sudo.

