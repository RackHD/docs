OS Installation
~~~~~~~~~~~~~~~~~~~~~~~

RackHD workflow support installing Operating System automatically from remote http repository.

Setting up RackHD OS repository with image service
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Image servie provides a convinient method for users to setup OS repository, details please refer to https://github.com/RackHD/image-service.
Alternatively, users could setup the repos by executing `Configuring RackHD OS Mirrors`_ and `Making the Mirrors`_.

Configuring RackHD OS Mirrors
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Setting up a Windows OS repo**

* **Mounting the OS Image**:

Windows' installation requires that Windows OS' ISO image must be mounted to a directory accessable to the node.
In the example below a windows server 2012 ISO image is being mounted to a directory name **Licensedwin2012**

.. code-block:: REST

    sudo mount -o loop /var/renasar/on-http/static/http/W2K2012_2015-06-08_1040.iso /var/renasar/on-http/static/http/Licensedwin2012

* **Export the directory**

Edit the samba config file in order to export the shared directory

.. code-block:: REST

    sudo nano /etc/samba/smb.conf

.. code-block:: REST

    [windowsServer2012]
        comment = not windows server 201
        path = /var/renasar/on-http/static/http/Licensedwin2012
        browseable = yes
        guest ok = yes
        writable = no
        printable = no


* **Restart the samba share**

.. code-block:: REST

    sudo service samba restart

**MIRRORS**

Mirrors may not be something you need to configure if you're utilizing the proxy capabilities
of RackHD. If you previously configured proxies to support OS installation workflows,
then those should be configured to provide access to the files needed. If you do not, or can
not, utilize proxies, then you can host the OS installation files directly on the same
instance with RackHD. The following instructions show how to create OS installation mirrors
in support of the existing workflows.

**Set the Links to the Mirrors:**

  .. code::

    sudo ln -s /var/mirrors/ubuntu <on-http directory>/static/http/ubuntu
    sudo ln -s /var/mirrors/ubuntu/14.04/mirror/mirror.pnl.gov/ubuntu/ <on-http directory>/static/http/trusty
    sudo ln -s /var/mirrors/centos <on-http directory>/static/http/centos
    sudo ln -s /var/mirrors/suse <on-http directory>/static/http/suse

Making the Mirrors
^^^^^^^^^^^^^^^^^^

**Centos 6.5**

Notes: Because CentOS 6.5 was deprecated by provider, the following rsync source is not available to
make mirror. Just leave it for an example.

  .. code::

    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" \
    --exclude "i386" rsync://centos.eecs.wsu.edu/centos/6.5/ /var/mirrors/centos/6.5

**Centos 7.0**

  .. code::

    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" \
    --exclude "i386" rsync://centos.eecs.wsu.edu/centos/7/ /var/mirrors/centos/7

**OpenSuse 12.3**

  .. code::

    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/distribution/12.3/ /var/mirrors/suse/distribution/12.3
    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/12.3 /var/mirrors/suse/update
    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/12.3-non-oss /var/mirrors/suse/update

**OpenSuse 13.1**

  .. code::

    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/distribution/13.1/ /var/mirrors/suse/distribution/13.1
    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/13.1 /var/mirrors/suse/update
    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/13.1-non-oss /var/mirrors/suse/update

**OpenSuse 13.2**

  .. code::

    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/distribution/13.2/ /var/mirrors/suse/distribution/13.2
    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/13.2 /var/mirrors/suse/update
    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/13.2-non-oss /var/mirrors/suse/update

**Ubuntu**

**[IMPORTANT]** DNS server is required in Ubuntu installation, make sure you have put following lines in /etc/dhcp/dhcpd.conf. 172.31.128.1 is a default option in RackHD

  .. code::

    option domain-name-servers 172.31.128.1;
    option routers 172.31.128.254;

