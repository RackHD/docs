OS Installation Workflow Support
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

RackHD workflow support installing Operating System automatically from remote http repository.

Supported OS Installation Workflows
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Supported OSes and their workflows are listed in table, and the listed versions have been verified by RackHD, but not limited to these, this table will be updated when more versions are verified.

============= ==================== ==================================================
OS            Workflow             Version
============= ==================== ==================================================
ESXi          Graph.InstallEsx     5.5/6.0
RHEL          Graph.InstallRHEL    7.0
CentOS        Graph.InstallCentOS  6.5/7
Ubuntu        Graph.InstallUbuntu  trusty(14.04)
SUSE          Graph.InstallSUSE    openSUSE: leap/42.1, SLES: 11/12
CoreOS        Graph.InstallCoreOS  899.17.0
============= ==================== ==================================================


OS Installation Workflow APIs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Start an OS installation workflow:

.. code-block:: REST

    curl -X POST \
         -H 'Content-Type: application/json' \
         -d @params.json \
         <server>/api/1.1/nodes/<identifier>/workflows?name=Graph.InstallCentOS


An example of params.json with minimal parameters for installing CentOS workflow:

.. literalinclude:: samples/install-os-minimal-parameters.json
   :language: JSON

An example of params.json with full set of parameters for installing CentOS workflow:

.. literalinclude:: samples/install-os-maximal-parameters.json
   :language: JSON

Check the workflow is active or inactive:

.. code-block:: REST

    curl <server>/api/1.1/nodes/<identifier>/workflows/active


Stop the active workflow to cancel OS installation:

.. code-block:: REST

    curl -X DELETE <server>/api/1.1/nodes/<identifier>/workflows/active


OS Installation Workflow Payload
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

All parameters descriptions of OS installation workflow payload are listed below , they are fit to all supported OSes except CoreOS, **NOTE:** CoreOS only supports parameters *version* and *repo*.

=================== ================ ============ ============================================
Parameters          Type              Flags       Description
=================== ================ ============ ============================================
version             String           **required** The version number of target OS that needs to install. **NOTE**: For Ubuntu, *version* should be the codename, not numbers, for example, it should be "trusty", not "14.04"
repo                String           **required** The OS repository address, currently only supports HTTP. Some examples of free OS distributions for reference. For **CentOS**, http://mirror.centos.org/centos/7/os/x86_64/. For **Ubuntu**, http://us.archive.ubuntu.com/ubuntu/. For **openSUSE**, http://download.opensuse.org/distribution/leap/42.1/repo/oss/. For enterprise OS distributions like **ESXi**, **RHEL**, **SLES**, the repository is the directory of mounted DVD ISO image, and http service is provided for this directory. 
rootPassword        String           *optional*   The password for the OS root account, it could be clear text, RackHD will do encryption before store it into OS installer's config file. default *rootPassword* is  **"RackHDRocks!"**. Some OS distributions' password requirements *must* be satisfied. For **ESXi 5.5**, `ESXi 5 Password Requirements <https://pubs.vmware.com/vsphere-50/index.jsp?topic=%2Fcom.vmware.vsphere.security.doc_50%2FGUID-DC96FFDB-F5F2-43EC-8C73-05ACDAE6BE43.html&resultof=%22password%22%20>`_. For **ESXi 6.0**, `ESXi 6 Password Requirements <http://pubs.vmware.com/vsphere-60/index.jsp#com.vmware.vsphere.security.doc/GUID-4BDBF79A-6C16-43B0-B0B1-637BF5516112.html?resultof=%2522%2550%2561%2573%2573%2577%256f%2572%2564%2522%2520%2522%2570%2561%2573%2573%2577%256f%2572%2564%2522%2520%2522%2552%2565%2571%2575%2569%2572%2565%256d%2565%256e%2574%2573%2522%2520%2522%2572%2565%2571%2575%2569%2572%2522%2520>`_.
hostname            String           *optional*   The hostname for target OS, default *hostname* is **"localhost"**
domain              String           *optional*   The domain for target OS, default *domain* is **"rackhd"**
users               Array            *optional*   If specified, this contains an array of objects, each object contains the user account information that will be created after OS installation. 0, 1, or multiple users could be specified.  If *users* is omitted, null or empty, no user will be created. See users_ more details.
dnsServers          Array            *optional*   If specified, this contains an array of string, each elements is the Domain Name Server, the first one will be primary, others are alternative.
networkDevices      Array            *optional*   The static IP setting for network devices after OS installation. If it is omitted, null or empty, RackHD will not touch any network devices setting, so all network devices remain at the default state (usually default is DHCP).If there is multiple setting for same device, RackHD will choose the last one as the final setting, both ipv4 and ipv6 are supported here. See networkDevices_ more details.
rootSshKey          String           *optional*   The public SSH key that will be appended to target OS.
installDisk         String/Number    *optional*   *installDisk* is to specify the target disk which the OS will be installed on. It can be a string or a number. **For string**, it is a disk path that the OS can recongize, its format varies with OS. For example, "/dev/sda" or "/dev/disk/by-id/scsi-36001636121940cc01df404d80c1e761e" for CentOS/RHEL, "t10.ATA_____SATADOM2DSV_3SE__________________________20130522AA0990120088" or "naa.6001636101840bb01df404d80c2d76fe" or "mpx.vmhba1:C0:T1:L0" or "vml.0000000000766d686261313a313a30" for ESXi. **For number**, it is a RackHD generated disk identifier (it could be obtained from "driveId" catalog). If *installDisk* is omitted, RackHD will assign the default disk by order: SATADOM -> first disk in "driveId" catalog -> "sda" for Linux OS.
kvm                 Boolean          *optional*   The value is *true* or *false* to indicates to install kvm or not, default is **false**.
switchDevices       Array            *optional*   **(ESXi only)** If specified, this contains an array of objects with switchName and uplinks (optional) parameters. If *uplinks* is omitted, null or empty, the vswitch will be created with no uplinks. See switchDevices_ more details.
postInstallCommands Array            *optional*   **(ESXi only)** If specified, this contains an array of string commands that will be run at the end of the post installation step.  This can be used by the customer to tweak final system configuration.

