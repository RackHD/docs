Disk Secure Erase Workflow Support
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Secure Erase (SE) also known as a wipe is to destroy data on a disk so that data can't or is difficult to be retrieved. RackHD implements solution to do disk Secure Erase.

Disk Secure Erase Workflow API
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

An example of starting secure erase for disks:

.. code-block:: REST

    curl -X POST \
         -H 'Content-Type: application/json' \
         -d @params.json \
         <server>/api/current/nodes/<identifier>/workflows?name=Graph.Drive.SecureErase


An example of params.json for disk secure erase:

.. literalinclude:: samples/secure-erase.json
   :language: JSON

Use below command to check the workflow is active or inactive:

::
    curl <server>/api/current/nodes/<identifier>/workflows?active=true


Deprecated 1.1 API - Use below command to check the workflow is active or inactive:

.. code-block:: REST

    curl <server>/api/1.1/nodes/<identifier>/workflows/active

Use below command to stop the active workflow to cancel secure erase workflow:

::
    curl -X PUT \
    -H 'Content-Type: application/json' \
    -d '{"command": "cancel"}' \
    <server>/api/current/nodes/<id>/workflows/action


Deprecated 1.1 API - Use below command to stop the active workflow to cancel secure erase workflow:

.. code-block:: REST

    curl -X DELETE <server>/api/1.1/nodes/<identifier>/workflows/active

Disk Secure Erase Workflow Payload
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Parameters descriptions of secure erase workflow payload are listed below.

=================== ================ ============ ============================================
Parameters          Type              Flags       Description
=================== ================ ============ ============================================
eraseSettings       Array            **required** Contains secure erase option list, each list element is made up of "disks" and optional "tool" and "arg" parameters
disks               Array            **required** Contains disks to be erased, both devName or identifier from driveId catalog are eligible
tool                String           optional     Specify tool to be used for secure erase. Default it would be scrub.
arg                 String           optional     Specify secure erase arguments with specified tools
=================== ================ ============ ============================================

Supported Disk Secure Erase Tools
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

RackHD currently supports disk secure erase with four tools: scrub, hdparm, sg_sanitize, sg_format. If "tool" is not specified in payload, "scrub" is used as default. Below table includes description for different tools.

============= ===============================================================================================================
Tool          Description
============= ===============================================================================================================
scrub         Scrub iteratively writes patterns on files or disk devices to make retrieving the data more difficult. Scrub supports almost all drives including SATA, SAS, USB and so on.
hdparm        Hdparm can be used to issue ATA instruction of Secure Erase or enhanced secure erase to a disk. Hdparm works well with SATA drives, but it can brick a USB drive if it doesn't support SAT (SCSI-ATA Command Translation).
sg_sanitize   Sg_sanitize (from sg3-utils package) removes all user data from disk with SCSI SANITIZE command. Sanitize is more likely to be implemented on modern disks (including SSDs) than FORMAT UNIT's security initialization feature and in some cases much faster. However since it is relative new and optional, not all SCSI drives support SANITIZE command
sg_format     Sg_format (from sg3-utils package) formats, resizes or modifies protection information of a SCSI disk. The primary goal of a format is the configuration of the disk at the end of a format (e.g. different logical block size or protection information added). Removal of user data is only a side effect of a format.
============= ===============================================================================================================

Supported Disk Secure Erase Arguments
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _scrub:

Default argument for scrub is "nnsa", below table shows supported arguments for **scrub** tool:

======================= ============================================
Supported args          Description
======================= ============================================
nnsa                    4-pass NNSA Policy Letter NAP-14.1-C (XVI-8) for sanitizing removable and non-removable hard disks, which requires overwriting all locations with a  pseudo‐random pattern twice and then with a known pattern: random(x2), 0x00, verify. scrub default arg=nnsa
dod                     4-pass DoD 5220.22-M section 8-306 procedure (d) for sanitizing removable and non-removable rigid disks which requires overwriting all addressable locations with a character, its complement, a random character, then verify.  NOTE: scrub performs the random pass first to make verification easier:random, 0x00, 0xff, verify.
bsi                     9-pass  method  recommended by the German Center of Security in Information Technologies (http://www.bsi.bund.de): 0xff, 0xfe, 0xfd, 0xfb, 0xf7, 0xef, 0xdf, 0xbf, 0x7f.
fillzero                1-pass pattern: 0x00.
fillff                  1-pass pattern: 0xff.
random                  1-pass pattern: random(x1).
random2                 2-pass pattern: random(x2).
custom=0xdd             1-pass custom pattern.
gutmann                 The canonical 35-pass sequence described in Gutmann's paper cited below.
schneier                7-pass method described by Bruce Schneier in "Applied Cryptography" (1996): 0x00, 0xff, random(x5)
pfitzner7               Roy Pfitzner's 7-random-pass method: random(x7).
pfitzner33              Roy Pfitzner's 33-random-pass method: random(x33).
old                     6-pass pre-version 1.7 scrub method: 0x00, 0xff, 0xaa, 0x00, 0x55, verify.
fastold                 5-pass pattern: 0x00, 0xff, 0xaa, 0x55, verify.
usarmy                  US Army AR380-19 method: 0x00, 0xff, random. The same with dod option
======================= ============================================

.. _hdparm:

Default argument for hdparm is "security-erase", below table shows supported arguments for **hdparm** tool:

======================= ============================================
Supported args          Description
======================= ============================================
security-erase          Issue ATA Secure Erase (SE) command. hdparm default arg="security-erase"
security-erase-enhanced Enhanced SE is more aggressive in that it ought to wipe every sector: normal, HPA, DCO, and G-list. Not all drives support this command
======================= ============================================

.. _sg_sanitize:

Default argument for sg_sanitize is "block", below table shows supported arguments for **sg_sanitize** tool:

======================= ============================================
Supported args          Description
======================= ============================================
block                   Perform a "block erase" sanitize operation. sg_sanitize default arg="block"
fail                    Perform an "exit failure mode" sanitize operation.
crypto                  Perform a "cryptographic erase" sanitize operation.
======================= ============================================

.. _sg_format:

Default argument for sg_format is "1", below table shows supported arguments for **sg_format** tool:

======================= ============================================
Supported args          Description
======================= ============================================
"1"                     Disable Glist erasing. sg_format default arg="1
"0"                     Enable Glist erasing
======================= ============================================

Disk Secure Erase Workflow Notes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Please pay attention to below items if you are using RackHD secure erase function:

* **Use RackHD to manage RAID operation**. RackHD relies on its catalog data for secure erase. If RAID operation is not done via RackHD, RackHD secure erase workflow is not able to recognize drive names given. A suggestion is to re-run discovery for the compute node if you did changed RAID configure not using RackHD
* **Secure Erase is time-consuming**. Hdparm, sg_format and sg_sanitize will leverage drive firmware to do secure erase, even so it might take hours for a 1T drive. Scrub is overwriting data to disks and its speed is depends on argument you chose. For a "gutmann" argument, it will take days to erase a 1T drive.
* **Cancel Secure Erase workflow can't cancel secure erase operation**. Hdparm, sg_sanitize and sg_format are leverage drive firmware to do secure erase, once started there is no proper way to ask drive firmware to stop it till now.
* **Power cycle is risky**. Except for scrub tool, other tools are actually issue a command to drive and drive itself will control secure erase. That means once you started secure erase workflow, you can't stop it until it is completed. If you power cycled compute node under this case, drive might be frozen, locked or in worst case bricked. All data will not be accessible. If this happens, you need extra effort to bring your disks back to normal status.
