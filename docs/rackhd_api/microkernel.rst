Microkernel image
=============================

.. contents:: Table of Contents

RackHD utilizes `RancherOS`_ booted in RAM and a customized docker image run in RancherOS to perform various operations such as node discovery and firmware management.

.. _RancherOS: https://rancher.com/rancher-os

The `on-imagebuilder`_ repository contains a set of scripts that uses `Docker`_ to build
docker images that run in RancherOS, primarily for use with the `on-taskgraph`_ workflow engine.

.. _on-imagebuilder: https://github.com/rackhd/on-imagebuilder
.. _Docker: https://www.docker.com
.. _on-taskgraph: https://github.com/rackhd/on-taskgraph

Requirements
-----------------------------

- Docker

Bootstrap Process
-----------------------------

The images produced by these scripts are intended to be netbooted and run in RAM.
The typical flow for how these images are used/booted is this:

- Netboot **RacherOS** (kernel and initrd) via PXE/iPXE
- The custom cloud-config file requests a **rackhd/micro** docker image from the boot server.
- It then starts a container with full container capabilities using the **rackhd/micro** docker image.

Building Images
-----------------------------

Instructions for building images, can be found in the `on-imagebuilder README`_.

.. _on-imagebuilder README: https://github.com/RackHD/on-imagebuilder/blob/master/README.md

How To Login Microkernel
-----------------------------

By default, RackHD has a workflow to let users login RancherOS based microkernel to debug.
The workflow name is `Graph.BootstrapRancher`.

.. code-block:: REST

    curl -X POST -H 'Content-Type: application/json' <server>/api/current/nodes/<identifier>/workflows?name=Graph.BootstrapRancher

When this workflow is running, it will set node to PXE boot, then reboot the node.
The node will boot into microkernel, finally you could SSH login node's microkernel from the RackHD server.
The node's IP address could be retrieved from 'GET /lookups' API like below,
the SSH username:password is `rancher:monorail`.

.. code-block:: REST

    curl <server>/api/current/lookups?q=<identifier>
