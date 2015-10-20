### Linux Commands task

The Linux Commands task is a generic task that enables running of any shell commands against a node booted into a microkernel. These commands are specified in JSON objects within the options.commands array of the task definition. Optional parameters can be specified as well to enable cataloging of command output.

A very simple example task definition looks like:

```
{
    "friendlyName" : "Shell commands basic",
    "implementsTask" : "Task.Base.Linux.Commands",
    "injectableName" : "Task.Linux.Commands.BasicExample",
    "options" : {
        "commands" : [ 
            {
                "command" : "echo testing"
            },
            {
            	"command": "ls"
            }
        ]
    },
    "properties" : { }
}

```

There is an example task included in the monorail system under the name "Task.Linux.Commands" that makes use of all the different parameters the task can take:

```
{
    "friendlyName" : "Shell commands",
    "implementsTask" : "Task.Base.Linux.Commands",
    "injectableName" : "Task.Linux.Commands",
    "options" : {
        "commands" : [ 
            {
                "command" : "sudo ls /var",
                "catalog" : {
                    "format" : "raw",
                    "source" : "ls var"
                }
            }, 
            {
                "command" : "sudo lshw -json",
                "catalog" : {
                    "format" : "json",
                    "source" : "lshw user"
                }
            }, 
            {
                "command" : "test",
                "acceptedResponseCodes" : [ 
                    1
                ]
            }
        ]
    },
    "properties" : {
        "commands" : {}
    }
}
```

The task above will run three commands. It will catalog the output of the first two.

```
sudo ls /var
sudo lshw -json
test
```

**Specifying scripts or binaries to download and run**

Some use cases are too complex to be performed by embedding commands in json, and using a pre-defined file may be more convenient. You can define a file to download and run by specifying a "downloadUrl" field in addition to the "command" field.

```
"options": {
    "commands" : [
        {
            "command": "bash myscript.sh",
            "downloadUrl": "/api/1.1/templates/myscript.sh"
        }
    ]
}
```

This will cause the command runner script on the node to download the script from the specified route (server:port will be prepended) to the working directory, and execute it according to the specified command (e.g. `bash myscript.sh`). You must specify how to run the script correctly in the command field (e.g. `node myscript.js arg1 arg2`, `./myExecutable`).

A note on convention: binary files should be uploaded via the /api/1.1/files route, and script templates should be uploaded/downloaded via the /api/1.1/templates route.

**Defining script templates**

Scripts can mean simple shell scripts, python scripts, etc. 

In many cases, you may need access to variables in the script that can be rendered at runtime. Templates are defined using ejs syntax (variables in <%=variable%> tags), and any variables will be rendered based on the option values of task definition, for example, if a task is defined with these options:

```
"options": {
    "foo": "bar",
    "baz": "qux",
    "commands" : [
        {
            "command": "bash myscript.sh",
            "downloadUrl": "/api/1.1/templates/myscript.sh"
        }
    ]
}
```

Then the following script template...

```
echo <%=foo%>
echo <%=baz%>
```

...will get rendered as below when it gets run by a node

```
echo bar
echo qux
```

**Uploading script templates**

Script templates can be uploaded using the Monorail templates API:

```
PUT /api/1.1/templates/<filename>
Content-type: application/octet-stream
---
curl -X PUT -H "Content-Type: application/octet-stream" --data-binary @<script> <server>/api/1.1/templates/library/<scriptname>
```

**Uploading binary files**

Binary executables can be uploaded using the Monorail files API

```
PUT /api/1.1/files/<filename>
---
curl -T <binary> <server>/api/1.1/templates/library/<filename>
```

**Available options for command JSON objects**

The task definition above makes use of the different options available for parsing and handling of command output. Available options are detailed below:

| Name | Type | Required | Description |
|------|------|----------|-------------|
|command|string|command or script field required|the command to run|
|downloadUrl|string|API route suffix for file download|the script/file to download and run|
|catalog|object|no|an object specifying cataloging parameters if the command output should be cataloged
|acceptedResponseCodes|arrayOfString|no| non-zero exit codes from the command that should not be treated as failures

