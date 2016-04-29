Customize Default iPXE Boot Setting
---------------------------

When a node's NIC's PXE boot is set to the first boot order in BIOS, default boot settings could be customized by iPXE in RackHD, node will boot with default boot settings after node is discovered by RackHD.

Default iPXE Boot To Customized OS in RAM
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In some use cases, node wants to boot customized kernel and initrd to OS in RAM, not boot into a installed distribution OS in disk. This functionality could be provided by `PATCH` **bootSettings** to a node by API command:

.. code-block:: REST

    curl -X PATCH \
        -H 'Content-Type: application/json' \
        -d @boot.json \
        <server>/api/1.1/nodes/<identifier>


A example of boot.json:

.. literalinclude:: samples/boot-settings.json
   :language: JSON

For **bootSettings**, **profile** and **options** are **MUST** required:

======== =========== ============ ============================================
Name       Type         Flags       Description
======== =========== ============ ============================================
profile   String     **required**  Profile that will be rendered by RackHD and used by iPXE
options   Object     **required**  Options in JSON format used to render variables in `profile`
======== =========== ============ ============================================


A default iPXE **profile** `defaultboot.ipxe` is provided by RackHD, and its **options** includes `url`, `kernel`, `initrd`, `bootargs`

======== =========== ============ ============================================
Name       Type         Flags       Description
======== =========== ============ ============================================
url       String     **required**  Location Link of `kernel` and `initrd`, it could be accessed by http in node, the http service is located in RackHD server or an external server which could be accessed by http proxy or after setting NAT in RackHD. In RackHD server, the root location could be set by `httpStaticRoot` in `config.json <https://github.com/RackHD/RackHD/blob/master/packer%2Fansible%2Froles%2Fmonorail%2Ffiles%2Fconfig.json>`_ or in SKU Pack's config.json
kernel    String     **required**  Kernel to boot
initrd    String     **required**  Init ramdisk to boot with `kernel`
bootargs  String     **required**  Boot arguments of `kernel`
======== =========== ============ ============================================


Customize iPXE Boot Profile
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**profile** in **bootSettings** could be customized instead of `defaultboot.ipxe`. `defaultboot.ipxe` is provided by default, and its **options** `url`, `kernel`, `initrd`, `bootargs` are aligned with the variables `<%=url%>` `<%=kernel%>` `<%=initrd%>` `<%=bootargs%>` in `defaultboot.ipxe`, so if the profile is customized, the options also should be aligned with the variables that will be rendered in customized iPXE profile just like `defaultboot.ipxe`

**defaultboot.ipxe**:

.. code-block:: REST

    kernel <%=url%>/<%=kernel%>
    initrd <%=url%>/<%=initrd%>
    imgargs <%=kernel%> <%=bootargs%>
    boot || prompt --key 0x197e --timeout 2000 Press F12 to investigate || exit shell

