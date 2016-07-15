The RackHD Task Annotation is a web UI documentation for RackHD tasks.

It mainly explains each option of every task according to the related task schema.

A web UI page will be dynamically generated everytime in build process.


**How to Build Task Annotation Manually**

.. code-block:: shell

    git clone https://github.com/RackHD/on-http
    cd on-http
    npm install
    npm run taskdoc
    

You can access it via **http(s)://<server>:<port>/taskdoc**, when on-http service is running.

See example:

.. image:: /_static/task_annotation.png
  :align: center
