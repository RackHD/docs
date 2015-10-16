
Ubuntu installation/upgrade instructions
========================================


Upgrading the code
------------------

- Log into the server (nuc, kylin, etc)

get all current patches and remove non-essential packages:

    sudo apt-get update
    sudo apt-get dist-upgrade
    sudo apt-get autoremove

install required packages:

    sudo apt-get install rabbitmq-server
    sudo apt-get install mongodb
    sudo apt-get install snmp
    sudo apt-get install ipmitool
    sudo apt-get install nodejs nodejs-legacy npm


install supplemental packages:

    sudo apt-get install python-pywbem
    sudo apt-get install ansible
    sudo apt-get install apt-mirror
    sudo apt-get install amtterm

create a apt-sources list file : `/etc/apt/sources.list.d/renasar.list`

    deb [arch=amd64] http://54.191.244.96/ trusty non-free

Update the package directories

    sudo apt-get update

Install the renasar packages:

    sudo apt-get install renasar-dhcp renasar-http renasar-taskgraph
    sudo apt-get install renasar-tftp renasar-syslog renasar-cli
    sudo apt-get install renasar-fog-ui

set the links to the mirrors:

    sudo ln -s /var/mirrors/esxi /var/renasar/renasar-http/static/http/vmware
    sudo ln -s /var/mirrors/esxi /var/renasar/renasar-tftp/static/tftp/vmware
    sudo ln -s /var/mirrors/ubuntu_trusty /var/renasar/renasar-http/static/http/ubuntu_trusty
    sudo ln -s /var/mirrors/ubuntu/14.04/mirror/mirror.pnl.gov/ubuntu/ /var/renasar/renasar-http/static/http/trusty
    sudo ln -s /var/mirrors/centos /var/renasar/renasar-http/static/http/centos
    sudo ln -s /var/mirrors/suse /var/renasar/renasar-http/static/http/suse
    
Install supplementary files:

    sudo apt-get install renasar-static-common renasar-mibs
    sudo apt-get install renasar-static-coreos
    sudo apt-get install renasar-static-emc
    sudo apt-get install renasar-static-arista
    sudo apt-get install renasar-static-intel
    sudo apt-get install renasar-static-vmware
    sudo apt-get install renasar-static-xen

retrieve the installer/distribution media:

    mkdir -p /var/mirrors
    cd /var/mirrors
    curl http://xfer.renasar.com/emc/ubuntu_trusty.tgz | tar xvzf -
    curl http://xfer.renasar.com/emc/esxi.tgz | tar xvzf -

Add ansible playbook for post-os install
----------------------------------------

    cd /var/renasar
    curl http://xfer.renasar.com/emc/ansible_test.tar.gz | tar xvzf -

Configure the applications to start
-----------------------------------

All of the renasar programs are set to respect and verify the existance of a
default configuration file. If this file doesn't exist, the services will
fail to start with a relevant message in the upstart log for that process.

To enable all the programs, create the files for each process after appropriate configuration has been put into place:

    touch /etc/default/renasar-http
    touch /etc/default/renasar-dhcp
    touch /etc/default/renasar-syslog
    touch /etc/default/renasar-tftp
    touch /etc/default/renasar-taskgraph

Upgrading the code
------------------

The upgrade from version 1.1-19 to later versions includes underlying changes that redefine the
exchanges used in RabbitMQ. Those aren't automatically reset at this time, and if the old
exchanges exist when the new code runs, errors will appear in all relevant programs. To reset
the exchanges, use the following commands:

    rabbitmqctl stop_app
    rabbitmqctl force_reset
    rabbitmqctl start_app

You do not need to restart the applications after this has occured - the applications will
automatically attempt to reconnect and establish the correct exchanges.


Making the mirrors
------------------

#Centos - 6.5

    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" \
    --exclude "i386" rsync://centos.eecs.wsu.edu/centos/6.5/ /var/mirrors/centos/6.5

#Centos - 7.0

    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" \
    --exclude "i386" rsync://centos.eecs.wsu.edu/centos/7/ /var/mirrors/centos/7

#OpenSuse - 12.3

    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/distribution/12.3/ /var/mirrors/suse/distribution/12.3
    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/12.3 /var/mirrors/suse/update
    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/12.3-non-oss /var/mirrors/suse/update

#OpenSuse - 13.1

    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/distribution/13.1/ /var/mirrors/suse/distribution/13.1
    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/13.1 /var/mirrors/suse/update
    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/13.1-non-oss /var/mirrors/suse/update

#OpenSuse - 13.2

    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/distribution/13.2/ /var/mirrors/suse/distribution/13.2
    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/13.2 /var/mirrors/suse/update
    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/13.2-non-oss /var/mirrors/suse/update


for the Ubuntu repo, you need some additional installation. The mirrors are easily made on Ubuntu, but not so easily replicated on other OS. On any recent distribution of Ubuntu:

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

How to wipe out the database to restart everything
--------------------------------------------------

    sudo service renasar-http stop
    sudo service renasar-dhcp stop
    sudo service renasar-syslog stop
    sudo service renasar-taskgraph stop
    sudo service renasar-tftp stop

    mongo
        use pxe
        db.dropDatabase()
        ^D

    sudo service renasar-http start
    sudo service renasar-dhcp start
    sudo service renasar-syslog start
    sudo service renasar-taskgraph start
    sudo service renasar-tftp start

Post-install - what can you do and how to do it
-----------------------------------------------

To get a list of workflows in the library, run

    curl <server>/api/1.1/workflows/library

Sample output:

    {"friendlyName":"Install Ubuntu","injectableName":"Graph.InstallUbuntu","tasks":[{"label":"set-boot-pxe","taskName":"Task.Obm.Node.PxeBoot","ignoreFailure":true},{"label":"reboot","taskName":"Task.Obm.Node.Reboot","waitOn":{"set-boot-pxe":"finished"}},{"label":"install-ubuntu","taskName":"Task.Os.Install.Ubuntu","waitOn":{"reboot":"succeeded"}}]}

To create workflows, reference them by the “injectableName” property, i.e.

    curl -X POST localhost/api/1.1/nodes/<identifier>/workflows?name=Graph.InstallUbuntu

There is now basic support for a custom type workflow that takes a list of shell commands and runs them. In order to utilize this feature, new workflows and tasks (units of work) must be registered in the system. Note that currently, these are only stored in memory, so they must be recreated whenever the renasar-taskgraph process is restarted. To create a basic workflow that runs user specified shell commands, with user specified images, the following steps are necessary:

Define a custom workflow task with the images specified to be used:

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
