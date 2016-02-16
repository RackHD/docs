Accessing RackHD APIs with Authentication
-----------------------------------------

With authentication is enabled in monorail.json configure file for an endpoint,
see `Setting up HTTP/HTTPS endpoint`_ and `Authentication`_, authentication is needed for
accessing the API group associated with the specified endpoint.

This section describes how to access APIs that need authentication.

Setup endpoints to enable authentication
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Please refer to `Setting up HTTP/HTTPS endpoint`_ on how to setup endpoints. Simply put,
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
    ],

The first endpoint represents a HTTPs service listening at port 8443 that serves northbound APIs, which are
APIs that being called by users. Noticed that authEnabled is set to true means the authentication is needed
for accessing northbound APIs.

The second endpoint represent a HTTP service listening at port 8080 that serves southbound APIs, which are
called by nodes. Authentication should NOT be enabled for southbound APIs in order for PXE to work fine.

Setup credentials and token for authentication
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Please refer to `Authentication`_ on how to setup endpoints.

Copy following items to /opt/onrack/etc/monorail.json if they are not there yet.::

    "authUsername": "admin",
    "authPasswordHash": "KcBN9YobNV0wdux8h0fKNqi4uoKCgGl/j8c6YGlG7iA0PB3P9ojbmANGhDlcSBE0iOTIsYsGbtSsbqP4wvsVcw==",
    "authPasswordSalt": "zlxkgxjvcFwm0M8sWaGojh25qNYO8tuNWUMN4xKPH93PidwkCAvaX2JItLA3p7BSCWIzkw4GwWuezoMvKf3UXg==",
    "authTokenSecret": "RackHDRocks!",
    "authTokenExpireIn": 86400,

Following above configuration, the default username/password for login are set to be admin/admin123,
and the token will expire 24 hours after it's generated.

Please follow the instruction on `Change the default password`_ if there is a need to change the
default password.

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

    curl -k -H "Con443/api/1.1/config?auth_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiYWRtaW4iLCJpYXQiOjE0NTU2MTI5MzMsImV4cCI6MTQ1NTY5OTMzM30.glW-IvWYDBCfDZ6cS_6APoty22PE_Ir5L1mO-YqO3eE | python -mjson.tool
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    100  1919  100  1919    0     0  81114      0 --:--:-- --:--:-- --:--:-- 83434
    {
        "$0": "index.js",
        "CIDRNet": "172.31.128.0/22",
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
        "CIDRNet": "172.31.128.0/22",
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

A 401 unauthorized response with 'invalid signature' message will be returned if:

- Invalid token found in query string, header or request body

For example:::

    curl -k -H "Content-Type:application/json" https://localhost:8443/api/1.1/config --header 'authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiYWRtaW4iLCJpYXQiOjE0NTU2MTI5MzMsImV4cCI6MTQ1NTY5OTMzM30.glW-IvWYDBCfDZ6cS_6APoty22PE_Ir5L1mO-YqO3eE-----------' | python -mjson.tool
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    100    31  100    31    0     0   1806      0 --:--:-- --:--:-- --:--:--  1823
    {
        "message": "invalid signature"
    }

A 401 bad request response with 'No auth token' message will be returned if:

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