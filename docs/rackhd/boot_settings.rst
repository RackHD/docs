Customize Default iPXE Boot Setting
------------------------------------

A compute server's BIOS can be set to always PXE network boot using the BIOS boot order. The default RackHD response when no workflow is operating is to do nothing - normally falling through to the next item in the BIOS boot order. RackHD can also be configured with a default iPXE script to provide boot instructions when no workflow is operational against the node.

.. container:: mytitle

    Default iPXE Boot Customized OS Into RAM

To configure RackHD to provide a custom iPXE response to a node outside of a workflow running, such as booting a customized kernel and initrd, you can do so by providing configuration to the Node resource in RackHD. This functionality can be enabled by using a PATCH REST API call adding **bootSettings** to a node.

.. code-block:: REST

    curl -X PATCH \
        -H 'Content-Type: application/json' \
        -d @boot.json \
        <server>/api/current/nodes/<identifier>


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


.. container:: mytitle

    Customize iPXE Boot Profile

**profile** in **bootSettings** could be customized instead of `defaultboot.ipxe`. `defaultboot.ipxe` is provided by default, and its **options** `url`, `kernel`, `initrd`, `bootargs` are aligned with the variables `<%=url%>` `<%=kernel%>` `<%=initrd%>` `<%=bootargs%>` in `defaultboot.ipxe`, so if the profile is customized, the options also should be aligned with the variables that will be rendered in customized iPXE profile just like `defaultboot.ipxe`

**defaultboot.ipxe**:

.. code-block:: REST

    kernel <%=url%>/<%=kernel%>
    initrd <%=url%>/<%=initrd%>
    imgargs <%=kernel%> <%=bootargs%>
    boot || prompt --key 0x197e --timeout 2000 Press F12 to investigate || exit shell
