General Naming Conventions
------------------------------------

Workflow conventions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We use the following conventions when creating workflow-related JSON documents:

**Tasks**

For task definitions, the only convention is for values in the "injectableName" field. We tend to prefix all names with "Task.", and then add some categorization to classify what functionality the task adds. Some examples:

- "Task.Os.Install.CentOS"
- "Task.Os.Install.Ubuntu"
- "Task.Obm.Node.PowerOff"
- "Task.Obm.Node.PowerOn"

**Graphs**

For graph definitions, conventions are pretty much the same as tasks, except "injectableName" is prefixed by "Graph.", some examples:

- "Graph.Arista.Zerotouch.vEOS"
- "Graph.Arista.Zerotouch.EOS"

Overlay conventions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Overlay names**

We tend to prefix overlays with "overlayfs_", along with some information about which kernel/base image the overlay was built off, and information about what is contained within the overlay. Overlays are suffixed with ".cpio.gz" because they are gzipped cpio archives. Examples:

- overlayfs_3.13.0-32_flashupdt.cpio.gz
- overlayfs_3.13.0-32_brocade.cpio.gz
- overlayfs_3.13.0-32_all_binaries.cpio.gz

**Overlay Files**

When adding scripts and binaries to an overlay, we typically put them in /opt within subdirectories based on vendor, e.g.

- /opt/MegaRAID/MegaCli/MegaCli64
- /opt/MegaRAID/StorCli/storcli64
- /opt/mpt/mpt3fusion/sas3flash

If you want to add binaries or scripts and reference them by name rather than their absolute paths, then add them to /usr/local/bin or any other directory in the default PATH for bash.

**File paths**

Our HTTP server will serve overlay files from /opt/monorail/static/http. It is recommended that you create subdirectories within this directory for further organization, e.g.:

- /opt/monorail/static/http/teamA/intel_flashing/overlayfs_3.13.0-32_flashupdt.cpio.gz
- /opt/monorail/static/http/teamA/generic/overlayfs_3.13.0-32_all_binaries.cpio.gz

These file paths can then be referenced in workflows starting from the base path of /opt/monorail/static/http, so the above paths are referenced for download as:

- teamA/intel_flashing/overlayfs_3.13.0-32_flashupdt.cpio.gz
- teamA/generic/overlayfs_3.13.0-32_all_binaries.cpio.gz