The catalog object in the above table may look like:

| Name | Type | Required | Description |
|------|------|----------|-------------|
|format|string|yes|what parser we should use for output. Available formats are 'raw', 'json' and 'xml'|
|source|string|no|what the 'source' key value in the database document should be. Will default to 'unknown' if not specified|

<hr>

### Creating a new graph with a custom shell commands task

In order to utilize this feature, new workflows and tasks (units of work) must be registered in the system. To create a basic workflow that runs user specified shell commands, with user specified images, do the following steps:

Define a custom workflow task with the images specified to be used (this is not necessary if you don't need to use a custom overlay):

    PUT <server>/api/1.1/workflows/tasks
    Content-Type: application/json
    {
        "friendlyName": "Bootstrap Linux Custom",
        "injectableName": "Task.Linux.Bootstrap.Custom",
        "implementsTask": "Task.Base.Linux.Bootstrap",
        "options": {
            "kernelversion": "vmlinuz-3.13.0-32-generic",
            "kernel": "common/vmlinuz-3.13.0-32-generic",
            "initrd": "common/initrd.img-3.13.0-32-generic",
            "basefs": "common/base.trusty.3.13.0-32.squashfs.img",
            "overlayfs": "common/overlayfs_all_files.cpio.gz",
            "profile": "linux.ipxe"
        },
        "properties": { }
    }

Now define a task that contains the commands to be run, adding or removing command objects below in the options.commands array.

    PUT <server>/api/1.1/workflows/tasks
    Content-Type: application/json
    {
        "friendlyName": "Shell commands user",
        "injectableName": "Task.Linux.Commands.User",
        "implementsTask": "Task.Base.Linux.Commands",
        "options": {
            "commands": [    <add command objects here>    ]
        },
        "properties": {"type": "userCreated" }
    }

The output from the first command (lshw) will be parsed as json and cataloged in the database under the "lshw user" source value. The output from the second command will only be logged, since format and source haven't been specified. The third command will normally fail, since \`test\` has an exit code of 1, but in this case we have specified that this is acceptable and not to fail. This feature is useful with certain binaries that have acceptable non-zero exit codes.


**Putting it all together**

Now define a custom workflow that combines these tasks and runs them in a sequence. This one is set up to make OBM calls as well.

    PUT <server>/api/1.1/workflows/
    Content-Type: application/json
    {
        "friendlyName": "Shell Commands User",
        "injectableName": "Graph.ShellCommands.User",
        "tasks": [
            {
                "label": "set-boot-pxe",
                "taskName": "Task.Obm.Node.PxeBoot",
                "ignoreFailure": true
            },
            {
                "label": "reboot-start",
                "taskName": "Task.Obm.Node.Reboot",
                "waitOn": {
                    "set-boot-pxe": "finished"
                }
            },
            {
                "label": "bootstrap-custom",
                "taskName": "Task.Linux.Bootstrap.Custom",
                "waitOn": {
                    "reboot-start": "succeeded"
                }
            },
            {
                "label": "shell-commands",
                "taskName": "Task.Linux.Commands.User",
                "waitOn": {
                    "bootstrap-custom": "succeeded"
                }
            },
            {
                "label": "reboot-end",
                "taskName": "Task.Obm.Node.Reboot",
                "waitOn": {
                    "shell-commands": "finished"
                }
            }
        ]
    }

With all of these data, the injectableName and friendlyName can be any string value, as long the references to injectableName are consistent across the three json documents.

After defining these custom workflows, you can then run one against a node by referencing the injectableName used in the json POSTed to /api/1.1/workflows/:

    curl -X POST localhost/api/1.1/nodes/<identifier>/workflows?name=Graph.ShellCommands.User
    
 Output from these commands will be logged by the taskgraph runner in /var/log/upstart/on-taskgraph.log.