**[Method-1]** For **iso** installation, see this payload https://github.com/RackHD/RackHD/blob/master/example/samples/install_ubuntu_payload_iso_minimal.json Remember to replace {{ file.server }} with your own, see 'fileServerAddress' and 'fileServerPort' in /opt/monorail/config.json

  .. code::

    mkdir ~/iso && cd !/iso

    # Download iso file
    wget http://releases.ubuntu.com/13.04/ubuntu-14.04.5-server-amd64.iso

    # Create mirror folder
    mkdir -p /var/mirrors/ubuntu

    # Replace {on-http-dir} with your own
    mkdir -p {on-http-dir}/static/http/mirrors

    # Mount iso
    sudo mount ubuntu-14.04.5-server-amd64.iso /var/mirrors/ubuntu

    sudo ln -s /var/mirrors/ubuntu {on-http-dir}/static/http/mirrors/

    # Create workflow
    # Replace the 9090 port if you are using other ports
    # You can configure the port in /opt/monorail/config.json -> 'httpEndPoints' -> 'northbound-api-router'
    curl -X POST -H 'Content-Type: application/json' -d @install_ubuntu_payload_iso_minimal.json 127.0.0.1:9090/api/current/nodes/{node-id}/workflows?name=Graph.InstallUbuntu | jq '.'


**[Method-2]** For **live** installation, see this payload https://github.com/RackHD/RackHD/blob/master/example/samples/install_ubuntu_payload_minimal.json Remember to replace **repo** with your own **{fileServerAddress}:{fileServerPort}/ubuntu**, you can find the proper parameters in /opt/monorail/config.json

Add following block into httpProxies in /opt/monorail/config.json

  .. code::

    {
      "localPath": "/ubuntu",
      "server": "http://us.archive.ubuntu.com/",
      "remotePath": "/ubuntu/"
    }

  .. code::

    # Create workflow
    # Replace the 9090 port if you are using other ports
    # You can configure the port in /opt/monorail/config.json -> 'httpEndPoints' -> 'northbound-api-router'
    curl -X POST -H 'Content-Type: application/json' -d @install_ubuntu_payload_minimal.json 127.0.0.1:9090/api/current/nodes/{node-id}/workflows?name=Graph.InstallUbuntu | jq '.'


**[Method-3]** For the **Ubuntu repo**, you need some additional installation. The mirrors are easily made on Ubuntu, but not so easily replicated on other OS. On any recent distribution of Ubuntu:

  .. code::

	# make the mirror directory (can sometimes hit a permissions issue)
	sudo mkdir -p /var/mirrors/ubuntu/14.04/mirror
	# create a file in /etc/apt/mirror.list (config below)
	sudo vi /etc/apt/mirror.list
	# run the mirror
	sudo apt-mirror


    ############# config ##################
    #
    set base_path    /var/mirrors/ubuntu/14.04
    #
    # set mirror_path  $base_path/mirror
    # set skel_path    $base_path/skel
    # set var_path     $base_path/var
    # set cleanscript $var_path/clean.sh
    # set defaultarch  <running host architecture>
    # set postmirror_script $var_path/postmirror.sh
    # set run_postmirror 0
    set nthreads     20
    set _tilde 0
    #
    ############# end config ##############

    deb-amd64 http://mirror.pnl.gov/ubuntu trusty main
    deb-amd64 http://mirror.pnl.gov/ubuntu trusty-updates main
    deb-amd64 http://mirror.pnl.gov/ubuntu trusty-security main
    clean http://mirror.pnl.gov/ubuntu

    #end of file
    ###################

**Reference: Here are some useful links to vendor's official website about Mirros setup.**

* **CentOS**: `Creating Local Mirrors for Updates or Installs <https://wiki.centos.org/HowTos/CreateLocalMirror>`_

Supported OS Installation Workflows
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Supported OSes and their workflows are listed in table, and the listed versions have been verified by RackHD, but not limited to these, this table will be updated when more versions are verified.

============= =========================== =================================================
OS            Workflow                     Version
============= =========================== =================================================
ESXi          Graph.InstallESXi            5.5/6.0
RHEL          Graph.InstallRHEL            7.0/7.1/7.2
CentOS        Graph.InstallCentOS          6.5/7
Ubuntu        Graph.InstallUbuntu          trusty(14.04)/xenial(16.04)/artful(17.10)
SUSE          Graph.InstallSUSE            openSUSE: leap/42.1, SLES: 11/12
CoreOS        Graph.InstallCoreOS          899.17.0
Windows       Graph.InstallWindowsServer   Server 2012
PhotonOS      Graph.InstallPhotonOS        1.0
============= =========================== =================================================


