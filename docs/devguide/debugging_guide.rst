RackHD Debugging Guide
-----------------------

Discovery with a Default Workflow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Sequence Diagram for the Discovery Workflow

.. image:: https://www.websequencediagrams.com/cgi-bin/cdraw?lz=dGl0bGUgRGVmYXVsdCBEaXNjb3ZlcnkgV29ya2Zsb3cKICAgIFNlcnZlci0-UmFja0hEOiBESENQIGZyb20gUFhFKG5pYyBvciBCSU9TKQAqBQAhBi0-ADEGOiBJU0MALQZyZXNwb25zZSB3aXRoIElQABkVREhDUC1wcm94eQAhD2Jvb3RmaWxlAIEOBW5vdGUgcmlnaHQgb2YAawc6IElmIHRoZSBub2RlIGlzIGFscmVhZHkgImtub3duIiwgaXQgd2lsbCBvbmwAVghkIGkAMAVyZSdzIGFuIGFjdGl2ZSB3AIF5ByB0aGF0J3MgYmVlbiBpbnZva2VkIHJlbGF0ZWQgdG8AZgkAghMVUmVxdWVzdCB0byBkb3dubG9hZACBPQkgdmlhIFRGVACBbxZURlRQIHNlbmRzIHIAPwZlZCAAMgUobW9ub3JhaWwuaXB4ZQCCawYAgggFbGVmAIIHBQCCbggAgy8GIGxvYWRzIAAoDSBhbmQgaW5pdGlhdGVzIG9uAIJWBWxvYWRlcgCDWxVJUFhFIHNjcmlwdACBBwhzIHdoYQCBUwcAhAUGAIQXBiAoaHR0cACBDAsAgxMRACEHbG9va3MgdXAgSVAgYWRkcmVzcyBvZiBIVFRQAIFrCACEXQYAcQx0byBmaW5kAINbCnZpYSBpdCdzIG1hYy0AQAcuAIMHgRoAhRIkbid0IACFKwUAhSUKY3JlYXRlIGEAhQwKKGQAhysHaXMgYSBkAIcuCQCFLwgpAINfBQCFWggAhmoFYW4gaQCCPQ4Ag3sIAIVfBQCHJRUAhCkFAINyBygAg2wFAINfB2NhbGxzIGEgUHJvZmlsZSkgKHZpYSAAg2wPAIUCEABCDACHfQUAgS4KbWljcm9rZXJuZWwAhRUJcmQAhEEMAIh0EACFRAUAhQQJc3RhdGljAIYcBi0Ah38Fdm1saW51eiAATwYAiQMVABsIAC83AIEeBgCCIhYAgS4RAIcBInRoZQCBJgcAgXYMYW5kIHRyYW5zZmVycyBjb250cm9sIChib290cwCJFwUAgi8MAIITFwCCRgYAiAMGYWRkaXRpb25hbACISAdvdmVybGF5KQCLSgYAiC4HdG8gZXh0ZW5kAIMNDACIURp0aGUAgzUXaXMgc2UAiXwFAIdMCGFuZCBsYXVuY2ggYSBOb2RlSlMgdGFzayBydW5ubgCIahcAiGwJdGhlAIwDBXN0cmFwLmpzIHRlbXBsYXQAjA8GAIxpEAAdDWZpbGxlZCBvdQCEdgd2YWx1ZXMgc3BlY2lmaWMAi0EMIGJhc2VkIG9uIGEAiSMFdXAAik0acnVucwAuBwCBDQsAjiIVAIEwCSBhc2tzIGZvcgCBdQVzAIZUB3Nob3VsZCBJIGRvPwCOMxZkYXRhIHBhY2tlAI1xBQA2BwCGVyMAj0YSIHBhc3NlcwCNEwUAgH8HAIgNBXRlcnJvZ2F0ZSBoYXJkd2FyAI5pBmxvb3AAgSwFZWFjaCBUYXNrAItpDACQIAUAkBUUb3V0cHUAgSMJAJBHBWVuAIYtBgCPKxYAhAUFAC8Hc3RvcmVkIGFzIGNhdGFsb2dzIGluAIxfCACKNC5pAJAbCCBpAIZABWZpZ3VyZQCKEgdTS1UgZGVmAI4MBW9ucwCQIQVwcm9jZQCCHAV0aGVzZQB4CnRvIGRldGVybWluZQCQZwVTS1UAWx4AkFMJAGQFAIRmCQCQWAkAdgVlZCwAh1QJAIEdBnRpbnUAkFsIYXQsIG90aGVyd2lzAHAGAJFdBQCRRgViZSB0b2wAkQcFcmVib28Ai0MGAIQqUU5vdGhpbmcgbW9yZSwgdGhhbmtzIC0gcGxlYXNlAHMHAIt6Cw&s=napkin

