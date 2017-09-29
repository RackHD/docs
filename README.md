# docs
RackHD documentation source for http://rackhd.readthedocs.io

[![Documentation Build Status](https://readthedocs.org/projects/rackhd/badge/?version=latest)](https://readthedocs.org/projects/rackhd/?badge=latest)

Copyright © 2017 Dell Inc. or its subsidiaries.  All Rights Reserved. 

## Setting up for editing and building the docs

create a Virtualenv and set up the requirements

    virtualenv .venv
    source .venv/bin/activate
    pip install -r requirements.txt

build the docs

    cd docs
    make html

to auto-rebuild docs while you're editing, also at the same directory (directory named docs), execute

    sphinx-autobuild -H 0.0.0.0 -p 8000 . _build/html

and the docs will be visible at the url http://&lt;server ip>:8000  
the default is http://127.0.0.1:8000

## Setting up for editing and building the docs (Windows)

* download and install python: https://www.python.org/ftp/python/2.7.10/python-2.7.10.amd64.msi
  * install for all users
  * default location (c:\python27)
  * all default components

* clone these docs from github and open in a command prompt

* install and set up the virtualenv

```
c:\python27\scripts\pip install virtualenv
c:\python27\scripts\virtualenv .venv
.venv\scripts\activate
pip install -r requirements.txt
```

* make the docs

```
.venv\scripts\activate
cd docs
.\make.bat html
```

* open the docs in a browser

```
.venv\scripts\activate
cd docs
start .\_build\html\index.html
```

* to auto-rebuild the docs on editing with live updates

```
sphinx-autobuild -H 0.0.0.0 -p 8000 . _build/html
```

and navigate to http://&lt;server ip>:8000 to see the docs  
the default url is http://127.0.0.1:8000


## Licensing

Licensed under the Apache License, Version 2.0 (the “License”); you may not use this file except in compliance with the License. You may obtain a copy of the License at http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an “AS IS” BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.

RackHD is a Trademark of Dell EMC
