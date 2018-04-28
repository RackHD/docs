Naming Conventions
=============================

.. contents:: Table of Contents

Workflows
-----------------------------

We use the following conventions when creating workflow-related JSON documents:

**Tasks**

For task definitions, the only convention is for values in the "injectableName" field.
We tend to prefix all names with "Task." and then add some categorization to classify what
functionality the task adds.

Examples::

    Task.Os.Install.CentOS
    Task.Os.Install.Ubuntu
    Task.Obm.Node.PowerOff
    Task.Obm.Node.PowerOn


**Graphs**

For graph definitions, conventions are pretty much the same as tasks, except "injectableName" is
prefixed by "Graph.".

Examples::

    Graph.Arista.Zerotouch.vEOS
    Graph.Arista.Zerotouch.EOS


Microkernel docker image
-----------------------------

**Image Names**

We tend to prefix docker images with *micro_*  along with some information about which
RancherOS the docker image was built off and information about what is contained
within the docker image. Images are suffixed with *docker.tar.xz* because they are xzed
tar archives contain docker image.

Examples::

    micro_1.2.0_flashupdt.docker.tar.xz
    micro_1.2.0_brocade.docker.tar.xz
    micro_1.2.0_all_binaries.docker.tar.xz


**Image Files**

When adding scripts and binaries to docker image, we typically put them in /opt within subdirectories
based on vendor.

Examples::

/opt/MegaRAID/MegaCli/MegaCli64
/opt/MegaRAID/StorCli/storcli64
/opt/mpt/mpt3fusion/sas3flash


If you want to add binaries or scripts and reference them by name rather than their absolute paths,
then add them to /usr/local/bin or any other directory in the default PATH for bash.

**File Paths**

Our HTTP server will serve docker images from /opt/monorail/static/http. It is recommended that you
create subdirectories within this directory for further organization.

Examples::

/opt/monorail/static/http/teamA/intel_flashing/micro_1.2.0_flashupdt.docker.tar.xz
/opt/monorail/static/http/teamA/generic/micro_1.2.0_all_binaries.docker.tar.xz


These file paths can then be referenced in workflows starting from the base path
of /opt/monorail/static/http, so the above paths are referenced for download as::

    teamA/intel_flashing/micro_1.2.0_flashupdt.docker.tar.xz
    teamA/generic/micro_1.2.0_all_binaries.docker.tar.xz
