RackHD Debugging Guide
-----------------------

Discovery with a Default Workflow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Sequence Diagram for the Discovery Workflow

.. image:: https://www.websequencediagrams.com/cgi-bin/cdraw?lz=dGl0bGUgRGVmYXVsdCBEaXNjb3ZlcnkgV29ya2Zsb3cKICAgIFNlcnZlci0-UmFja0hEOiBESENQIGZyb20gUFhFKG5pYyBvciBCSU9TKQAqBQAhBi0-ADEGOiBJU0MALQZyZXNwb25zZSB3aXRoIElQABkVREhDUC1wcm94eQAhD2Jvb3RmaWxlAIEOBW5vdGUgcmlnaHQgb2YAawc6IElmIHRoZSBub2RlIGlzIGFscmVhZHkgImtub3duIiwgaXQgd2lsbCBvbmwAVghkIGkAMAVyZSdzIGFuIGFjdGl2ZSB3AIF5ByB0aGF0J3MgYmVlbiBpbnZva2VkIHJlbGF0ZWQgdG8AZgkAghMVUmVxdWVzdCB0byBkb3dubG9hZACBPQkgdmlhIFRGVACBbxZURlRQIHNlbmRzIHIAPwZlZCAAMgUobW9ub3JhaWwuaXB4ZQCCawYAgggFbGVmAIIHBQCCbggAgy8GIGxvYWRzIAAoDSBhbmQgaW5pdGlhdGVzIG9uAIJWBWxvYWRlcgCDWxVJUFhFIHNjcmlwdACBBwhzIHdoYQCBUwcAhAUGAIQXBiAoaHR0cACBDAsAgxQQAIRMBSAgICBtdWx0aWxpbgCDSgYAhC4KIGxvb2tzIHVwIElQIGFkZHJlc3Mgb2YgSFRUUACCBQgAhHcGaQCBDAt0byBmaW5kAIN1CnZpYSBpdHMgbWFjLQA_By4AcwkAgymAfwBxEm4ndCAAhSAFAIUaCmNyZWF0ZSBhAIUBCihkAIcgB2lzAIViBQCFGwknR3JhcGguU2t1LgCHOQknKQCDYgUAhV0IAIZtBWFuAIIkEACDfggAhWIFAIMSCWVuZACGXAUAhzkVAIJqDCgAhAAFAINzB2NhbGxzIGEgUHJvZmlsZSkgKHZpYSAAhAAPAIUWEACDOAwAiBEFZACIegltaWNyb2tlcm5lbACFKQlyZACEVQwAiQgQAIQMBQCFGAlzdGF0aWMAhjAGLQCIEwV2bWxpbnV6IABPBgCJFxUAGwgALzcAgR4GAIIiFgCBLhEAhxUidGhlAIEmBwCBdgxhbmQgdHJhbnNmZXJzIGNvbnRyb2wgKGJvb3RzAIkrBQCCLwwAghMXAIJGBgCIFwZhZGRpdGlvbmFsAIhcB292ZXJsYXkpAIteBgCIQgd0byBleHRlbmQAgw0MAIhlGnRoZQCDNRdpcyBzZQCKEAUAh0YIYW5kIGxhdW5jaCBhIE5vZGVKUyB0YXNrIHJ1bm5uAIh-FwCJAAl0aGUAjBcFc3RyYXAuanMgdGVtcGxhAIUoFwAdDWZpbGxlZCBvdQCEdgd2YWx1ZXMgc3BlY2lmaWMAi1UMIGJhc2VkIG9uIGEAiR0FdXAAimEacnVucwAuBwCBDQsAjjYVAIEwCSBhc2tzIGZvcgCBdQVzAIZUB3Nob3VsZCBJIGRvPwCORxZkYXRhIHBhY2tlAI4FBQA2BwCGVyMAj1oSIHBhc3NlcwCNJwUAgH8HAIgeBXRlcnJvZ2F0ZSBoYXJkd2FyAI59Bmxvb3AAgSwFZWFjaCBUYXNrAIt9DACLXwkAkC0Qb3V0cHUAgSMJAJBbBWVuAIYtBgCPQRQAjB4SIACMOgkAhB8FAEkHc3RvcmVkIGFzIGNhdGFsb2dzIGluAI0NCACKbRxpAJA3CCBpAIZIBWZpZ3VyZQCKKwdTS1UgZGVmAI4oBW9ucwCQPQVwcm9jZQCCJAV0aGVzZQBmCnRvIGRldGVybWluZQCRAwVTS1UAWwwAkF0JAFIFAIRcCQCQYgkAZAVlZCwAh0oJAIELBnRpbnUAkGUIAIsaCwCGUw4AkScJAIwvDW4gZW5jbG9zdXIAhTUQdGgAgTAJAIQ-BQAyJWFsc28AjRAISVBNSSBwb2xsZXIAhTAHAJJwCWYgcmVsZXZlbnQgaW5mb3JtYXRpb24gY2FuIGJlIGZvdQCQegUAdwwAjGQWAIVbUU5vdGhpbmcgbW9yZSwgdGhhbmtzIC0gcGxlYXNlIHJlYm9vdACNKgw&s=napkin

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
    note right of RackHD:
        multiline
        RackHD looks up IP address of HTTP request from iPXE script to find the node via its mac-address.
        If the node is already "known", it will only respond if there's an active workflow that's been invoked related to the node
        If the node isn't known, it will create a workflow (default is the workflow 'Graph.Sku.Discovery') and respond with an iPXE script to initiate that
        end note
    RackHD->Server: iPXE script (what RackHD calls a Profile) (via http)
    note left of Server: iPXE script with discovery microkernel and initrd (http)
    Server->RackHD: iPXE requests static file - the vmlinuz kernel
    RackHD->Server: vmlinuz (http)
    Server->RackHD: iPXE requests static file - initrd
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
    note right of RackHD
        multiline
        task output stored as catalogs in RackHD related to the node
        if RackHD is configured with SKU definitions, it processes these catalogs to determine the SKU
        if there's a SKU specific workflow defined, control is continued to that
        the discovery workflow will create an enclosure node based on the catalog data
        the discovery workflow will also create IPMI pollers for the node if relevent information can be found in the catalog
        end note
    Server->RackHD: bootstrap asks for tasks (what should I do?)
    RackHD->Server: Nothing more, thanks - please reboot (via http)
