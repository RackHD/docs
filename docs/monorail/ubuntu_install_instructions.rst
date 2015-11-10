Ubuntu Installation/Upgrade
----------------------------------------------


Upgrading the code
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Log into the Server**

**Get All Current Patches and Remove Non-essential Packages:**


 .. code::

    sudo apt-get update
    sudo apt-get dist-upgrade
    sudo apt-get autoremove

**Install Required Packages:**

 .. code::

    sudo apt-get install rabbitmq-server
    sudo apt-get install mongodb
    sudo apt-get install snmp
    sudo apt-get install ipmitool
    sudo apt-get install nodejs nodejs-legacy npm

 .. code::

**Install Supplemental Packages:**

 .. code::

    sudo apt-get install ansible
    sudo apt-get install apt-mirror
    sudo apt-get install amtterm

**Create an apt-sources List FDile:** `/etc/apt/sources.list.d/monorail.list`

 .. code::

    deb [arch=amd64] http://TBD/ trusty non-free

.. note::

   RackHD is in the process of setting up debian packages in a public repository
   with a service like bintray, but it's not yet complete. In the meantime, it's
   possible to build the packages directly (yourself) or we recommend you install
   from source.

**Update the Package Directories:**

 .. code::

    sudo apt-get update

**Install the mMnorail Packages:**

 .. code::

    sudo apt-get install on-dhcp-proxy on-http on-taskgraph
    sudo apt-get install on-tftp on-syslog
    sudo apt-get install on-web-ui

**Set the Links to the Mirrors:**

  .. code::

    sudo ln -s /var/mirrors/esxi <on-http directory>/static/http/vmware
    sudo ln -s /var/mirrors/esxi <on-tftp directory>/static/tftp/vmware
    sudo ln -s /var/mirrors/ubuntu_trusty <on-http directory>/static/http/ubuntu_trusty
    sudo ln -s /var/mirrors/ubuntu/14.04/mirror/mirror.pnl.gov/ubuntu/ <on-http directory>/static/http/trusty
    sudo ln -s /var/mirrors/centos <on-http directory>/static/http/centos
    sudo ln -s /var/mirrors/suse <on-http directory>/static/http/suse

**Install Supplementary Files:**

  .. code::

    sudo apt-get install on-static-common

  TODO: add more instructions here

**Retrieve the Installer/Distribution Media**

  TODO: add instructions here

Add Ansible Playbook for Post-OS Install:**

  TODO: update instructions here

**Configure the Applications to Start**

All of the monorail programs are set to respect and verify the existence of a
default configuration file (/opt/onrack/etc/monorail.json). If this file doesn't exist, the services will
fail to start with a relevant message in the upstart log for that process.

Making the Mirrors
~~~~~~~~~~~~~~~~~~~~~~~~~~

**Centos 6.5**

  .. code::

    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" \
    --exclude "i386" rsync://centos.eecs.wsu.edu/centos/6.5/ /var/mirrors/centos/6.5

**Centos 7.0**

  .. code::

    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" \
    --exclude "i386" rsync://centos.eecs.wsu.edu/centos/7/ /var/mirrors/centos/7

**OpenSuse 12.3**

  .. code::

    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/distribution/12.3/ /var/mirrors/suse/distribution/12.3
    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/12.3 /var/mirrors/suse/update
    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/12.3-non-oss /var/mirrors/suse/update

**OpenSuse 13.1**

  .. code::

    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/distribution/13.1/ /var/mirrors/suse/distribution/13.1
    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/13.1 /var/mirrors/suse/update
    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/13.1-non-oss /var/mirrors/suse/update

**OpenSuse 13.2**

  .. code::

    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/distribution/13.2/ /var/mirrors/suse/distribution/13.2
    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/13.2 /var/mirrors/suse/update
    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/13.2-non-oss /var/mirrors/suse/update


