Comparison to other open source efforts
=======================================

Cobbler comparison
------------------

Razor comparison
----------------

* HTTP wrapper to configure standard open source tooling for imaging
 - No dynamic events or control for TFTP, DHCP
* Catalog and policy equivalent to discovery workflow and SKU mechanism
oriented on single, static OS for hardware
 - Focused solely on OS installation
 - No workflow engine or concept of orchestration with multiple reboots (firmware update)
* Tightly bound to and maintained by Puppet
 - “Loss leader” to provide bare-metal provisioning lock for Puppet
 - Forked variant “Hanlon” used for Chef Metal driver

xCat comparison
---------------

* HPC Cluster Centric tool focused on IBM supported hardware
* Many features restricted to IBM/Lenovo proprietary hardware
* Has no concept of workflow or sequencing
* Has no concept of failure recovery
* Competing with Puppet/Chef/Ansible/cfEngine to own config management story
* Extensibility model tied exclusively to Perl code
* REST API is anemic with focus on CLI management
* Built as a master controller of infratructure vs an element in the process
* Actively maintained only by IBM staff
