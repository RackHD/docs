OS Installation Workflow Support
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

RackHD workflow support installing Operating System automatically from remote repository that node could access by HTTP protocol.

Supported OS Installation Workflows
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Supported OSes and their workflows are listed in table, and the listed versions have been verified by RackHD, but not limited to these, when more versions are verified, this table will be updated.

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


A example of params.json with minimal parameters for installing CentOS workflow:

.. literalinclude:: samples/install-os-minimal-parameters.json
   :language: JSON

A example of params.json with full set of parameters for installing CentOS workflow:

.. literalinclude:: samples/install-os-maximal-parameters.json
   :language: JSON

Check the OS installation workflow is active or inactive:

.. code-block:: REST

    curl <server>/api/1.1/nodes/<identifier>/workflows/active


Delete the active OS installation workflow:

.. code-block:: REST

    curl -X DELETE <server>/api/1.1/nodes/<identifier>/workflows/active


OS Installation Workflow Payload
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

All parameters descriptions of OS installation workflow payload are listed below , they are fit to all supported OS installations:

=================== ================ ============ ============================================
Parameters          Type              Flags        Description
=================== ================ ============ ============================================
version             String           **required** The version number of target OS that needs to install. **NOTE**: For Ubuntu, the version should be 'trusty' not '14.04'.
repo                String           **required** The OS repository address, currently only supports HTTP
rootPassword        String           **required** The password for the OS root account
hostname            String           **required** The hostname for target OS
domain              String           **required** The domain for target OS
users               Array            *optional*   If specified, this contains an array of objects, each object contains the user account information that will be created after OS installation. 0, 1, or multiple users could be specified. If *users* is omitted, null or 0 length, no user will be created.
dnsServers          Array            *optional*   If specified, this contains an array of objects, each object contains the Domain Name Servers, multiple DNS servers could be specified, the first one will be primary, others are alternative.
networkDevices      Array            *optional*   The static IP setting for network devices after OS installation. If it is omitted, null or 0 length, RackHD will not touch any network devices setting, so all network devices remain at the default state (usually default is DHCP).If there is multiple setting for same device, RackHD will choose the last one as the final setting, both ipv4 and ipv6 are supported here.
rootSshKey          String           *optional*   The SSH key that will be appended into the file /root/.ssh/authorized_keys in target OS.
installDisk         String/Number    *optional*   The value could be string or integer number. For string, it could be the value like '/dev/sda' or the value of 'esxiWwid'(for ESXi), 'linuxWwid'(for Linux) , For number, it's the value of 'identifier' index. If *installDisk* not specified, OS will be installed into default SATADOM disk, and if there's no SATADOM disk, OS will be installed into 'identifier' 0 disk. **NOTE:** the value of 'esxiWwid', 'linuxWwid', 'identifier' could be got from node catalog's 'driveId' source.
kvm                 Boolean          *optional*   The value is true/false, If *kvm* is omitted, null or 0 length, kvm will not be installed.  indicates to install kvm or not, default is false.
switchDevices       Array            *optional*   **(ESXi only)** If specified, this contains an array of objects with switchName and uplinks (optional) parameters.  If *uplinks* is omitted, null or 0 length, the vswitch will be created with no uplinks.
postInstallCommands Array            *optional*   **(ESXi only)** If specified, this contains an array of string commands that will be run at the end of the post installation step.  This can be used by the customer to tweak final system configuration

=================== ================ ============ ============================================

For **users** in payload:

=========== ======== ============ ============================================
Parameters  Type     Flags        Description
=========== ======== ============ ============================================
name        String   **required** The name of user
password    String   **required** The password of user, it could be clear text, RackHD will do encryption before store it into kickstart file.
uid         Number   **required** The unique identifier of user.
sshKey      String   *optional*   The SSH key that will be appended into the file ~/.ssh/authorized_keys in target OS.
=========== ======== ============ ============================================

For **networkDevices** in payload, both ipv4 and ipv6 are supported:

=========== ======== ============ ============================================
Parameters  Type     Flags        Description
=========== ======== ============ ============================================
ipAddr      String   **required** The assigned static IP address
gateway     String   **required** The gateway.
netmask     String   **required** The subnet mask.
vlanId      String   *optional*   The VLAN ID. This is an array of integers (0-4095)
=========== ======== ============ ============================================


For **switchDevices** **(ESXi only)** in payload:

=========== ======== ============ ============================================
Parameters  Type     Flags        Description
=========== ======== ============ ============================================
switchName  String   **required** The name of the vswitch
uplinks     String   *optional*   The array of vmnic# devices to set as the uplinks.(Ex: uplinks: ["vmnic0", "vmnic1"])

=========== ======== ============ ============================================