OS Installation Workflow APIs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

An example of starting an OS installation workflow for CentOS:

.. code-block:: REST

    curl -X POST \
         -H 'Content-Type: application/json' \
         -d @params.json \
         <server>/api/current/nodes/<identifier>/workflows?name=Graph.InstallCentOS


An example of params.json with minimal parameters for installing CentOS workflow:

.. literalinclude:: samples/install-os-minimal-parameters.json
   :language: JSON

An example of params.json with full set of parameters for installing CentOS workflow:

.. literalinclude:: samples/install-os-maximal-parameters.json
   :language: JSON

There are few more payload examples at https://github.com/RackHD/RackHD/tree/master/example/samples.

Check the active workflow running on a node

::

    curl <server>/api/current/nodes/<identifier>/workflows?active=true


Deprecated 1.1 API - Check the active workflow running on a node

.. code-block:: REST

    curl <server>/api/1.1/nodes/<identifier>/workflows/active

Stop the currently active workflow on a node:

::

    curl -X PUT \
    -H 'Content-Type: application/json' \
    -d '{"command": "cancel"}' \
    <server>/api/current/nodes/<id>/workflows/action

Deprecated 1.1 API - Stop the active workflow to cancel OS installation:

.. code-block:: REST

    curl -X DELETE <server>/api/1.1/nodes/<identifier>/workflows/active




Non-Windows OS Installation Workflow Payload
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

All parameters descriptions of OS installation workflow payload are listed below, they are fit for use with all supported OSes except for CoreOS (see note below).

**NOTE:** The CoreOS installer is pretty basic, and only supports certain parameters shown below. Configurations not directly supported by RackHD may still be made via a custom Ignition template. Typical parameters for CoreOS include: *version*, *repo*, and *installScriptUri*|*ignitionScriptUri* and optionally *vaultToken* and *grubLinuxAppend*.

=================== ================ ============ ============================================
Parameters          Type              Flags       Description
=================== ================ ============ ============================================
version             String           **required** The version number of target OS that needs to install. **NOTE**: For Ubuntu, *version* should be the codename, not numbers, for example, it should be "trusty", not "14.04"
repo                String           **required** The OS repository address, currently only supports HTTP. Some examples of free OS distributions for reference. For **CentOS**, http://mirror.centos.org/centos/7/os/x86_64/. For **Ubuntu**, http://us.archive.ubuntu.com/ubuntu/. For **openSUSE**, http://download.opensuse.org/distribution/leap/42.1/repo/oss/. For **ESXi**, **RHEL**, **SLES** and **PhotonOS**, the repository is the directory of mounted DVD ISO image, and http service is provided for this directory.
rootPassword        String           *optional*   The password for the OS root account, it could be clear text, RackHD will do encryption before store it into OS installer's config file. default *rootPassword* is  **"RackHDRocks!"**. Some OS distributions' password requirements *must* be satisfied. For **ESXi 5.5**, `ESXi 5 Password Requirements <https://pubs.vmware.com/vsphere-50/index.jsp?topic=%2Fcom.vmware.vsphere.security.doc_50%2FGUID-DC96FFDB-F5F2-43EC-8C73-05ACDAE6BE43.html&resultof=%22password%22%20>`_. For **ESXi 6.0**, `ESXi 6 Password Requirements <http://pubs.vmware.com/vsphere-60/index.jsp#com.vmware.vsphere.security.doc/GUID-4BDBF79A-6C16-43B0-B0B1-637BF5516112.html?resultof=%2522%2550%2561%2573%2573%2577%256f%2572%2564%2522%2520%2522%2570%2561%2573%2573%2577%256f%2572%2564%2522%2520%2522%2552%2565%2571%2575%2569%2572%2565%256d%2565%256e%2574%2573%2522%2520%2522%2572%2565%2571%2575%2569%2572%2522%2520>`_.
hostname            String           *optional*   The hostname for target OS, default *hostname* is **"localhost"**
domain              String           *optional*   The domain for target OS
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


Windows OS Installation Workflow Payload
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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

