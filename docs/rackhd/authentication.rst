Accessing RackHD APIs with Authentication
-----------------------------------------

When 'authEnabled' is set to 'true' in the config.json file for an endpoint, authentication
will be needed to access the APIs that are defined within that endpoint.  Enabling authentication
will also enable authorization control when accessing API 2.0 and Redfish APIs.

This section describes how to access APIs that need authentication.

Setup endpoints to enable authentication
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Please refer to :ref:`http-endpoint-config-ref-label` on how to setup endpoints. Simply put,
the following endpoint configuration will be a good start.::

    "httpEndpoints": [
        {
            "address": "0.0.0.0",
            "port": 8443,
            "httpsEnabled": true,
            "proxiesEnabled": false,
            "authEnabled": true,
            "routers": "northbound-api-router"
        },
        {
            "address": "172.31.128.1",
            "port": 8080,
            "httpsEnabled": false,
            "proxiesEnabled": false,
            "authEnabled": false,
            "routers": "southbound-api-router"
        }
    ]

The first endpoint represents an HTTPS service listening at port 8443 that serves northbound APIs, which are
APIs being called by users. Note that authEnabled is set to true means that authentication is needed
to access northbound APIs.

The second endpoint represents an HTTP service listening at port 8080 that serves southbound APIs, which are
called by nodes interacting with the system. Authentication should NOT be enabled for southbound APIs in
order for PXE to work fine.

.. _localhost-exception-label:

Setup the first user with Localhost Exception
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Please refer to :ref:`authentication-config-ref-label` on how to setup endpoints.

The localhost exception permits unauthenticated access to create the first user in the system.  With
authentication enabled, the first user can be created by issuing a POST to the /users API only if the
API is issued from localhost.  The first user must be assigned a role with privileges to create other
users, such as an Administrator role.

Here is an example of creating an initial 'admin' user with a password of 'admin123'.::

    curl -ks -X POST -H "Content-Type:application/json" https://localhost:8443/api/2.0/users -d '{"username": "admin", "password": "admin123", "role": "Administrator"}' | python -m json.tool
    {
        "role": "Administrator",
        "username": "admin"
    }

The localhost exception can be disabled by setting the configuration value "enableLocalHostException" to
false.  The default value of "enableLocalHostException" is true.

Login to get a token
~~~~~~~~~~~~~~~~~~~~

Following the endpoint settings, a token is needed to access any northbound APIs, except the /login API.

Posting a request to /login with username and password in the request body will get a token returned from
RackHD, which will be used to access any other northbound APIs.

Here is an example of getting a token using curl.::

    curl -k -X POST -H "Content-Type:application/json" https://localhost:8443/login -d '{"username":"admin", "password":"admin123" }' | python -m json.tool
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    100   204  100   160  100    44   3315    911 --:--:-- --:--:-- --:--:--  3333
    {
        "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiYWRtaW4iLCJpYXQiOjE0NTU2MTI5MzMsImV4cCI6MTQ1NTY5OTMzM30.glW-IvWYDBCfDZ6cS_6APoty22PE_Ir5L1mO-YqO3eE"
    }

A 401 unauthorized response with 'Invalid username or password' message will be returned if:

- Username or password is wrong in the http request body

For example:::

    curl -k -X POST -H "Content-Type:application/json" https://localhost:8443/login -d '{"username":"admin", "password":"admin123balabala" }' | python -m json.tool
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    100    94  100    42  100    52    909   1125 --:--:-- --:--:-- --:--:--  1130
    {
        "message": "Invalid username or password"
    }

Accessing API using the token
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There are three ways of using the token in a http/https request:

- send the token as a query string
- send the token as a query header
- send the token as request body

Example of sending the token as query string:::

    curl -k -H "Content-Type:application/json" https://localhost:8443/api/1.1/config?auth_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiYWRtaW4iLCJpYXQiOjE0NTU2MTI5MzMsImV4cCI6MTQ1NTY5OTMzM30.glW-IvWYDBCfDZ6cS_6APoty22PE_Ir5L1mO-YqO3eE | python -mjson.tool
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    100  1919  100  1919    0     0  81114      0 --:--:-- --:--:-- --:--:-- 83434
    {
        "$0": "index.js",
        ...
        "tftpRoot": "./static/tftp"
    }

Example of sending the token as query header.

