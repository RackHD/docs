Discovering/Configuring Network Switches
---------------------------

Utilizing network switch installation environments like POAP (Cisco), ZTP (Arista) and ONIE (Cumulus, etc.), 
RackHD offers the capability to discover, inventory, and configure network switches during bootup.

Active Discovery
~~~~~~~~~~~~~~~~~~~~~~~~~

The terms "active discovery" and "passive discovery" are used by RackHD to differentiate between
a discovery workflow that occurs as part of a switch bootup process, and may potentially make
persistent changes to the switch operating system (active discovery), versus discovery workflow
that queries out of band endpoints against an already-configured switch without making
any persistent changes to it (e.g. SNMP polling).

During active discovery, by default the RackHD system will do light cataloging as part 
of the discovery process, generating enough data to identify the SKU/model of a switch in order
to dynamically generate workflows and templates specific to it.

For example, active discovery of a Cisco switch booting with POAP (Power On Auto-Provisioning) 
will create a catalog document with source "version" that SKU definitions can be built against:

.. code-block:: JSON

    {
        "node" : ObjectId("5708438c3bfc361c5cca74dc"),
        "source" : "version",
        "data" : {
            "kern_uptm_secs" : "2",
            "kick_file_name" : "bootflash:///n3000-uk9-kickstart.6.0.2.U5.2.bin",
            "rr_service" : null,
            "loader_ver_str" : "N/A",
            "module_id" : "48x10GT + 6x40G Supervisor",
            "kick_tmstmp" : "03/17/2015 10:50:07",
            "isan_file_name" : "bootflash:///n3000-uk9.6.0.2.U5.2.bin",
            "sys_ver_str" : "6.0(2)U5(2)",
            "bootflash_size" : "2007040",
            "kickstart_ver_str" : "6.0(2)U5(2)",
            "kick_cmpl_time" : "3/17/2015 2:00:00",
            "chassis_id" : "Nexus 3172T Chassis",
            "proc_board_id" : "FOC1928169X",
            "memory" : "3793756",
            "kern_uptm_mins" : "6",
            "bios_ver_str" : "2.0.0",
            "cpu_name" : "Intel(R) Pentium(R) CPU  @ 2.00GHz",
            "bios_cmpl_time" : "04/01/2014",
            "kern_uptm_hrs" : "0",
            "rr_usecs" : "981748",
            "isan_tmstmp" : "03/17/2015 12:29:49",
            "rr_sys_ver" : "6.0(2)U5(2)",
            "rr_reason" : "Reset Requested by CLI command reload",
            "rr_ctime" : "Fri Apr  8 23:35:28 2016",
            "header_str" : "Cisco Nexus Operating System (NX-OS) Software",
            "isan_cmpl_time" : "3/17/2015 2:00:00",
            "host_name" : "switch",
            "mem_type" : "kB",
            "kern_uptm_days" : "0",
            "power_seq_ver_str" : "Module 1: version v1.1"
        },
        "createdAt" : ISODate("2016-04-08T23:49:36.985Z"),
        "updatedAt" : ISODate("2016-04-08T23:49:36.985Z"),
        "_id" : ObjectId("57084390a2eb38385c3998b7")
    }



Extending the Active Discovery Workflow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

RackHD utilizes the ability of most switch installation environments to run python scripts.
This makes it easy to extend the active discovery process to produce custom catalogs, and deploy
switch configurations and boot images.

It will be helpful to understand the RackHD concepts of a SKU and a Workflow before
reading ahead.

SKU documentation: :ref:`sku-ref-label`

Workflow documentation: :ref:`workflows-ref-label`

In order to extend the discovery process, a SKU definition must be created and 
added to the system (see :ref:`sku-ref-label` ). An example SKU definition that matches the above
Cisco catalog might look like this:

.. code-block:: JSON

    {
        "name": "Cisco Nexus 3000 Switch - 54 port",
        "rules": [
            {
                "path": "version.chassis_id",
                "regex": "Nexus\\s\\d\\d\\d\\d\\w?\\sChassis"
            },
            {
                "path": "version.module_id",
                "equals": "48x10GT + 6x40G Supervisor"
            }
        ],
        "discoveryGraphName": "Graph.Switch.CiscoNexus3000.MyCustomWorkflow",
        "discoveryGraphOptions": {}
    }

Using the :code:`discoveryGraphName` field of the SKU definition, custom workflows
can be triggered during switch installation. Creation of these workflows is detailed below.

For the examples below, let's start with an empty workflow definition for our custom switch
workflow:

.. code-block:: JSON


    {
         "friendlyName": "My Custom Cisco Switch Workflow",
         "injectableName": "Graph.Switch.CiscoNexus3000.MyCustomWorkflow",
         "options": {},
         "tasks": []
    }

**Extending Cataloging**