For the Ubuntu repo, you need some additional installation. The mirrors are easily made on Ubuntu, but not so easily replicated on other OS. On any recent distribution of Ubuntu:

  .. code::

	# make the mirror directory (can sometimes hit a permissions issue)
	sudo mkdir -p /var/mirrors/ubuntu/14.04/mirror
	# create a file in /etc/apt/mirror.list (config below)
	sudo vi /etc/apt/mirror.list
	# run the mirror
	sudo apt-mirror


    ############# config ##################
    #
    set base_path    /var/mirrors/ubuntu/14.04
    #
    # set mirror_path  $base_path/mirror
    # set skel_path    $base_path/skel
    # set var_path     $base_path/var
    # set cleanscript $var_path/clean.sh
    # set defaultarch  <running host architecture>
    # set postmirror_script $var_path/postmirror.sh
    # set run_postmirror 0
    set nthreads     20
    set _tilde 0
    #
    ############# end config ##############

    deb-amd64 http://mirror.pnl.gov/ubuntu trusty main
    deb-amd64 http://mirror.pnl.gov/ubuntu trusty-updates main
    deb-amd64 http://mirror.pnl.gov/ubuntu trusty-security main
    clean http://mirror.pnl.gov/ubuntu

    #end of file
    ###################

How to Erase the Database to Restart Everything
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

  .. code::

    sudo service on-http stop
    sudo service on-dhcp-proxy stop
    sudo service on-syslog stop
    sudo service on-taskgraph stop
    sudo service on-tftp stop

    mongo pxe
        db.dropDatabase()
        ^D

    sudo service on-http start
    sudo service on-dhcp-proxy start
    sudo service on-syslog start
    sudo service on-taskgraph start
    sudo service on-tftp start

Post-Installation Procedures
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To get a list of workflows in the library, run

  .. code::

    curl <server>/api/1.1/workflows/library

Sample output:

.. code-block:: rest

    {"friendlyName":"Install Ubuntu","injectableName":"Graph.InstallUbuntu",
     "tasks":[{"label":"set-boot-pxe","taskName":"Task.Obm.Node.PxeBoot",
     "ignoreFailure":true},{"label":"reboot","taskName":"Task.Obm.Node.Reboot",
     "waitOn":{"set-boot-pxe":"finished"}}, {"label":"install-ubuntu",
     "taskName":"Task.Os.Install.Ubuntu","waitOn":
     {"reboot":"succeeded"}}]     }

To create workflows, reference them by the “injectableName” property, i.e.

  .. code::

    curl -X POST localhost/api/1.1/nodes/<identifier>/workflows?name=Graph.InstallUbuntu

There is now basic support for a custom type workflow that takes a list of shell commands and runs them. In order to utilize this feature, new workflows and tasks (units of work) must be registered in the system. Note that currently, these are only stored in memory, so they must be recreated whenever the on-taskgraph process is restarted. To create a basic workflow that runs user specified shell commands, with user specified images, the following steps are necessary:

Define a custom workflow task with the images specified to be used:

  .. code::

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

Now define a custom workflow task that contains the commands to be run. The below format for the command objects must be followed (command, format, and source keys). Supported format values are ‘raw’, ‘json’, and ‘xml’. The ‘source’ key is the source value in the catalogs entry in the database.

  .. code::

    PUT <server>/api/1.1/workflows/tasks
    Content-Type: application/json
    {
        "friendlyName": "Shell commands user",
        "injectableName": "Task.Linux.Commands.User",
        "implementsTask": "Task.Base.Linux.Commands",
        "options": {
            "commands": [
                { "command": "sudo ls /var", "format": "raw", "source": "ls var" },
                { "command": "sudo lshw -json", "format": "json", "source": "lshw user" }
            ]
        },
        "properties": {"type": "userCreated" }
    }

Now define a custom workflow that combines these tasks and runs them in a sequence. This one is set up to make OBM calls as well.

  .. code::

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

  .. code::
    curl -X POST localhost/api/1.1/nodes/<identifier>/workflows?name=Graph.ShellCommands.User
