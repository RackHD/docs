RAID Configuration
~~~~~~~~~~~~~~~~~~~~~~~

RackHD supports RAID configuration to create and delete RAID for hardwares with LSI RAID controller.

Create overlay with Storcli/Perccli
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

RackHD leverages LSI provided tool Storcli to configure RAID. RackHD requires user to build overlay including Storcli.
As on how to build overlay for RakcHD, please refer to https://github.com/RackHD/on-imagebuilder.
Perccli is a Dell tool which is based on Storcli and has the same commands with it.
If user wants to configure RAID on Dell servers, Perccli instead of Storcli should be built in overlay.
The newly built overlay (default named "dell.raid.overlay.cpio.gz" for Dell and "raid.overlay.cpio.gz" for others) should be put in RackHD static file path.

Create RAID
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
An example of creating RAID workflow is as below:

.. code-block:: REST

    curl -X POST \
         -H 'Content-Type: application/json' \
         -d @params.json \
         <server>/api/current/nodes/<identifier>/workflows?name=Graph.Raid.Create.MegaRAID'


An example of params.json with minimal parameters for creating RAID workflow:

.. literalinclude:: samples/create-raid.json
   :language: JSON

For details on items of create-raid.options, please refer to: https://github.com/RackHD/on-tasks/blob/master/lib/task-data/schemas/create-megaraid.json.

**Note**:

* User need make sure drives are under UGOOD status before creating RAID. If drives are under other status (JBOD, online/offline or UBAD), RackHD won't be able to create RAID with them.
* For Dell servers, overlay tool path should be specified in param.json as below:

.. literalinclude:: samples/create-raid-dell.json
   :language: JSON


Delete RAID
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

An example of deleting RAID workflow is as below:

.. code-block:: REST

    curl -X POST \
         -H 'Content-Type: application/json' \
         -d @params.json \
         <server>/api/current/nodes/<identifier>/workflows?name=Graph.Raid.Delete.MegaRAID'


An example of params.json for deleting RAID workflow:

.. code-block:: JSON

    {
        "options": {
            "delete-raid": {
                "raidIds": [0, 1]
            },
            "bootstrap": {
                "overlayfsFile": "raid.overlay.cpio.gz"
            }
        }
    }


"raidIds" is the virtual disk id to be deleted.

For Dell servers, the payload should look like:

.. code-block:: JSON

    {
        "options": {
            "delete-raid": {
                "path": "/opt/MegaRAID/perccli/percli64",
                "raidIds": [0, 1]
            },
            "bootstrap": {
                "overlayfsFile": "dell.raid.overlay.cpio.gz"
            }
        }
    }