**Note**: the header should be 'authorization' and the token
should start will 'JWT' followed by a whitespace and then the token itself.::

    curl -k -H "Content-Type:application/json" https://localhost:8443/api/1.1/config --header 'authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiYWRtaW4iLCJpYXQiOjE0NTU2MTI5MzMsImV4cCI6MTQ1NTY5OTMzM30.glW-IvWYDBCfDZ6cS_6APoty22PE_Ir5L1mO-YqO3eE' | python -mjson.tool
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    100  1919  100  1919    0     0    99k      0 --:--:-- --:--:-- --:--:--  104k
    {
        "$0": "index.js",
        ...
        "tftpRoot": "./static/tftp"
    }

Example of sending the token as query body:::

    curl -k -X POST -H "Content-Type:application/json" https://localhost:8443/api/1.1/lookups -d '{"auth_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiYWRtaW4iLCJpYXQiOjE0NTU2MTI5MzMsImV4cCI6MTQ1NTY5OTMzM30.glW-IvWYDBCfDZ6cS_6APoty22PE_Ir5L1mO-YqO3eE","macAddress":"aa:bb:cc:dd:ee:ff", "ipAddress":"192.168.1.1", "node":"123453134" }' | python -m json.tool
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    100   599  100   353  100   246  19932  13890 --:--:-- --:--:-- --:--:-- 20764
    {
        "auth_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiYWRtaW4iLCJpYXQiOjE0NTU2MTI5MzMsImV4cCI6MTQ1NTY5OTMzM30.glW-IvWYDBCfDZ6cS_6APoty22PE_Ir5L1mO-YqO3eE",
        "createdAt": "2016-02-16T09:07:29.995Z",
        "id": "56c2e6d140408f6a2d17cb23",
        "ipAddress": "192.168.1.1",
        "macAddress": "aa:bb:cc:dd:ee:ff",
        "node": "123453134",
        "updatedAt": "2016-02-16T09:07:29.995Z"
    }

A 401 unauthorized response with a 'invalid signature' message will be returned if:

- Invalid token found in query string, header or request body

For example:::

    curl -k -H "Content-Type:application/json" https://localhost:8443/api/1.1/config --header 'authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiYWRtaW4iLCJpYXQiOjE0NTU2MTI5MzMsImV4cCI6MTQ1NTY5OTMzM30.glW-IvWYDBCfDZ6cS_6APoty22PE_Ir5L1mO-YqO3eE-----------' | python -mjson.tool
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    100    31  100    31    0     0   1806      0 --:--:-- --:--:-- --:--:--  1823
    {
        "message": "invalid signature"
    }

A 401 bad request response with a 'No auth token' message will be returned if:

- Empty token in request body, ie, auth_token="" or authorization=""
- No auth_token key in query string or request body, or
- No authorization key in request header

For example:::

    curl -k -H "Content-Type:application/json" https://localhost:8443/api/1.1/config | python -mjson.tool                                                                   % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    100    27  100    27    0     0   1644      0 --:--:-- --:--:-- --:--:--  1687
    {
        "message": "No auth token"
    }

Creating a Redfish Session
~~~~~~~~~~~~~~~~~~~~~~~~~~

Posting a request to the Redfish Session Service with UserName and Password in the request body will get a token returned from
the Redfish service which can be used to access any other Redfish APIs.  The token is returned in the 'X-Auth-Token' header in
the response object.

