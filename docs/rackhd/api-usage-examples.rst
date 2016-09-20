Rest-2.0 API examples
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


Firmware update Example using SKU Pack
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This example provides instructions on how to flash a BMC image on a Quanta (node) using SKU Pack.

1. Wait for discovery to complete and Get nodes to check if node has been discovered successfully

     **Get Nodes**
     ::
     
         GET /api/2.0/nodes
     ::
     
         curl <server>/api/2.0/nodes


2. Post the obm settings if they don't already exist for the node. An example on how to do this is shown in Section 7.1.8.1 here http://rackhd.readthedocs.io/en/latest/tutorials/vagrant.html#adding-a-sku-definition Section 7.1.8.1


3. The firmware files need to be built into a SKU package
  
     **Build SKU Package**
     ::
     
         $ ./build-package.bash <sku_pack_directory> <subname>

   The new tarball is built in a folder called tarballs within the same directory 
    
     ::
        
        $ /tarballs 
          sku_pack_directory_subname.tar.gz
          
4. The SKU package that was built needs to be registered            

     **POST the tarball**
     ::
     
        curl -X POST --data-binary @tarballs/sku_pack_directory_subname.tar.gz localhost:8080/api/current/skus/pack
   
   The above command will return a SKU ID which will be used to post the Workflow against the node. If an error like "Duplicate entry
   exists" is returned in place of the the SKU ID, check the database and delete a prexisting SKU package.
   
5. The pollers associated with the node need to be paused before POST'ing the Workflow to flash a new BMC image. This is needed to avoid seeing any poller errors in the log while BMC is offline. Further information on IPMI poller properties can be found at http://rackhd.readthedocs.io/en/latest/rackhd/pollers.html?highlight=ipmi%20pollers

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

6. The workflow to flash a new BMC image to a Quanta node needs to be POST'ed

     **POST Workflow**
     
       .. code-block:: REST

          POST /api/2.0/nodes/:id/workflows?name=Graph.Flash.Quanta.Bmc::sku.ID

       .. code-block:: REST

          curl -X POST <server>/api/2.0/nodes/<nodeid>/workflows?name=Graph.Flash.Quanta.Bmc::sku.ID
          
7. Check if any active workflows on that node exist to make sure the workflow has completed
   
      **GET active Workflow**
          
          .. code-block:: REST

          GET /api/2.0/nodes/<id>/workflows/active

          .. code-block:: REST

          curl <server>/api/2.0/nodes/<id>/workflows/active
          
          
If a remote viewing session exists for the node, check the BMC firmware to verify the version has been updated.      
      
