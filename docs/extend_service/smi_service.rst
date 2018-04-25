SMI Service
========================

Introduction
------------------------

The System Management Integration (SMI) Microservices are add-on services that are used by RackHD workflows and tasks, primarily focused on adding value for the managemenet of Dell servers. These services use a Zuul gateway and Consul Registry service to present a unified API. Documentation for each service is avialiable on Github in repositories that begin with "smi-service" or on the dockerhub page for the service.

How to start
------------------------

1. Clone the RackHD repo if you don't already have it,
and change into the "rackhd/docker/dell" folder

.. code::

    git clone http://github.com/rackhd/rackhd
    cd rackhd/docker/dell

2. Edit the .env file with your IP addresses.

* By default the IP addresses are set to 172.31.128.1 to match the default southbound IP for RackHD.
* Optionally, if you wish to have available the PDF generation feature of the swagger-aggregator, the "HOST_IP" setting in the .env file should be changed to your "Northbound" IP.

3. Start Consul only in detached mode

.. code::

    sudo docker-compose up -d consul

You can view the consul UI by navigating to http://<your_HOST_IP_address>:8500

4. Post in microservice key/value properties into consul

.. code::

    ./set_config.sh

You can view the key/value data in consul by clicking on the Key/Value tab.

5. Start remaining containers (or just the ones you want to start) in detached mode

Note: Not all the microservices need to run. You have the option of starting only the ones needed, or manually editing the docker-compose.yml file.
.. code::

    sudo docker-compose up -d

It takes about 2 minutes for the services to come up. To start just the containers you want, specify the names of the containers to start at the end of the command seperated by a space.

6. Verify your services are online
.. code::

    sudo docker-compose ps

You can also look for your services to register in the consul UI

7. Config smiConfig.json for RackHD
.. code::

    ./set_rackhd_smi_config.sh

SMI Workflows
------------------------

============================================== ===================================================================================
Workflow Name                                  Description
============================================== ===================================================================================
Graph.Dell.Wsman.GetInventory                  Get inventory
Graph.Dell.Wsman.Configure.Idrac               Configure IDRAC, including IP, netmask, gateway
Graph.Dell.Wsman.GetSystemComponentsCatalog    Get server system configuration
Graph.Dell.Wsman.UpdateSystemComponents        Update server system configuration
Graph.Dell.Wsman.Add.Volume                    Add new RAID virtual disk
Graph.Dell.Wsman.Delete.Volume                 Delete RAID virtual disk
Graph.Dell.Wsman.Add.Hotspare                  Add new HotSpare for RAID virtual disk
Graph.Dell.Wsman.Discovery                     Discovery by scanning the IDRAC IP ranges
Graph.Dell.Wsman.PostDiscovery                 Tasks run after discovery
Graph.Dell.Wsman.Os.Create                     Read files from a source ISO and create a new, repackaged ISO that specifies the location of a Kickstart file to use
Graph.Dell.Wsman.Os.Deploy                     Deploy an ISO image stored on a network share to to a Dell server
Graph.Dell.Wsman.ConfigServices                Configure smiConfig.json
Graph.Dell.Wsman.Create.Repo                   Create firmware repo
Graph.Dell.Wsman.Download.Catalog              Download catalog
Graph.Dell.Wsman.Simple.Update.Firmware        Use firmware image to update single component's firmware
Graph.Dell.Wsman.Update.Firmware               Use firmware repo to update all components' firmware
Graph.Dell.Wsman.Import.SCP                    Import system configuration from a file located on remote share
Graph.Dell.Wsman.Export.SCP                    Export system configuration to a file on a remote share
Graph.Dell.Wsman.GetBios                       Get BIOS inventory
Graph.Dell.Wsman.ConfigureBios                 Configure BIOS settings
Graph.Dell.Wsman.GetTrapConfig                 Get server trap config
Graph.Dell.Wsman.Configure.Redfish.Alert       Configure redfish alert
Graph.Dell.Wsman.Reset.Components              Reset components, such as bios, diag, drvpack, idrac, lcdata
Graph.Dell.Wsman.Powerthermal                  Set Power Cap Policy
============================================== ===================================================================================


Run Workflow Example
------------------------

**Run Discovery Workflow Example**

.. code-block:: REST

   curl -X POST \
        -H 'Content-Type: application/json' \
        -d '{ "name":"Graph.Dell.Wsman.Discovery",
              "options": {
                  "defaults": {
                      "ranges": [
                        {
                            "startIp": "<startIP>",
                            "endIp": "<endIp>",
                            "credentials": {
                                "userName": "<user>",
                                "password": "<password."
                            }
                        }
                      ],
                      "inventory": "true"
                  },
              }
            }' \
        <server>/api/2.0/workflows

**Run ConfigureBios Workflow Example**

.. code-block:: REST

   curl -X POST \
        -H 'Content-Type: application/json' \
        -d '{ "name":"Graph.Dell.Wsman.ConfigureBios",
              "options": {
                  "defaults": {
                      "attributes": [{
                          "name": "NumLock",
                          "value": "On"
                      }],
                      "rebootJobType": 1
                  },
              }
            }' \
        <server>/api/2.0/nodes/<nodeId>/workflows
