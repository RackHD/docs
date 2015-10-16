# Creating a local debian package repo

install pre-req packages:

    sudo apt-get install reprepro, nginx

create repo structure:

    mkdir -p /var/repository/conf
    touch /var/repository/conf/distributions

edit distributions files

    Origin: Renasar, Inc
    Label: Renasar, Inc
    Codename: trusty
    Architectures: amd64
    Components: non-free

set up nxginx to serve the repository

    sudo rm /etc/nginx/sites-enabled/default
    vi /etc/nginx/sites-available/repo

repo:

    server {

        ## Let your repository be the root directory
        root        /var/repository;

        ## Prevent access to Reprepro's files
        location ~ /(db|conf) {
            deny        all;
            return      404;
        }
    }

link in the nginx config:

    sudo ln /etc/nginx/sites-enabled/repo /etc/nginx/sites-available/repo

test the config:

    sudo nginx -t

restart nginx to enable:

    sudo service nginx restart

## repo manipulation

adding a debian file

    reprepro -Vb /var/repository includedeb trusty /tmp/something.deb


listing the packages:

    reprepro -b repository list trusty
