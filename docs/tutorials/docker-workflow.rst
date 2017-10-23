Workflow Editor
================

Step 1: Configure on-web-ui
----------------------------

1. On the Windows desktop of launchpad, open Chrome, and then and go to the following URL. ``http://localhost:9090/ui``

2. click the "gear" button on the left panel

3. leave the 2 switches as default ( in this Lab, in the /opt/monorail/config.json, the https is enabled and api authentication is disabled )

4. On the Windows desktop of launchpad, verify the API 2.0 end point (127.0.0.1:9090/api/2.0)

5. click "apply settings" button on the bottom.

.. image:: ../_static/workflow_UI.png
     :align: center

Step 2: Try on-web-ui
-----------------------

1. Click the meter button in the left panel.
2. You can view the nodes list in the table.
3. You can view the workflow history in the table.

.. image:: ../_static/workflow_op1.png
     :align: center

4. Click a compute node in the Node List.

.. image:: ../_static/workflow_op2.png
     :align: center

5. In the right panel, you can view the different APIs that are available, such as pollers, catalogs, and so on.

6. Experiment with the catalog of a node by clicking the "Catalogs" button.

.. image:: ../_static/workflow_op3.png
     :align: center

7. Try one of the catalogs link shown in the available catalogs list. Example: click "SMART" to show the Disks S.M.A.R.T information captured on the node.

.. image:: ../_static/workflow_op4.png
     :align: center

8. Click the "Operations Center" icon on the left panel

9. You can view the workflow history and the current running workflow status.

10. Click one the the workflow (example: "Discovery") to view the workflow diagram and status.

.. image:: ../_static/workflow_op5.png
     :align: center


Step 3: Create A New Workflow
-----------------------------

In this session, you will customize a RackHD workflow to implement your own logic.

**Workflow Scenario**

You have a number of new bare metal servers coming online.

- Before the OS and applications are deployed to the new servers, you want to run a quick sanity check (diagnostic) on the servers.

- Due to a special demand of your application, you want to include a temperature check and CPU frequency check in the diagnostic step.

To fulfill the demand of scenario, you can use On-Web-UI to customize a new workflow named My_Workflow.

This example is a simple one. However, your customized workflows can be as complex as needed.


**"Workflow" in RackHD**

A workflow in RackHD is a JSON document, which describes a flow of execution and is built as a graph. A graph is composed by several tasks.

The tasks can be executed in serial or in parallel. Each task has a conditional output that can be used to drive the workflow down different paths based on how the task is completed (for example, Error, Failed, Succeeded).

Add A New Workflow
~~~~~~~~~~~~~~~~~~

1. Click the Workflow Editor button on the left panel.
2. Type your workflow name (My_Workflow)
3. Press Enter on your keyboard. Do not use the Save button on the right.

.. image:: ../_static/workflow_op6.png
     :align: center

4. On the pop up Confirm diagram, click "SUBMIT"

.. image:: ../_static/workflow_op7.png
     :align: center

The Web-UI refreshes itself.

5. Click the Workflow Editor button on the left panel.

6. Type My_Workflow on the name box. The name is auto-populated. You can select the workflow you created.

.. image:: ../_static/workflow_op8.png
     :align: center

The on-web-ui will show there's a dummy operation (no-op) in this workflow.

7. Use your mouse wheel to zoom in and zoom out on the view.

8. Drag and drop from left to right to move the view point.

.. image:: ../_static/workflow_op9.png
     :align: center

9. On the right side, above the panel that displays the workflow source code, in the Task field, type **Set Node Pxeboot**, to select an existing task.

10. Click the + button, to add the task to your customized workflow.

.. image:: ../_static/workflow_op10.png
     :align: center

11. Then a piece of workflow source code(json) will be appended into your workflow code .

12. On the left view, a new "task box" appeared, it will be named as "new-task-xxxxxx" (xxxxxx is randomly generated)

