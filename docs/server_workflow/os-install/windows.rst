Windows Installation
=======================

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



Get payload example:

.. code-block:: shell

    wget https://raw.githubusercontent.com/RackHD/RackHD/master/example/samples/install_windows_payload_minimal.json

Call API to install OS:

.. code-block:: shell

    curl -X POST -H 'Content-Type: application/json' -d install_windows_payload_minimal.json 127.0.0.1:9090/api/current/nodes/{node-id}/workflows?name=Graph.InstallWindowsServer | jq '.'


.. note::

    For more detail about payload file please refer to :ref:`windows-payload`
