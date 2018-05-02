Frequent Asks
=============================

.. tip::

  Q: How can I set obms automatically when node discovered in HP server?

  A: There is a "autoCreateObm" property you can set to true in your config.json file.
     When the autoCreateObm and arpCacheEnabled in opt/monorail/config.json are set to true, Discovery workflow will
     create a random credential using ipmitool in the container inside RancherOS and get the MAC Address from catalog,
     and use arp to lookup the IP of the specific server.