Here is an example of getting a token using curl.::

    curl -vk -X POST -H "Content-Type:application/json" https://localhost:8443/redfish/v1/SessionService/Sessions -d '{"UserName":"admin", "Password":"admin123" }' | python -m json.tool
    < HTTP/1.1 200 OK
    < X-Powered-By: Express
    < Access-Control-Allow-Origin: *
    < X-Auth-Token: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiYWRtaW4iLCJpZCI6ImNlYjk0MzIzLTQyZDYtNGM3MC05ZDIxLTEwNWYyYThlNWNjOCIsImlhdCI6MTQ3MzcwNzM5OCwiZXhwIjoxNDczNzkzNzk4fQ.EpxRI911dS25-yr3CiSI-RzvrgM9JYioQUqdKq6HQ1k
    < Content-Type: application/json; charset=utf-8
    < Content-Length: 294
    < ETag: W/"126-K9SNCTT10D9033EnNBAPcQ"
    < Date: Mon, 12 Sep 2016 19:09:58 GMT
    < Connection: keep-alive
    <
    { [data not shown]
    100   338  100   294  100    44   4785    716 --:--:-- --:--:-- --:--:--  4819
    * Connection #0 to host localhost left intact
    {
        "@odata.context": "/redfish/v1/$metadata#SessionService/Sessions/Members/$entity",
        "@odata.id": "/redfish/v1/SessionService/Sessions",
        "@odata.type": "#Session.1.0.0.Session",
        "Description": "User Session",
        "Id": "ceb94323-42d6-4c70-9d21-105f2a8e5cc8",
        "Name": "User Session",
        "Oem": {},
        "UserName": "admin"
    }

A 401 unauthorized response will be returned if:

- Username or password is wrong in the http request body

For example:::

    curl -vk -X POST -H "Content-Type:application/json" https://localhost:8443/redfish/v1/SessionService/Sessions -d '{"UserName":"admin", "Password":"bad" }' | python -m json.tool
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    < HTTP/1.1 401 Unauthorized
    < X-Powered-By: Express
    < Access-Control-Allow-Origin: *
    < Content-Type: text/html; charset=utf-8
    < Content-Length: 12
    < ETag: W/"c-4G0bpw8TMen5oRPML4h9Pw"
    < Date: Mon, 12 Sep 2016 19:11:33 GMT
    < Connection: keep-alive
    <
    { [data not shown]
    100    56  100    12  100    44    195    716 --:--:-- --:--:-- --:--:--   721
    * Connection #0 to host localhost left intact
    No JSON object could be decoded

Once the X-Auth-Token is acquired, it can be included in all future Redfish requests by adding a X-Auth-Token
header to the request object:::

    curl -k -H "Content-Type:application/json" -H 'X-Auth-Token:eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiYWRtaW4iLCJpZCI6ImNlYjk0MzIzLTQyZDYtNGM3MC05ZDIxLTEwNWYyYThlNWNjOCIsImlhdCI6MTQ3MzcwNzM5OCwiZXhwIjoxNDczNzkzNzk4fQ.EpxRI911dS25-yr3CiSI-RzvrgM9JYioQUqdKq6HQ1k' https://localhost:8443/redfish/v1/SessionService/Sessions | python -m json.tool
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    100   784  100   784    0     0  27303      0 --:--:-- --:--:-- --:--:-- 28000
    {
        "@odata.context": "/redfish/v1/$metadata#SessionService/Sessions/$entity",
        "@odata.id": "/redfish/v1/SessionService/Sessions",
        "@odata.type": "#SessionCollection.SessionCollection",
        "Members": [
            {
                "@odata.id": "/redfish/v1/SessionService/Sessions/ceb94323-42d6-4c70-9d21-105f2a8e5cc8"
            }
        ],
        "Members@odata.count": 1,
        "Name": "Session Collection",
        "Oem": {}
    }

Deleting a Redfish Session
~~~~~~~~~~~~~~~~~~~~~~~~~~

To invalidate a Redfish session token, the respective session instance should be deleted:::

    curl -k -X DELETE -H "Content-Type:application/json" -H 'X-Auth-Token:eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiYWRtaW4iLCJpZCI6ImNlYjk0MzIzLTQyZDYtNGM3MC05ZDIxLTEwNWYyYThlNWNjOCIsImlhdCI6MTQ3MzcwNzM5OCwiZXhwIjoxNDczNzkzNzk4fQ.EpxRI911dS25-yr3CiSI-RzvrgM9JYioQUqdKq6HQ1k' https://localhost:8443/redfish/v1/SessionService/Sessions/ceb94323-42d6-4c70-9d21-105f2a8e5cc8 | python -m json.tool
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
      0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
    No JSON object could be decoded

Once the session has been deleted, the session token will no longer be valid:::

    curl -vk -H "Content-Type:application/json" -H 'X-Auth-Token:eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiYWRtaW4iLCJpZCI6ImNlYjk0MzIzLTQyZDYtNGM3MC05ZDIxLTEwNWYyYThlNWNjOCIsImlhdCI6MTQ3MzcwNzM5OCwiZXhwIjoxNDczNzkzNzk4fQ.EpxRI911dS25-yr3CiSI-RzvrgM9JYioQUqdKq6HQ1k' https://localhost:8443/redfish/v1/SessionService/Sessions | python -m json.tool
    < HTTP/1.1 401 Unauthorized
    < X-Powered-By: Express
    < Access-Control-Allow-Origin: *
    < Content-Type: application/json; charset=utf-8
    < Content-Length: 2
    < ETag: W/"2-mZFLkyvTelC5g8XnyQrpOw"
    < Date: Mon, 12 Sep 2016 20:04:32 GMT
    < Connection: keep-alive
    <
    { [data not shown]
    100     2  100     2    0     0     64      0 --:--:-- --:--:-- --:--:--    66
    * Connection #0 to host localhost left intact
    {}

