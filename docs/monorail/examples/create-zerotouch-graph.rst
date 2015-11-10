This section provides instructions for creating a custom zerotouch graph for Arista machines,
including defining a custom EOS image, custom startup-config, and custom zerotouch script.


Below is an example zerotouch graph for booting a vEOS (virtual arista) machine. It uses
an inline task definition (as opposed to creating a new task definition as a separate step):



.. code-block:: guess

 {
    friendlyName: 'Zerotouch vEOS Graph',
    injectableName: 'Graph.Arista.Zerotouch.vEOS',
    tasks: [
        {
            label: 'zerotouch-veos',
            taskDefinition: {
                friendlyName: 'Arista Zerotouch vEOS',
                injectableName: 'Task.Inline.Arista.Zerotouch.vEOS',
                implementsTask: 'Task.Base.Arista.Zerotouch',
                options: {
                    profile: 'zerotouch-configure.zt',
                    bootConfig: 'arista-boot-config',
                    startupConfig: 'arista-startup-config',
                    eosImage: 'common/zerotouch-vEOS.swi',
                    bootfile: 'zerotouch-vEOS.swi',
                    hostname: 'MonorailVEOS'
                },
                properties: {
                    os: {
                        switch: {
                            type: 'vEOS',
                            virtual: true
                        }
                    }
                }
            }
        }
    ]
 }



To customize this graph, change the following fields:


.. list-table::
   :widths: 10 80
   :header-rows: 1

   * - Field
     - Description
   * - friendlyName
     - A unique friendly name for the graph.
   * - injectableName
     - A unique injectable name for the graph.
   * - task/friendlyName
     - A unique friendlyName for the task.
   * - task/injectableName
     - A unique injectableName for the task.
   * - profile
     - The default profile is sufficient for most cases. See the Zerotouch Profile section for more information.
   * - bootConfig
     - The default bootConfig is sufficient for most cases. See the Zerotouch Profile section for more information.
   * - startupConfig
     - Specify the name of the custom startup config. See the Adding Zerotouch Templates section for more information.
   * - eosImage
     - Specify the name of the EOS image. See the Adding EOS Images section for more information.
   * - bootfile
     - In most cases, specify the eosImage name.
   * - hostname
     - An value rendered into the default arista-startup-config template. Depending on the template, this may be optional.
   * - properties
     - A object containing any tags/metadata that you wish to add.


**Adding Zerotouch Templates**

**Creation**

Templates are defined using `ejs`_ syntax. To define template
variables, use this syntax:

.. _ejs: https://github.com/tj/ejs

.. code-block:: ejs

   <%=variableName%>


In order to provide a value for this variable when the template is rendered, add the variable
name as a key in the options object of the custom zerotouch task definition:

.. code-block:: guess


 taskDefinition: {
    <other values>
    options: {
        hostname: 'CustomHostName'
    }
 }


The above code renders the following startup config as shown here:

.. code-block:: guess

 Unrendered:
 !
 hostname <%=hostname%>
 !

 Rendered:
 !
 hostname CustomHostName
 !


**Uploading**

To upload a template, use the templates API:

.. code-block:: BatchLexer

 PUT /api/1.1/templates/library/<filename>
 Content-Type: application/octet-stream
 ---
 curl -X PUT \
     -H 'Content-Type: application/octet-stream' \
     -d "<startup config template>" \
    <server>/api/1.1/templates/library/<filename>



**Adding EOS Images**

Move any EOS images you would like to use into <on-http directory>/static/http/common/.

In the task options, reference the EOS image name along with the common
directory, e.g. eosImage: common/<eosImageName>.

**Zerotouch Profile**

A zerotouch profile is a script template that is executed by the switch during zerotouch.
A basic profile looks like the following:


.. code-block:: BatchLexer

 #!/usr/bin/Cli -p2
 enable
 copy http://<%=server%>:<%=port%>/api/1.1/templates/<%=startupConfig%> flash:startup-config
 copy http://<%=server%>:<%=port%>/api/1.1/templates/<%=bootConfig%> flash:boot-config
 copy http://<%=server%>:<%=port%>/common/<%=eosImage%> flash:
 exit


Adding #!/usr/bin/Cli -p2 tells the script to be executed by the Arista's CLI parser.
Using #!/bin/bash for more control is also an option. If using bash for zerotouch config, any
config and imaging files should go into the /mnt/flash/ directory.

**Zerotouch Boot Config**

The zerotouch boot config is a very simple config that specifies which EOS image file to boot.
This should almost always match the EOS image filename you have provided, e.g.:


.. code-block:: BatchLexer

 SWI=flash:/<%=bootfile%>
