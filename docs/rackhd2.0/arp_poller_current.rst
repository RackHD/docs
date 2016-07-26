ARP Cache Poller
----------------
.. _ARP: https://en.wikipedia.org/wiki/Address_Resolution_Protocol
.. _/proc/net/arp: https://www.kernel.org/doc/Documentation/filesystems/proc.txt

With the Address Resolution Protocol (`ARP`_) cache poller service enabled, the RackHD lookup service will update MAC/IP bindings based on the Linux kernel's `/proc/net/arp`_ table. This ARP poller deprecates the need for running the DHCP lease file poller since any IP request made to the host will attempt to resolve the hardware addresses IP and update the kernel's ARP cache. 