=================== ================ ============ ============================================

.. _users:

For **users** in payload:

=========== ======== ============ ============================================
Parameters  Type     Flags        Description
=========== ======== ============ ============================================
name        String   **required** The name of user
password    String   **required** The password of user, it could be clear text, RackHD will do encryption before store it into OS installer's config file. Some OS distributions' password requirements *must* be satisfied. For **ESXi 5.5**, `ESXi 5 Password Requirements <https://pubs.vmware.com/vsphere-50/index.jsp?topic=%2Fcom.vmware.vsphere.security.doc_50%2FGUID-DC96FFDB-F5F2-43EC-8C73-05ACDAE6BE43.html&resultof=%22password%22%20>`_. For **ESXi 6.0**, `ESXi 6 Password Requirements <http://pubs.vmware.com/vsphere-60/index.jsp#com.vmware.vsphere.security.doc/GUID-4BDBF79A-6C16-43B0-B0B1-637BF5516112.html?resultof=%2522%2550%2561%2573%2573%2577%256f%2572%2564%2522%2520%2522%2570%2561%2573%2573%2577%256f%2572%2564%2522%2520%2522%2552%2565%2571%2575%2569%2572%2565%256d%2565%256e%2574%2573%2522%2520%2522%2572%2565%2571%2575%2569%2572%2522%2520>`_.
uid         Number   **required** The unique identifier of user. **required** for non-ESXi OS, Not support for ESXi OS
sshKey      String   *optional*   The public SSH key that will be appended into target OS.
=========== ======== ============ ============================================

.. _networkDevices:

For **networkDevices** in payload, both ipv4 and ipv6 are supported

============== ======== ============ ============================================
Parameters     Type     Flags        Description
============== ======== ============ ============================================
device         String   **required**  Network device name in target OS (ex. "eth0", "enp0s1")
ipv4           Object   *optional*    See `ipv4 or ipv6`_ more details.
ipv6           Object   *optional*    See `ipv4 or ipv6`_ more details.
esxSwitchName  String   *optional*    **(ESXi only)** The vswitch to attach the vmk device to. vSwitch0 is used by default if no esxSwitchName is specified.
============== ======== ============ ============================================

.. _`ipv4 or ipv6`:

For **ipv4** or **ipv6** configurations:

=========== ======== ============ ============================================
Parameters  Type     Flags        Description
=========== ======== ============ ============================================
ipAddr      String   **required** The assigned static IP address
gateway     String   **required** The gateway.
netmask     String   **required** The subnet mask.
vlanId      Number   *optional*   The VLAN ID. This is an array of integers (0-4095)
=========== ======== ============ ============================================


.. _switchDevices:

For **switchDevices** **(ESXi only)** in payload:

=========== ======== ============ ============================================
Parameters  Type     Flags        Description
=========== ======== ============ ============================================
switchName  String   **required** The name of the vswitch
uplinks     String   *optional*   The array of vmnic# devices to set as the uplinks.(Ex: uplinks: ["vmnic0", "vmnic1"]). If an uplink is attached to a vSwitch, it will be removed from the old vSwitch before being added to the vSwitch named by 'switchName'.
=========== ======== ============ ============================================
