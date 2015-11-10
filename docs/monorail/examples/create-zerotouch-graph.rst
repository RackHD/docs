Instructions for creating a custom zerotouch graph for Arista machines,
including defining a custom EOS image, custom startup-config, and custom zerotouch script.


Below is an example zerotouch graph for booting a vEOS (virtual arista) machine, utilizing
an inline task definition (as opposed to creating a new task definition as a separate step):



::

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

- friendlyName - choose a new unique friendly name for your graph
- injectableName - choose a new unique name for your graph
- tasks[0]
    - friendlyName - choose a new unique friendly name for your task
    - injectableName - choose a new unique name for your task
    - options
        - profile - the default profile should be sufficient for most cases, see
                    the Zerotouch Profile] section for more information
        - bootConfig - the default bootConfig should be sufficient for most cases,
                    see the Zerotouch Boot Config section for more information
        - startupConfig - change this to the name of your custom startup config. See
                    the Adding Zerotouch Templates] section.
        - eosImage - change this to the name of your EOS image. See the Adding EOS Images
                    section.
        - bootfile - in almost all cases this should be the same as your eosImage name
        - hostname - an option value rendered into the default arista-startup-config template.
                    Optional depending on the template.
    - properties - a object containing any tags/metadata you wish to add


### Adding Zerotouch Templates

#### Creation

Templates are defined using [ejs](https://github.com/tj/ejs) syntax. To define template
variables, use this syntax:

::

<%=variableName%>


In order to provide a value for this variable when the template is rendered, add the variable
name as a key in the options object of the custom zerotouch task definition, e.g.

::


taskDefinition: {
    <other values>
    options: {
        hostname: 'CustomHostName'
    }
}


will render the following startup config as:

::

Unrendered:

!
hostname <%=hostname%>
!

Rendered:
!
hostname CustomHostName
!
```

**Uploading**

To upload a template, use the templates API:

::

PUT /api/1.1/templates/library/<filename>
Content-Type: application/octet-stream
---
curl -X PUT \
     -H 'Content-Type: application/octet-stream' \
     -d "<startup config template>" \
    <server>/api/1.1/templates/library/<filename>



**Adding EOS Images**

Move any EOS images you would like to use into <on-http directory>/static/http/common/

In your task options, reference the EOS image name along with the common
directory, e.g. eosImage: common/<eosImageName>

**Zerotouch Profile**

A zerotouch profile is a script template that is executed by the switch during zerotouch.
A basic profile looks like:


::

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


::

SWI=flash:/<%=bootfile%>
