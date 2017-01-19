Workflows Progress
---------------------------

RackHD workflow progress feature provide message notification mechanism to indicate status of an active workflow or task. User can get to know what have been done and what is left for an active workflow or task with progress message. It is helpful when user is running time-consuming workflows like OS installation or secure erase.

Progress Message Publishment Criteria
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

RackHD will publish a workflow progress message if any of below events happened:

* Workflow started or finished events
* Task started or finished events
* RackHD marked important milestone events achieved for an active long-run task.

  In some cases RackHD can't easily get progress information, some milestones are created to divide a task into several small sections. Progress message will be sent if any of those milestones is achieved.

* Progress timer timeout for an active long-run task.

  In some cases progress of a task is continuous, RackHD can't keep sending progress information all the time. Thus a timer is set and progress information is publish with a fixed interval.

Progress Message Payload
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Progress message is made up of 4 items as below:

=============== ======= ==============================================================================================
properties      Type    Description
=============== ======= ==============================================================================================
maximum         Integer Maximum step quantity for a workflow or a task. For tasks with continuous progress, it is 100.
value           Integer Completed step quantity for a workflow or a task.
                        For tasks with continuous progress,
                        it varies from 0-100, which is inversely calculated from percentage and rounded to integer if
                        calculation gives non-integer value.
percentage      String  Percentage of a workflow or task that is completed.
                        Normally `value` divided by `maximum` will give `percentage`.
                        However in the case that tasks have continuous progress and we can only get percentage
                        instead of `maximum/value`, `maximum` will be always
                        set to 100 and `value` will be set to the percent number.
                        For example, a percentage "65%" will give `maximum` 100 and `value` 65.
description     String  Short description for progress message events
=============== ======= ==============================================================================================

Below is an example of message payload for a workflow that has 4 steps and we have just finished the first step. Percentage is 25% given by 1 / 4.

::

    progress: {
        value: 1,
        maximum: 4,
        description: 'Task "Install CentOS" started',
        percentage: '25%'
    }

A complete RackHD progress message contains two levels of progress information (refer to `Workflow Progress Measurement`_) as well as some useful information like graphId, graphName, nodeId, taskId and taskName, below is an example of a complete progress message:

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
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

RackHD progress information contains two levels of progress as shows in `Progress Message Payload Example`_ :

- `Task level progress`: progress measurement of the executing task of an active workflow.
- `Workflow level progress`: progress measurement of an active workflow.

Task progress is actually part of a workflow progress. However task and workflow has two independent progress measurement methods.

Workflow level progress measurement
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Workflow level progress is based on tasks counting. It is measured by completed tasks count (which will be assigned to `value`) against total tasks count (which will be assigned to `maximum`) for an active workflow.

Task level progress measurement
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

RackHD provides progress has different task level progress measurement methods for non-long-run tasks and two long-run tasks, OS installation tasks and secure erase task.

**Non-long-run task progress**

Every RackHD has two events:

- `task started`
- `task finished`

A non-long-run task will complete in short time and only the started and finished events can be sensed. Thus only two progress messages will be published for non-long-run tasks.

Besides task started and finished events, a time-consuming task is not proper to only publish two events, thus different measurements are created.

**OS installation task progress**

As a typical long-run task, OS installation task progress can not be easily measured. As a compromise, RackHD creates some milestones at important timeslot of installation process and divides the task into several sub-tasks.

Take CentOS installation for example, 4 milestones is created for this tasks:

- Profile downloaded: installation task start executing
- Kernel downloaded: installer kernel downloaded
- Installation started: installer is started
- Installation finished: installation and configuration completed

With completion of each subtask, RackHD will send progress information to user with `value` varies from 1 to 4 and `maximum` 4;

**Secure erase task progress**

For secure erase task, RackHD can get continuous progress information with percentage from node. Thus node is required to send the percentage data to RackHD with fixed internal. RackHD will receive and parser the percentage to get `value` and `maximum` and then publish progress message.

Progress Message Retrieving Channels
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As instant data, progress message can't be retrieved via API.
Instead progress messages will be published in AMQP channel and posted to webhook urls after adding RackHD standard message header.

Below are basic information for user to retrieve data with AMQP

- Exchange: on.events
- Routing Key: graph.progress.updated.information.<graphId>.<nodeId>

More details on RackHD AMQP events and webhook features, please refer to :doc:`event_notification`.