To collect custom catalog data from the switch, a Python script must be created for each
catalog entry that can return either JSON or XML formatted data, and that is able to run on
the target switch (all imported modules must exist, and the syntax must be compatible 
with the switch OS' version of Python).

Custom Python scripts must execute their logic within a single :code:`main` function, that returns
the catalog data, for example the following script catalogs SNMP group information on a
Cisco Nexus switch:

**1. Define a cataloging script**

.. code-block:: Python

    def main():
         import json
         # Python module names vary depending on nxos version
         try:
             from cli import clid
         except:
             from cisco import clid
         data = {}
     
         try:
             data['group'] = json.loads(clid('show snmp group'))
         except:
             pass

         return data

*In this example, the cli module exists in the Nexus OS in order to run Cisco CLI commands.*

**2. Upload the script as a template**

Next, the script must be uploaded as a template to the RackHD server:

.. code-block:: Bash

    # PUT https://<server>:<port>/api/1.1/templates/library/cisco-catalog-snmp-example.py
    # via curl:
    curl -X PUT -H "Content-type: text/raw" -d @<script path> https://<server>:<port>/api/1.1/templates/library/cisco-catalog-snmp-example.py

**3. Add script to a workflow**

Scripts are sent to the switch to be run via the Linux Commands task, utilizing the
:code:`downloadUrl` option. More information on this task can be found in the 
documentation for the :ref:`linux-commands-ref-label`

After adding the cataloging script as a template, add a task definition to the custom workflow, so now it becomes:

.. code-block:: JSON


    {
         "friendlyName": "My Custom Cisco Switch Workflow",
         "injectableName": "Graph.Switch.CiscoNexus3000.MyCustomWorkflow",
         "options": {},
         "tasks": [
            {
                "label": "catalog-switch-config",
                "taskDefinition": {
                    "friendlyName": "Catalog Cisco Snmp Group",
                    "injectableName": "Task.Inline.Catalog.Switch.Cisco.SnmpGroup",
                    "implementsTask": "Task.Base.Linux.Commands",
                    "options": {
                        "commands": [
                            {
                                "downloadUrl": "/api/1.1/templates/cisco-catalog-snmp-example.py",
                                "catalog": { "format": "json", "source": "snmp-group" }
                            }
                        ]
                    },
                    "properties": {}
                },
            }
        ]
    }


**Deploying a startup config**

In order to deploy a startup config to a switch, another Python script needs to
be created that will download and copy the startup config, and a template must be created
for the startup config file itself.

The below Python script deploys a startup config to a Cisco Nexus switch during POAP:

.. code-block:: Python

    def main():
        # Python module names vary depending on nxos version
        try:
            from cli import cli
        except:
            from cisco import cli

        tmp_config_path = "volatile:poap.cfg"

        cli("copy <%=startupConfigUri%> %s vrf management" % tmp_config_path)
        cli("copy %s running-config" % tmp_config_path)
        cli("copy running-config startup-config")
        # copying to scheduled-config is necessary for POAP to exit on the next
        # reboot and apply the configuration
        cli("copy %s scheduled-config" % tmp_config_path)

The deploy script and startup config file should be uploaded via the templates API:

.. code-block:: Bash

    # Upload the deploy script
    # PUT https://<server>:<port>/api/1.1/templates/library/deploy-cisco-startup-config.py
    # via curl:
    curl -X PUT -H "Content-type: text/raw" -d @<deploy script path> https://<server>:<port>/api/1.1/templates/library/deploy-cisco-startup-config.py

    # Upload the startup config
    # PUT https://<server>:<port>/api/1.1/templates/library/cisco-example-startup-config
    # via curl:
    curl -X PUT -H "Content-type: text/raw" -d @<startup config path> https://<server>:<port>/api/1.1/templates/library/cisco-example-startup-config

*Note the ejs template variable used in the above python script* (:code:`<%=startupConfigUri%>`). 
*This is used by the RackHD server to render its own API address dynamically, and must be specified within the workflow options.*

Now the custom workflow can be updated again with a task to deploy the startup config:

.. code-block:: JSON


    {
         "friendlyName": "My Custom Cisco Switch Workflow",
         "injectableName": "Graph.Switch.CiscoNexus3000.MyCustomWorkflow",
         "options": {},
         "tasks": [
            {
                "label": "deploy-startup-config",
                "taskDefinition": {
                    "friendlyName": "Deploy Cisco Startup Config",
                    "injectableName": "Task.Inline.Switch.Cisco.DeployStartupConfig",
                    "implementsTask": "Task.Base.Linux.Commands",
                    "options": {
                        "startupConfig": "cisco-example-startup-config",
                        "startupConfigUri": "{{ api.base }}/templates/{{ options.startupConfig }}",
                        "commands": [
                            {
                                "downloadUrl": "/api/1.1/templates/deploy-cisco-startup-config.py
                            }
                        ]
                    },
                    "properties": {}
                },
            },
            {
                "label": "catalog-switch-config",
                "taskDefinition": {
                    "friendlyName": "Catalog Cisco Snmp Group",
                    "injectableName": "Task.Inline.Catalog.Switch.Cisco.SnmpGroup",
                    "implementsTask": "Task.Base.Linux.Commands",
                    "options": {
                        "commands": [
                            {
                                "downloadUrl": "/api/1.1/templates/cisco-catalog-snmp-example.py",
                                "catalog": { "format": "json", "source": "snmp-group" }
                            }
                        ]
                    },
                    "properties": {}
                },
            }
        ]
    }

Note that the :code:`startupConfigUri` template variable is set in the options for the task definition, so that
the deploy script can download the startup config from the right location.

In order to make this workflow more re-usable for a variety of switches, 
the startupConfig option can be specified as an override
in the SKU definition using the :code:`discoveryGraphOptions` field, for example:

.. code-block:: JSON

    {
        "name": "Cisco Nexus 3000 Switch - 24 port",
        "rules": [
            {
                "path": "version.chassis_id",
                "regex": "Nexus\\s\\d\\d\\d\\d\\w?\\sChassis"
            },
            {
                "path": "version.module_id",
                "equals": "24x10GT.*"
            }
        ],
        "discoveryGraphName": "Graph.Switch.CiscoNexus3000.MyCustomWorkflow",
        "discoveryGraphOptions": {
                "deploy-startup-config": {
                        "startupConfig": "example-cisco-startup-config-24-port"
                }
        }
    }
