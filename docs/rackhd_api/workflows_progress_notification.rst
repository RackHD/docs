Workflow Progress Notification
==============================

.. contents:: Table of Contents

RackHD workflow progress feature provides message notification mechanism to indicate status of an active workflow or task. User can get to know what has been done and what is to be done for an active workflow or task with progress messages.

Workflow Progress Events
-----------------------------

RackHD will publish a workflow progress message if any of below events happens:

* Workflow started or finished events
* Task started or finished events
* RackHD marked important milestone events for an active long-run task.

  In some cases RackHD can't easily get progress information, some milestones are created to divide a task into several small sections. Progress messages will be sent if any of those milestones is achieved.

* Progress timer timeout for an active long-run task.

  Some tasks don't have milestones but progress information is continuous and can be got all the time. In this case progress messages is generated with fixed interval.

Progress Message Payload
-----------------------------

4 attributes are used to describe progress information:

=============== ======= ==============================================================================================
properties      Type    Description
=============== ======= ==============================================================================================
maximum         Integer Maximum step quantity for a workflow or a task.
                        For tasks with continuous progress, it is 100.
value           Integer Completed step quantity for a workflow or a task.
                        For tasks with continuous progress,
                        it varies from 0-100,
                        which is inversely calculated from percentage and rounded to integer if
                        calculation gives non-integer value.
percentage      String  Percentage of a workflow or task that is completed.
                        Normally `value` divided by `maximum` will give `percentage`.
                        However in the case that tasks have continuous progress, percentage is directly got.
                        In this case `maximum` will be always set to 100 and `value` will be set to the percent number.
                        For example, a percentage "65%" will give `maximum` 100 and `value` 65.
description     String  Short description for progress events
=============== ======= ==============================================================================================

Below is an example of progress information payload for a workflow that has 4 steps and we have just finished the first step. Percentage is 25% given by 1 / 4.

::

    progress: {
        value: 1,
        maximum: 4,
        description: 'Task "Install CentOS" started',
        percentage: '25%'
    }

A complete RackHD progress message payload contains two levels of progress information (refer to `Workflow Progress Measurement`_) as well as some useful information like graphId, graphName, nodeId, taskId and taskName, below is an example of a complete progress message:

.. _Progress Message Payload Example:

::

    {
        progress: {
            value: 1,
            maximum: 4,
            description: 'Task "Install CentOS" started',
            percentage: '25%'
        },
        graphName: 'Install CentOS',
        graphId: '12a8f275-7abf-46ee-834b-6aa34cce8d78',
        nodeId: '58542c752be86d0672cef383',
        taskProgress: {
            taskId: 'cb7d5793-abcf-4a7f-aef6-e768e999de1d',
            taskName: 'Install CentOS',
            progress: {
                value: 0,
                maximum: 4,
                description: 'Task started',
                percentage: '0%'
            }
        }
    }

Though RackHD provides percentage number as progress measurement in progress message, most of the time workflow progress is based on events counting. RackHD progress message is not always proper to be used for workflow executing time estimation.

.. _Workflow Progress Measurement:

Workflow Progress Measurement
-----------------------------

RackHD progress information contains two levels of progress as shows in `Progress Message Payload Example`_ :

- `Task level progress`: progress measurement of the executing task of an active workflow.
- `Workflow level progress`: progress measurement of an active workflow.

Task progress is actually part of workflow progress. However task and workflow have two independent progress measurement methods.

Workflow level progress measurement
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Before a workflow's completion workflow level progress is based on tasks counting. It is measured by completed tasks count (which will be assigned to `value`) against total tasks count (which will be assigned to `maximum`) for the workflow.

Percentage will be set to 100% and `value` be set to `maximum` at workflow's completion. After completion workflow level progress will not be updated even though some tasks may still be running.

Task level progress measurement
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

RackHD has different task level progress measurement methods for non-long-run tasks and two long-run tasks, OS installation tasks and secure erase task.

**Non-long-run task progress**

Each RackHD task has two progress events:

- `task started`
- `task finished`

A non-long-run task will complete in short time and only the started and finished events can be sensed. Thus only two progress messages will be published for non-long-run tasks.

Besides task started and finished events, a time-consuming task is not proper to only publish two events, thus different measurements are created.

**OS installation task progress**

As a typical long-run task, OS installation task progress can't be easily measured. As a compromise, RackHD creates some milestones at important timeslot of installation process thus divides OS install task into several sub-tasks.

Below table includes descriptions for all existing RackHD OS installation milestones:

=================== ==============================================================================================
Milestone name      Milestone description
=================== ==============================================================================================
requestProfile      Enter ipxe and request OS installation profile. Common milestone for all OSes.
enterProfile        Enter profile, start to download kernel or installer. Common milestone for all OSes.
startInstaller      Start installer and prepare installation. Common milestone for all OSes.
preConfig           Enter Pre OS configuration.
startSetup          Net use Windows Server 2012 and start setup.exe. Only used for Windows Server.
installToDisk       Execute OS installation. Only used for CoreOS.
startPartition      Start partition. Only used for Ubuntu.
postPartitioning    Finished partitioning and mounting, start package installation. Only used for SUSE.
chroot              Finished package installation, start first boot. Only used for SUSE.
postConfig          Enter Post OS configuration.
completed           Finished OS installation. Common milestone for all OSes.
=================== ==============================================================================================

Below table includes default milestone sequence for RackHD supported OSes:

=============== =================== ==============================================================================================
OS Name         Milestone Quantity  Milestones in Sequence
=============== =================== ==============================================================================================
CentOS, RHEL    6                   1.requestProfile; 2.enterProfile; 3.startInstaller; 4.preConfig; 5.postConfig; 6.completed
Esxi            6                   1.requestProfile; 2.enterProfile; 3.startInstaller; 4.preConfig; 5.postConfig; 6.completed
CoreOS          5                   1.requestProfile; 2.enterProfile; 3.startInstaller; 4.installToDisk; 5.completed
Ubuntu          7                   1.requestProfile; 2.enterProfile; 3.startInstaller; 4.preConfig; 5.startPartition; 6.postConfig; 7.completed
WindowServer    5                   1.requestProfile; 2.enterProfile; 3.startInstaller; 4.startSetup; 5.completed
SUSE            7                   1.requestProfile; 2.enterProfile; 3.startInstaller; 4.preConfig; 5.postPartitioning; 6.chroot; 7.completed
PhotonOS        5                   1.requestProfile; 2.enterProfile; 3.startInstaller; 4.postConfig; 5.completed
=============== =================== ==============================================================================================

In progress message, milestone quantity will be set to `maximum` and sequence number to `value` while RackHD is installing OS.

**Secure erase task progress**

For secure erase task, RackHD can get continuous percentage progress from node. Thus node is required to send the percentage data to RackHD with fixed interval. RackHD will receive and parse the percentage to get `value` and `maximum` and then publish progress message.

Progress Message Retrieve Channels
----------------------------------

As instant data, progress messages can't be retrieved via API.
Instead progress messages will be published in AMQP channel and posted to webhook urls after adding RackHD standard message header.

Below is basic information for user to retrieve data from AMQP channel:

- Exchange: on.events
- Routing Key: graph.progress.updated.information.<graphId>.<nodeId>

More details on RackHD AMQP events and webhook feature, please refer to :doc:`northbound_event_notification`.

