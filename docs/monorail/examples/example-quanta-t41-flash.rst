## Walkthrough: Running the example Quanta T41 flashing workflow ##
<br>
The monorail system includes a sample, yet functional, flashing workflow that
can be used to flash the MegaRAID, BIOS, and BMC on a Quanta T41 server.

#### Step 1: Update/verify the on-taskgraph config ####

In lieu of a centralized configuration file, the on-taskgraph config must contain some duplicate values from the http config. Edit `/var/renasar/on-taskgraph/config.json` and make sure the `"server"`, `"httpPort"` and `"httpsPort"` values match those in `/var/renasar/on-http/config.json`.

#### Step 2: Upload firmware files ####

All flashing workflows require firmware files to be flashed onto a target system.
These files are stored via the Monorail files backend 
    (stored on disk, in conjunction with a simple database file for metadata and
     a management API). You can upload these using the files API:

```
# Using curl to send a PUT request
curl -T <path to firmware file> <server>/api/1.1/files/<filename>
```

The files will be stored in the database, and accessible via the <filename> field
of the above API request.

To ensure the uploaded file matches the checksum of the source file, compare the
checksums of the source and uploaded files:

```
curl <server>/api/1.1/files/md5/<filename>/latest
md5sum <path to firmware file>
```

After uploading the files, make sure to note the filenames used, as these are
options required by the flashing workflow.


#### Step 3: Run the workflow against a node ####

Before you run the workflow, ensure you have a recent overlay package containing
the quanta t41 flashing overlay. The on-static-common package must be version (1.0-7)
or greater.
 
The example Quanta T41 flashing workflow is stored in the system under the name
'Graph.Flash.Quanta'. To create/run this workflow against a node, send the following
request:

```
POST
<server>/api/1.1/nodes/<nodeId>/workflows
Content-type: application/json
{
    "name": "Graph.Flash.Quanta",
    "options": {
        "download-megaraid-firmware": {
            "file": "<megaraid firmware filename>"
        },
        "download-bios-firmware": {
            "file": "<bios firmware filename>"
        },
        "download-bmc-firmware": {
            "file": "<bmc firmware filename>"
        },
        "flash-megaraid": {
            "file": "<megaraid firmware filename>"
        },
        "flash-bios": {
            "file": "<bios firmware filename>"
        },
        "flash-bmc": {
            "file": "<bmc firmware filename>
        }
    }
}
```
