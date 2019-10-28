# docs
RackHD documentation source for http://rackhd.readthedocs.io

[![Documentation Build Status](https://readthedocs.org/projects/rackhd/badge/?version=latest)](https://readthedocs.org/projects/rackhd/?badge=latest)

## Setting up for editing and building the docs

create a Virtualenv and set up the requirements

    virtualenv .venv
    source .venv/bin/activate
    pip install -r requirements.txt

build the docs

    cd docs
    make html

to auto-rebuild docs while you're editing

    cd docs
    sphinx-autobuild -H 0.0.0.0 -p 8000 . _build/html

and the docs will be visible at the url http://&lt;server ip>:8000  
the default is http://127.0.0.1:8000

