Details about payload
=======================

.. _non-windows-payload:


Non-Windows OS Installation Workflow Payload
--------------------------------------------

All parameters descriptions of OS installation workflow payload are listed below, they are fit for use with all supported OSes except for CoreOS (see note below).

**NOTE:** The CoreOS installer is pretty basic, and only supports certain parameters shown below. Configurations not directly supported by RackHD may still be made via a custom Ignition template. Typical parameters for CoreOS include: *version*, *repo*, and *installScriptUri*|*ignitionScriptUri* and optionally *vaultToken* and *grubLinuxAppend*.

=================== ================ ============ ============================================
Parameters          Type              Flags       Description
=================== ================ ============ ============================================
version             String           **required** The version number of target OS that needs to install. **NOTE**: For Ubuntu, *version* should be the codename, not numbers, for example, it should be "trusty", not "14.04"
repo                String           **required** The OS repository address, currently only supports HTTP. Some examples of free OS distributions for reference. For **CentOS**, http://mirror.centos.org/centos/7/os/x86_64/. For **Ubuntu**, http://us.archive.ubuntu.com/ubuntu/. For **openSUSE**, http://download.opensuse.org/distribution/leap/42.1/repo/oss/. For **ESXi**, **RHEL**, **SLES** and **PhotonOS**, the repository is the directory of mounted DVD ISO image, and http service is provided for this directory.
osName              String           **required** **(Debian/Ubuntu only)** The OS name, the default value is **debian** for ubuntu installation use **ubuntu**.
rootPassword        String           *optional*   The password for the OS root account, it could be clear text, RackHD will do encryption before store it into OS installer's config file. default *rootPassword* is  **"RackHDRocks!"**. Some OS distributions' password requirements *must* be satisfied. For **ESXi 5.5**, `ESXi 5 Password Requirements <https://pubs.vmware.com/vsphere-50/index.jsp?topic=%2Fcom.vmware.vsphere.security.doc_50%2FGUID-DC96FFDB-F5F2-43EC-8C73-05ACDAE6BE43.html&resultof=%22password%22%20>`_. For **ESXi 6.0**, `ESXi 6 Password Requirements <http://pubs.vmware.com/vsphere-60/index.jsp#com.vmware.vsphere.security.doc/GUID-4BDBF79A-6C16-43B0-B0B1-637BF5516112.html?resultof=%2522%2550%2561%2573%2573%2577%256f%2572%2564%2522%2520%2522%2570%2561%2573%2573%2577%256f%2572%2564%2522%2520%2522%2552%2565%2571%2575%2569%2572%2565%256d%2565%256e%2574%2573%2522%2520%2522%2572%2565%2571%2575%2569%2572%2522%2520>`_.
hostname            String           *optional*   The hostname for target OS, default *hostname* is **"localhost"**
domain              String           *optional*   The domain for target OS
timezone            String           *optional*   **(Debian/Ubuntu only)** The Timezone based on $TZ. Please refer to https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
ntp                 String           *optional*   **(Debian/Ubuntu only)** The NTP server address.
users               Array            *optional*   If specified, this contains an array of objects, each object contains the user account information that will be created after OS installation. 0, 1, or multiple users could be specified.  If *users* is omitted, null or empty, no user will be created. See users_ for more details.
dnsServers          Array            *optional*   If specified, this contains an array of string, each element is the Domain Name Server, the first one will be primary, others are alternative.
ntpServers          Array            *optional*   If specified, this contains an array of string, each element is the Network Time Protocol Server.
networkDevices      Array            *optional*   The static IP setting for network devices after OS installation. If it is omitted, null or empty, RackHD will not touch any network devices setting, so all network devices remain at the default state (usually default is DHCP).If there is multiple setting for same device, RackHD will choose the last one as the final setting, both ipv4 and ipv6 are supported here. (**ESXi only**, RackHD will choose the first one in networkDevices as the boot network interface.) See networkDevices_ for more details.
rootSshKey          String           *optional*   The public SSH key that will be appended to target OS.
installDisk         String/Number    *optional*   *installDisk* is to specify the target disk which the OS will be installed on. It can be a string or a number. **For string**, it is a disk path that the OS can recongize, its format varies with OS. For example, "/dev/sda" or "/dev/disk/by-id/scsi-36001636121940cc01df404d80c1e761e" for CentOS/RHEL, "t10.ATA_____SATADOM2DSV_3SE__________________________20130522AA0990120088" or "naa.6001636101840bb01df404d80c2d76fe" or "mpx.vmhba1:C0:T1:L0" or "vml.0000000000766d686261313a313a30" for ESXi. **For number**, it is a RackHD generated disk identifier (it could be obtained from "driveId" catalog). If *installDisk* is omitted, RackHD will assign the default disk by order: SATADOM -> first disk in "driveId" catalog -> "sda" for Linux OS. **NOTE**: Users need to make sure the *installDisk* (either specified by user or by default) is the first bootable drive from BIOS and raid controller setup. **PhotonOS** only supports '/dev/sd*' format currently.
installPartitions   Array            *optional*   *installPartitions* is to specify the *installDisk*'s partitions when OS installer's default auto partitioning is not wanted. (Only support **CentOS** at present, Other Linux OS will be supported). See installPartitions_ for more details.
kvm                 Boolean          *optional*   The value is *true* or *false* to indicates to install kvm or not, default is **false**. (**ESXi, PhotonOS** doesn't support this parameter)
switchDevices       Array            *optional*   **(ESXi only)** If specified, this contains an array of objects with switchName, uplinks (optional), and failoverPolicy (optional) parameters. If *uplinks* is omitted, null or empty, the vswitch will be created with no uplinks. If *failoverPolicy* is omitted, null or empty, the default ESXi policy will be used. See switchDevices_ for more details.
postInstallCommands Array            *optional*   **(ESXi only)** If specified, this contains an array of string commands that will be run at the end of the post installation step.  This can be used by the customer to tweak final system configuration.
installType         String           *optional*   **(PhotonOS only)** The value is *minimal* or *full* to indicate the type of installed OS, defualt *installType* is **minimal**
installScriptUri    String           *optional*   The download URI for a custom kickstart/preseed/autoyast/cloud-config template to be used for automatic installation/configuration.
ignitionScriptUri   String           *optional*   **(CoreOS only)** The download URI for a custom Ignition template used for post-install system configurations for CoreOS Container Linux
vaultToken          String           *optional*   **(CoreOS only)** The token used for unwrapping a wrapped Vault response -- currently only an Ignition template (ignitionScriptUri) or cloud-config userdata (installScriptUri) payload is supported.
grubLinuxAppend     String           *optional*   **(CoreOS only)** Extra (persistent) kernel boot parameters

                                                  NOTE: There are RackHD specific commands within all default install templates that should be copied into any custom install templates. The built-in templates support the above options, and any additional install logic is best added by copying the default templates and modifying from there. The default install scripts can be found in https://github.com/RackHD/on-http/tree/master/data/templates, and the filename is specified by the `installScript` field in the various OS installer task definitions (e.g. https://github.com/RackHD/on-tasks/blob/master/lib/task-data/tasks/install-centos.js)
remoteLogging       Boolean          *optional*   If set to true, OS installation logs will be sent to RackHD server from nodes if installer supports remote logging. Note you must configure rsyslog on RackHD server if you want to receive those logs. Please refer to https://github.com/RackHD/RackHD/blob/master/example/config/rsyslog_rackhd.cfg.example as how to enable rsyslog service on RackHD server. Currently only CentOS installation supports this feature, we are still working on other OS installation workflows to enable this feature.
bonds               Array            *optional*   **(RHEL/CentOS only)** Bonded interface configuration. Bonded interfaces will be created after OS installation. If it is omitted, null or empty, RackHD will not create any bond interface.
packages            Array            *optional*   **(RHEL/CentOS only)** List of packages, package groups, package environments that needs to be installed along with base RPMs. If it is omitted, null or empty, RackHD will just install packages in base package group.
enableServices      Array            *optional*   **(RHEL/CentOS only)** List of services that needs to be enabled explicitly after OS installation is completed.
disableServices     Array            *optional*   **(RHEL/CentOS only)** List of services that needs to be disabled explicitly after OS installation is completed. If it is omitted, null or empty, RackHD will not not disable any installed service.

=================== ================ ============ ============================================

.. _users:

For **users** in payload:

=========== ======== ============ ============================================
Parameters  Type     Flags        Description
=========== ======== ============ ============================================
name        String   **required** The name of user. it should start with a letter or digit or underline and the length of it should larger than 1(>=1).
password    String   **required** The password of user, it could be clear text, RackHD will do encryption before store it into OS installer's config file. The length of password should larger than **4(>=5)**. Some OS distributions' password requirements *must* be satisfied. For **ESXi 5.5**, `ESXi 5 Password Requirements <https://pubs.vmware.com/vsphere-50/index.jsp?topic=%2Fcom.vmware.vsphere.security.doc_50%2FGUID-DC96FFDB-F5F2-43EC-8C73-05ACDAE6BE43.html&resultof=%22password%22%20>`_. For **ESXi 6.0**, `ESXi 6 Password Requirements <http://pubs.vmware.com/vsphere-60/index.jsp#com.vmware.vsphere.security.doc/GUID-4BDBF79A-6C16-43B0-B0B1-637BF5516112.html?resultof=%2522%2550%2561%2573%2573%2577%256f%2572%2564%2522%2520%2522%2570%2561%2573%2573%2577%256f%2572%2564%2522%2520%2522%2552%2565%2571%2575%2569%2572%2565%256d%2565%256e%2574%2573%2522%2520%2522%2572%2565%2571%2575%2569%2572%2522%2520>`_.
uid         Number   *optional*   The unique identifier of user. It should be between **500** and **65535**.(Not support for ESXi OS)
sshKey      String   *optional*   The public SSH key that will be appended into target OS.
=========== ======== ============ ============================================

.. _networkDevices:

For **networkDevices** in payload, both ipv4 and ipv6 are supported

============== ======== ============ ============================================
Parameters     Type     Flags        Description
============== ======== ============ ============================================
device         String   **required**  Network device name (**ESXi only**, or MAC address) in target OS (ex. "eth0", "enp0s1" for Linux, "vmnic0" or "2c:60:0c:ad:d5:ba" for ESXi)
ipv4           Object   *optional*    See `ipv4 or ipv6`_ more details.
ipv6           Object   *optional*    See `ipv4 or ipv6`_ more details.
esxSwitchName  String   *optional*    **(ESXi only)** The vswitch to attach the vmk device to. vSwitch0 is used by default if no esxSwitchName is specified.
============== ======== ============ ============================================

.. _installPartitions:

For **installPartitions** in payload:

============== ======== ============ ============================================
Parameters     Type     Flags        Description
============== ======== ============ ============================================
mountPoint     String   **required**  Mount point, it could be "/boot", "/", "swap", etc. just like the mount point input when manually installing OS.
size           String   **required**  Partition size, it could be a number string or "auto", For number, default unit is **MB**. For "auto", all available free disk space will be used.
fsType         String   *optional*    File system supported by OS, it could be "ext3", "xfs", "swap", etc. If *mountPoint* is "swap", the fsType must be "swap".
============== ======== ============ ============================================

* **Debian/Ubuntu** installation requires boot, root and swap partitions, make sure your auto sized partition must be the last partition.

.. _`ipv4 or ipv6`:

For **ipv4** or **ipv6** configurations:

=========== ======== ============ ============================================
Parameters  Type     Flags        Description
=========== ======== ============ ============================================
ipAddr      String   **required** The assigned static IP address
gateway     String   **required** The gateway.
netmask     String   **required** The subnet mask.
vlanIds     Array    *optional*   The VLAN ID. This is an array of integers (0-4095).
                                  In the case of Windows OS, the vlan is an array of one parameter only
mtu         Number   *optional*   Size of the largest network layer protocol data unit
=========== ======== ============ ============================================

.. _switchDevices:

For **switchDevices** **(ESXi only)** in payload:

============== ======== ============ ============================================
Parameters     Type     Flags        Description
============== ======== ============ ============================================
switchName     String   **required** The name of the vswitch
uplinks        String   *optional*   The array of vmnic# devices or MAC address to set as the uplinks.(Ex: uplinks: ["vmnic0", "2c:60:0c:ad:d5:ba"]). If an uplink is attached to a vSwitch, it will be removed from the old vSwitch before being added to the vSwitch named by 'switchName'.
failoverPolicy String   *optional*   This can be one of the following options: explicit: Always use the highest order uplink from the list of active adapters which pass failover criteria. iphash: Route based on hashing the src and destination IP addresses mac: Route based on the MAC address of the packet source. portid: Route based on the originating virtual port ID.
============== ======== ============ ============================================

.. _bonds:

For **bonds** **(RHEL/CentOS only)** in payload:

=================== ======== ============ ============================================
Parameters          Type     Flags        Description
=================== ======== ============ ============================================
name                String   **required** The name of the bond. Example 'bond0'
nics                Array    *optional*   The array of server NICs that needs to be included in the bond.
bondvlaninterfaces  Array    *optional*   List of tagged sub-interfaces to be created associated with the bond interface
=================== ======== ============ ============================================

.. _bondvlaninterfaces:

For **bondvlaninterfaces** in payload, both ipv4 and ipv6 are supported

============== ======== ============ ============================================
Parameters     Type     Flags        Description
============== ======== ============ ============================================
vlanid         Number   **required**  VLAN ID to be associated with the tagged sub interface
ipv4           Object   *optional*    See `ipv4 or ipv6`_ more details.
ipv6           Object   *optional*    See `ipv4 or ipv6`_ more details.
============== ======== ============ ============================================


.. _windows-payload:

Windows OS Installation Workflow Payload
------------------------------------------

============= ======== ============ ============================================
Parameters    Type     Flags        Description
============= ======== ============ ============================================
productkey    String   **required** Windows License
domain        String   **optional** Windows domain
hostname      String   **optional** Windows hostname to be giving to the node after installation
smbUser       String   **required** Smb user for the share to which Windows' iso is mounted
smbPassword   String   **required** Smb password
repo          String   **required** The share to to which Windows' iso is mounted
============= ======== ============ ============================================

**Example of minimum payload**
https://github.com/RackHD/RackHD/blob/master/example/samples/install_windows_payload_minimal.json

**Example of full payload**
https://github.com/RackHD/RackHD/blob/master/example/samples/install_windows_payload_full.json

