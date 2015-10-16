## Walkthrough: Creating workflows with custom overlays and commands

This example will demonstrate how to create a workflow that runs shell commands or scripts packaged into in a custom overlay image.

##### Prerequisites:

- All monorail server processes up and running
- The custom overlay image you will use added somewhere within the /opt/monorail/static/http directory. For creating overlay images, see the guide to [creating custom overlays](https://hwstashprd01.isus.emc.com:8443/projects/ONRACK/repos/on-integration-test/browse/docs/creating-overlays.md) first if you have not done so already.


##### Terminology:

*Workflow*:  A pre-defined series of operations to run against a server node, specified by a graph definition  
*Task*: A JSON definition for a specific step in a workflow  
*Graph*: A JSON definition that specifies a series of tasks, with optional configurations, that constitutes a workflow  
*Overlay/Overlay Image*:  A minimal filesystem image that is overlaid on top of a base filesystem image (it gets mounted as a filesystem of type: overlayfs)

#### Step 1: Creating the workflow tasks

**Creating a task for custom commands**

Let's say we want to run a workflow that runs two custom scripts against a node. We've packaged these scripts into our overlay, stored in /opt/scripts/myscript.sh and would like to be able to use it.  In order to do that, we can define a workflow task with the following JSON and POST it to the server:

```
PUT /api/1.1/workflows/tasks
Content-Type: application/json
{
    "friendlyName" : "My Custom Scripts",
    "injectableName" : "Task.Linux.Commands.MyCustomScripts",
    "implementsTask" : "Task.Base.Linux.Commands",
    "options" : {
        "commands" : [ 
            {
                "command" : "/opt/scripts/myscript1.sh" 
            },
            {
                "command" : "/opt/scripts/myscript2.sh" 
            }
        ]
    },
    "properties" : { }
}
```

*For a more in-depth overview of what each of these fields mean, see [tasks](https://hwstashprd01.isus.emc.com:8443/projects/ONRACK/repos/on-integration-test/browse/docs/tasks.md)*

This task implements a pre-existing base task definition for running custom shell commands ("Task.Base.Linux.Commands"). What this task will do is run the two commands: `/opt/scripts/myscript1.sh` and `/opt/scripts/myscript2.sh` in order, succeeding if both commands have an exit code of 0.

The above JSON is a definition of a new workflow task that implements the "Task.Base.Linux.Commands" task. The "friendlyName" field is for convenient human-recognizable naming. The "injectableName" field is the string value you will to reference this task via the API or other workflow definitions. They can be any string, but must be unique system-wide. For each command you want to run, add a command object to the options.commands field as demonstrated. Any valid shell command is acceptable, and any number of command objects can be added (see [create-linux-commands](https://hwstashprd01.isus.emc.com:8443/projects/ONRACK/repos/on-integration-test/browse/docs/examples/create-linux-commands-graph.md) for more information about customizing options around these objects).

**Creating a task to boot a custom overlay**

In order to run the above task, we first need to bootstrap the node into the custom overlay image we have made that contains our scripts. To do this, we will leverage the pre-existing base task definition for bootstrapping a node. We can define this task with the following JSON:

```
PUT /api/1.1/workflows/tasks
Content-Type: application/json
{
    "friendlyName": "Bootstrap Linux overlayfs_custom_scripts",
    "injectableName": "Task.Linux.Bootstrap.overlayfs_custom_scripts",
    "implementsTask": "Task.Base.Linux.Bootstrap",
    "options": {
        "kernelversion": "vmlinuz-3.13.0-32-generic",
        "kernel": "common/vmlinuz-3.13.0-32-generic",
        "initrd": "common/initrd.img-3.13.0-32-generic",
        "basefs": "common/base.trusty.3.13.0-32.squashfs.img",
        "overlayfs": "extensions/myoverlays/overlayfs_custom_scripts.cpio.gz",
        "profile": "linux.ipxe"
    },
    "properties": { }
}
```

*This task assumes we created our overlay off the base image `base.trusty.3.13.0-32.squashfs.img` and the `3.13.0-32` linux kernel.*

In the overlayfs field above, we specify the path to our overlay image. The base path for serving image files on the server is /opt/monorail/static/http, and path strings should start from there. In this example, the file specified above would be located at `/opt/monorail/static/http/extensions/myoverlays/overlayfs_custom_scripts.cpio.gz`.

Again, the friendlyName and injectableName can be whatever you want as long as they are unique, with the injectableName being what we will use later to refer to the task in the API and in other workflow definitions.


#### Step 2: Creating the workflow graph

The workflow graph is the definition of which tasks should be run and in what order. In this case, we want to reboot a node, PXE boot it into our microkernel (which will run in RAM), run our scripts, then reboot the node again. We can combine pre-defined system tasks with the ones we have created with the following JSON:

```
PUT /api/1.1/workflows
Content-Type: application/json
{
    "friendlyName": "Custom Overlay Scripts",
    "injectableName": "Graph.Custom.OverlayScripts",
    "tasks": [
        {
            "label": "set-boot-pxe",
            "taskName": "Task.Obm.Node.PxeBoot"
        },
        {
            "label": "reboot-start",
            "taskName": "Task.Obm.Node.Reboot",
            "waitOn": {
                "set-boot-pxe": "finished"
            }
        },
        {
            "label": "bootstrap-custom-overlay",
            "taskName": "Task.Linux.Bootstrap.overlayfs_custom_scripts",
            "waitOn": {
                "reboot-start": "succeeded"
            }
        },
        {
            "label": "run-custom-scripts",
            "taskName": "Task.Linux.Commands.MyCustomScripts",
            "waitOn": {
                "bootstrap-custom-overlay": "succeeded"
            }
        },
        {
            "label": "reboot-end",
            "taskName": "Task.Obm.Node.Reboot",
            "waitOn": {
                "run-custom-scripts": "finished"
            }
        }
    ]
}
```

*For more information on graph definitions, see [graphs](https://hwstashprd01.isus.emc.com:8443/projects/ONRACK/repos/on-integration-test/browse/docs/graphs.md)*

The third and fourth task objects in this definition reference the custom tasks created above via their taskName fields, which map to the injectableName values of the task definitions.
		
#### Step 3: Running the workflow

To run the workflow against a node, first retrieve the nodeId, and then submit the following API request:

```
POST /api/1.1/nodes/<nodeId>/workflows
Content-Type: application/json
{
    "name": "Graph.Custom.OverlayScripts"
}
```

The "name" field value should equal the "injectableName" string of the graph to be run. In these examples, the graph JSON definition has an "injectableName" field that equals "Graph.Custom.OverlayScripts".

After submitting the API request, the server should respond with JSON data representing the serialized state of the active graph being run against the target node. You can also tail the `/var/log/upstart/on-taskgraph.log` file for warnings and errors related to the running of the graph.

