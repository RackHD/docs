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


Stop the active OS installation workflow:

.. code-block:: REST

    curl -X DELETE <server>/api/1.1/nodes/<identifier>/workflows/active


OS Installation Workflow Payload
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

All parameters descriptions of OS installation workflow payload are listed below , they are fit to all supported OSes:

=================== ================ ============ ============================================
Parameters          Type              Flags       Description
=================== ================ ============ ============================================
version             String           **required** The version number of target OS that needs to install. **NOTE**: For Ubuntu, the version should be 'trusty' not '14.04'
repo                String           **required** The OS repository address, currently only supports HTTP
rootPassword        String           *optional*   The password for the OS root account, default *rootPassword* is  **"RackHDRocks!"**
hostname            String           *optional*   The hostname for target OS, default *hostname* is **"localhost"**
domain              String           *optional*   The domain for target OS, default *domain* is **"rackhd"**
users               Array            *optional*   If specified, this contains an array of objects, each object contains the user account information that will be created after OS installation. 0, 1, or multiple users could be specified.  If *users* is omitted, null or empty, no user will be created.
dnsServers          Array            *optional*   If specified, this contains an array of string, each elements is the Domain Name Server, the first one will be primary, others are alternative.
networkDevices      Array            *optional*   The static IP setting for network devices after OS installation. If it is omitted, null or empty, RackHD will not touch any network devices setting, so all network devices remain at the default state (usually default is DHCP).If there is multiple setting for same device, RackHD will choose the last one as the final setting, both ipv4 and ipv6 are supported here.
rootSshKey          String           *optional*   The SSH key that will be appended to target OS.
installDisk         String/Number    *optional*   *installDisk* is to specify the target disk which the OS will be installed on. It can be a string or a number. **For string**, it is a disk path that the OS can recongize, its format varies with OS. For example, "/dev/sda" or "/dev/disk/by-id/scsi-36001636121940cc01df404d80c1e761e" for CentOS/RHEL, "t10.ATA_____SATADOM2DSV_3SE__________________________20130522AA0990120088" or "naa.6001636101840bb01df404d80c2d76fe" or "mpx.vmhba1:C0:T1:L0" or "vml.0000000000766d686261313a313a30" for ESXi. **For number**, it is a RackHD generated disk identifier (it could be obtained from "driveId" catalog). If *installDisk* is omitted, RackHD will assign the default disk by order: SATADOM -> first disk in "driveId" catalog -> "sda" for Linux OS.
kvm                 Boolean          *optional*   The value is *true* or *false* to indicates to install kvm or not, default is **false**.
switchDevices       Array            *optional*   **(ESXi only)** If specified, this contains an array of objects with switchName and uplinks (optional) parameters. If *uplinks* is omitted, null or empty, the vswitch will be created with no uplinks. 
postInstallCommands Array            *optional*   **(ESXi only)** If specified, this contains an array of string commands that will be run at the end of the post installation step.  This can be used by the customer to tweak final system configuration.

=================== ================ ============ ============================================

For **users** in payload:

=========== ======== ============ ============================================
Parameters  Type     Flags        Description
=========== ======== ============ ============================================
name        String   **required** The name of user
password    String   **required** The password of user, it could be clear text, RackHD will do encryption before store it into kickstart file.
uid         Number   **required** The unique identifier of user. **required** for non-ESXi OS, Not support for ESXi OS
sshKey      String   *optional*   The SSH key that will be appended into target OS.
=========== ======== ============ ============================================


For **networkDevices** in payload, both ipv4 and ipv6 are supported

============== ======== ============ ============================================
Parameters     Type     Flags        Description
============== ======== ============ ============================================
device         String   **required**  Network device name in target OS (ex. "eth0", "enp0s1")
ipv4           Object   *optional*    The object contains ipv4 configurations
ipv6           Object   *optional*    The object contains ipv6 configurations
esxSwitchName  String   *optional*    **(ESXi only)** The vswitch to attach the vmk device to. vSwitch0 is used by default if no esxSwitchName is specified.
============== ======== ============ ============================================


For **ipv4** or **ipv6** configurations:

=========== ======== ============ ============================================
Parameters  Type     Flags        Description
=========== ======== ============ ============================================
ipAddr      String   **required** The assigned static IP address
gateway     String   **required** The gateway.
netmask     String   **required** The subnet mask.
vlanId      Number   *optional*   The VLAN ID. This is an array of integers (0-4095)
=========== ======== ============ ============================================


For **switchDevices** **(ESXi only)** in payload:

=========== ======== ============ ============================================
Parameters  Type     Flags        Description
=========== ======== ============ ============================================
switchName  String   **required** The name of the vswitch
uplinks     String   *optional*   The array of vmnic# devices to set as the uplinks.(Ex: uplinks: ["vmnic0", "vmnic1"]). If an uplink is attached to a vSwitch, it will be removed from the old vSwitch before being added to the vSwitch named by 'switchName'.
=========== ======== ============ ============================================
