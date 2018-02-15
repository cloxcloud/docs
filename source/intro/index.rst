.. _intro:

================================================================
Introduction
================================================================

.. toctree::
   :maxdepth: 2

Clox is a Cloud platform designed with efficiency and simplicity in mind. It is meant to be run on Single Board Computers (SBC) based on ARM processors. Supported SBCs at the moment are Raspberry Pi3 and Pine64. We recommend using Pine64 for it's bigger amount of RAM and Gigabit Ethernet. Also, support for Pine64 is better at the moment.

CloX is a solution for small companies, enabling them to easily and cheaply deploy their Private Clouds and stop depending on Public Clouds. It is based on OpenNebula and LXD, so you can expect an easy of use, fully featured cloud orchestrator with minimum overhead, great performance and reduced deployment times. CloX is meant to be deployed over Single Board Computers (SBCs) based on ARM processors providing low capital expenses and power consumption.

If you only want to test Clox, one supported SBC will be enough. We recommend at least two SBCs, a file system shared via NFS and a switch to interconnect all. This could be accomplished by a NAS drive or any computer serving the share. The reason to use NFS is that microSD cards are too slow to run the containers that will power this platform. With a descent hard drive exported via NFS you will have a much better performance. Please note  that a single NFS drive will be a bottleneck once the cluster starts getting big. In order to solve this you could add more NFS datastores or switch to a SAN. For the last case we recommend Ceph which is supported by CloX.
The physical design, on it most simple form, should look like this:

.. image:: ../picts/architecture.png

To get started, choose from the menu the the model of the SBC(s) where you will deploy CloX. For a quick deployment you should follow *The short road*. Clox is totally free and open. All the software used is OpenSource and you can check how everything was integrated by following *The not so short road*. There, you will find how to do it all by yourself and even contribute to the project.
