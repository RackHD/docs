The RackHD Task Annotation is a web UI documentation for RackHD tasks.

The UI page leverage the template of `apidoc <https://github.com/apidoc/apidoc>`_ to display the auto generated task_doc_data.json.


**How to Build Task Annotation Manually**

.. code-block:: shell

    git clone https://github.com/RackHD/on-http
    cd on-http
    npm install
    npm run taskdoc
    

You can access it via http://127.0.0.1/taskdoc, when on-http service is running

.. image:: /_static/task_annotation.png
  :align: center
