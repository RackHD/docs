Accessing RackHD APIs with Authorization
----------------------------------------

API access control is enabled when authentication is enabled.  The Access Control is controlled per
API and per API method.  A GET on an API can have different access control than a POST on the same API.

Roles
~~~~~

A role grants a set of privileges.  Each privilege is specified explicitly within the role.
Authenticated users have a single role assigned to them.

Privileges
~~~~~~~~~~

A privilege grants access to a API resource and an action to perform on that resource.  For example,
a 'read' privilege may grant GET access on a set of APIs, but may not also grant POST/PUT/PATCH/DELETE
access to those same APIs.  To issue POST/PUT/PATCH/DELETE methods to an API, a 'write' privilege 
may be required.

Built-in Roles
~~~~~~~~~~~~~~

The following Roles are built-in to RackHD:

.. list-table::
    :widths: 20 100
    :header-rows: 1

    * - Role
      - Description
    * - Administrator
      - Possess all built-in privileges
    * - ReadOnly
      - Possess Read, Login and ConfigureSelf privileges

Built-in Privileges
~~~~~~~~~~~~~~~~~~~

The following Privileges are built-in to RackHD:

.. list-table::
    :widths: 20 100
    :header-rows: 1

    * - Privilege
      - Description
    * - Read
      - Used to specify an ability to read data from an API
    * - Write
      - Used to specify an ability to write data to an API
    * - Login
      - Used to specify an ability to login to RackHD
    * - ConfigureUsers
      - Used to specify an ability to configure aspects of other users
    * - ConfigureSelf
      - Used to specify an ability to configure aspects of the logged in user


