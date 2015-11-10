Development Guide
=================

        *----------------------------- Content in Development -----------------------------*

Example RackHD Setup
--------------------

**Prerequisites**

The example setup uses Vagrant and ansible locally. Please install them
directly on your local machine.



1. Clone at minimum the rackHD repository. This repository includes both git
   submodules with all the relevant source code, as well as an example vagrant
   instance to get an instance up and running locally very easily.

   `git clone https://github.com/RackHD/RackHD`

   Within the example/ directory is a setup script to set up an "example rack"
   entirely virtual - albiet a very simple one. It creates a virtualbox with
   RackHD running within it, and another that is connected to it to PXE boot

2. Create a local configuration file and spin up vagrant

   `cd RackHD`
   `cp example/config/monorail_rack.cfg.example example/config/monorail_rack.cfg`

3. Fire up the virtual rack

   `cd example/bin`
   `./monorail_rack`

4. ssh into the RackHD server and use on-imagebuilder to build the static files

   `vagrant ssh dev`
   `sudo apt-get install -y ansible`
   `cd ~`
   `git clone https://github.com/rackhd/on-imagebuilder`
   `cd on-imagebuilder`
   `sudo ansible-playbook -i hosts all.yml`

5. Move the resulting files into the HTTP server's static files directory

   `mv /tmp/on-imagebuilder/builds/* ~/src/on-http/static/http/common/`


6. Finally, bring up the RackHD services

    `sudo nf start` or `sudo nf start [graph,http,dhcp,tftp,syslog]`

   Now that the services are running we can begin powering on pxe clients
   and watch them boot.


Working with RackHD APIs
------------------------

.. include:: monorail/graphs.rst
.. include:: monorail/tasks.rst
.. include:: monorail/skus.rst
.. include:: monorail/workflow_examples.rst
.. include:: monorail/pollers.rst
.. include:: monorail/microkernel.rst
.. include:: monorail/passive_discovery.rst
.. include:: monorail/monorail_api.rst
.. include:: monorail/naming_conventions.rst
.. include:: monorail/api_versioning.rst
.. include:: monorail/amqp_conventions.rst

Code Conventions
--------------------

Content Under Development