13. To make the name more readable, please change the label name from "new-task-xxxxxx" to **"set-boot-pxe"** (by clicking the string on the box then you can edit it.)

.. image:: ../_static/workflow_op11.png
     :align: center

14. As below example, the newly added box has been renamed to **set-boot-pxe**.

.. image:: ../_static/workflow_op12.png
     :align: center

15. Select the existing task Reboot Node.

16. Click the + button. The new task is added to the source code and a new task box is added to the visual editor.

17. Change the box name from random generated string to reboot.

.. image:: ../_static/workflow_op13.png
     :align: center

``[Note]`` Besides, you need to edit the code block of **Reboot**, as is shown in the picture above.

18. Select the existing task Boostrap Ubuntu

19. Click the + button.

20. Change the newly added box name to boostrap-ubuntu

.. image:: ../_static/workflow_op14.png
     :align: center

Customize A Shell Command Task
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. In the Task field, type Shell commands, to add a task.

2. Click the + button.

.. image:: ../_static/workflow_op15.png
     :align: center

3. Change the new task's name to Diagnostic by clicking the name on the box.

.. image:: ../_static/workflow_op16.png
     :align: center

4. In the workflow editor window on the right hand side, you can see three default shell commands for the Diagnostic task that you created.

The following example shows the default, automatically generated, json output.

.. code::

  "commands": [
   {
     "command": "sudo ls /var",
     "catalog": {
     "format": "raw",
     "source": "ls var"
     }
   },
   {
     "command": "sudo lshw -json",
     "catalog": {
     "format": "json",
     "source": "lshw user"
     }
   },
   {
     "command": "test",
     "acceptedResponseCodes": [ 1 ]
   }
  ]

.. image:: ../_static/workflow_op17.png
     :align: center

Set The Task Relationship
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Tasks display indicators that you can connect to set the task relationship. Each task displays a trigger indicator in the top left.

Each task also displays the following condition indicators on the right side:

- Red: when fail
- Green: when success
- Blue: when finish

For example, when you connect the green condition indicator of task A to the trigger indicator for Task B: when task A has succeeded, then task B is triggered.

1. Connect the blue condition indicator of the set-boot-pxe task to the trigger indicator of the reboot task: whether the set-boot-pxe task is successful or not, the reboot task is triggered

.. image:: ../_static/workflow_op19.png
     :align: center

2. Connect the green condition indicator of the reboot task to the trigger indicator of the bootstrap-ubuntu task.

When the reboot task is successfully completed, the bootstrap-ubuntu task is started.

Note: Use your mouse wheel to zoom in and zoom out on the view. Drag and drop from left to right to move the view point.

.. image:: ../_static/workflow_op20.png
     :align: center

3. Click x to remove the no-op task.

.. image:: ../_static/workflow_op21.png
     :align: center

4. Connect the green condition indicator for the reboot task to the trigger indicator for the Diagnostic task.

5. View your new workflow.

.. image:: ../_static/workflow_op22.png
     :align: center

Save The Workflow
~~~~~~~~~~~~~~~~~

1. Click the save icon to save the workflow

.. image:: ../_static/workflow_op23.png
     :align: center


Step 4: Run The New Workflow
----------------------------

Click the run icon, to run the workflow that you created in 7.5.4.

.. image:: ../_static/workflow_op24.png
     :align: center


On the pop up diagram,

1. Select a node (Note: choose a compute node identified with a MAC address, instead of an Enclosure Node.)

2. Click **SAVE** to run this workflow

.. image:: ../_static/workflow_op25.png
     :align: center

3. On the desktop, double-click the UltraVNC Viewer tool, to check the bootstrap progress of the node you sent this workflow to.

4. Click the Operations Center tab. You can see that My_Workflow" is running. The target node ID is under the workflow name.

5. Click the running My_Workflow, to view the progress. After several minutes, the workflow is completed, and the color of the workflow indicates the running result (red for fail, yellow for canceled, green for success).

.. image:: ../_static/workflow_op26.png
     :align: center
