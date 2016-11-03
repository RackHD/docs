Rest-2.0 API examples
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


Firmware update Example using SKU Pack
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This example provides instructions on how to flash a BMC image on a Quanta (node) using SKU Pack.

1. Wait for discovery to complete and get nodes to check if node has been discovered successfully

     **Get Nodes**
     ::
     
         GET /api/2.0/nodes
     ::
     
         curl <server>/api/2.0/nodes


2. Post the obm settings if they don't already exist for the node. An example on how to do this is shown in Section 7.1.8.1 here http://rackhd.readthedocs.io/en/latest/tutorials/vagrant.html#adding-a-sku-definition Section 7.1.8.1


3. Acquire BMC files and utilities from the vendor. Go to the Quanta directory, a sub-directory of the root folder of on-skupack, extract the BMC image and BMC upgrade executable into the static/bmc of the skupack and update the config.json with the md5sum of the firmware image.

4. The firmware files and update utilities need to be built into a SKU package
  
     **Build SKU Package**
     ::
     
         $ ./build-package.bash <sku_pack_directory> <subname>
     <sku_pack_directory> must be one of the directory names containing the node type on the root directory of on-skupack, e.g., it can be quanta-d51-1u, quanta-t41,dell-r630, etc, and <subname> can be any name a user likes. A {sku_pack_directory_subname}.tar.gz will be created in tarballs folder of the same directory.
    
     ::
        
        $ ls ./tarballs
          sku_pack_directory_subname.tar.gz
          
5. The SKU package that was built needs to be registered

     **POST the tarball**
     ::
     
        curl -X POST --data-binary @tarballs/sku_pack_directory_subname.tar.gz localhost:8080/api/current/skus/pack
   
   The above command will return a SKU ID. If an error like "Duplicate name found" is returned in place of the SKU ID, check the database and delete the preexisting SKU package.

6. The pollers associated with the node need to be paused before POST'ing the Workflow to flash a new BMC image. This is needed to avoid seeing any poller errors in the log while BMC is offline. Further information on IPMI poller properties can be found at http://rackhd.readthedocs.io/en/latest/rackhd/pollers.html?highlight=ipmi%20pollers

    **Get List of Active Pollers Associated With a Node**

       .. code-block:: REST

          GET /api/2.0/nodes/:id/pollers

       .. code-block:: REST

          curl <server>/api/2.0/nodes/<nodeid>/pollers
  
    **Update a Single Poller to pause the poller**

       .. code-block:: REST

           PATCH /api/2.0/pollers/:id
           {
                "paused": true
           }

       .. code-block:: REST

           curl -X PATCH \
              -H 'Content-Type: application/json' \
              -d '{"paused":true}' \
              <server>/api/2.0/pollers/<pollerid>  

7. The workflow to flash a new BMC image to a Quanta node needs to be POST'ed
If a user would upgrade a node without reboot at the end or run BMC upgrade with a file override, a user need add a payload when posting the workflow. Details please refer to the README.md under Quanta directory.

     **POST Workflow**
     
       .. code-block:: REST

          POST /api/2.0/nodes/:id/workflows?name=Graph.Flash.Quanta.Bmc

       .. code-block:: REST

          curl -X POST <server>/api/2.0/nodes/<nodeid>/workflows?name=Graph.Flash.Quanta.Bmc
          
8. Check if any active workflows on that node exist to make sure the workflow has completed
   
     **GET active Workflow**
          
       .. code-block:: REST

          GET /api/2.0/nodes/<id>/workflows/active

       .. code-block:: REST

          curl <server>/api/2.0/nodes/<id>/workflows/active
          
          
If a remote viewing session exists for the node, check the BMC firmware to verify the version has been updated.      
      