The diagram is made with `WebSequenceDiagrams`_.

To see if the DHCP request was received by ISC DHCP, look in `/var/log/syslog` of the RackHD host. `grep DHCP /var/log/syslog` works reasonably well - you're looking for a sequence like this::

    Jan  8 15:43:43 rackhd-demo dhclient: DHCPDISCOVER on eth0 to 255.255.255.255 port 67 interval 3 (xid=0x5b3b9260)
    Jan  8 15:43:43 rackhd-demo dhclient: DHCPREQUEST of 10.0.2.15 on eth0 to 255.255.255.255 port 67 (xid=0x60923b5b)
    Jan  8 15:43:43 rackhd-demo dhclient: DHCPOFFER of 10.0.2.15 from 10.0.2.2
    Jan  8 15:43:43 rackhd-demo dhclient: DHCPACK of 10.0.2.15 from 10.0.2.2

You should also see the DHCP proxy return the bootfile. In the DHCP-proxy logs, look for lines with `DHCP.messageHandler`::

    S 2016-01-08T19:31:43.268Z [on-dhcp-proxy] [DHCP.messageHandler] [Server] Unknown node 08:00:27:f3:9f:2e. Sending down default bootfile.

And immediately thereafter, you should see the server request the file from TFTP::

    S 2016-01-08T19:31:43.352Z [on-tftp] [Tftp.Server] [Server] tftp: 67.300 monorail.ipxe

.. _WebSequenceDiagrams: https://www.websequencediagrams.com


Default discovery workflow
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code::

    title Default Discovery Workflow
    Server->RackHD: DHCP from PXE(nic or BIOS)
    RackHD->Server: ISC DHCP response with IP
    RackHD->Server: DHCP-proxy response with bootfile
    note right of RackHD: If the node is already "known", it will only respond if there's an active workflow that's been invoked related to the node
    Server->RackHD: Request to download bootfile via TFTP
    RackHD->Server: TFTP sends requested file (monorail.ipxe)
    note left of Server: Server loads monorail.ipxe and initiates on bootloader
    Server->RackHD: IPXE script requests what to do from RackHD (http)
    note right of RackHD: RackHD looks up IP address of HTTP request from IPXE script to find the node via it's mac-address.
    note right of RackHD: If the node is already "known", it will only respond if there's an active workflow that's been invoked related to the node
    note right of RackHD: If the node isn't known, it will create a workflow (default is a discovery workflow) and respond with an iPXE script to initiate that
    RackHD->Server: ipxe script (what RackHD calls a Profile) (via http)
    note left of Server: ipxe script with discovery microkernel and initrd (http)
    Server->RackHD: ipxe requests static file - the vmlinuz kernel
    RackHD->Server: vmlinuz (http)
    Server->RackHD: ipxe requests static file - initrd
    RackHD->Server: initrd (http)
    note left of Server: Server loads the kernel and initrd and transfers control (boots that microkernel)
    Server->RackHD: initrd loads additional file (overlay) from Server to extend microkernel
    note left of Server: the discovery microkernel is set to request and launch a NodeJS task runnner
    Server->RackHD: requests the bootstrap.js template
    RackHD->Server: bootstrap.js filled out with values specific to the node based on a lookup
    note left of Server: runs node bootstrap.js
    Server->RackHD: bootstrap asks for tasks (what should I do?)
    RackHD->Server: data packet of tasks (via http)
    note left of Server: Discovery Workflow passes down tasks to interrogate hardware
    loop for each Task from RackHD
        Server->RackHD: output of task
    end
    note right of RackHD: task output stored as catalogs in RackHD related to the node
    note right of RackHD: if RackHD is configured with SKU definitions, it processes these catalogs to determine the SKU
    note right of RackHD: if there's a SKU specific workflow defined, control is continued to that, otherwise the node will be told to reboot
    Server->RackHD: bootstrap asks for tasks (what should I do?)
    RackHD->Server: Nothing more, thanks - please reboot (via http)
