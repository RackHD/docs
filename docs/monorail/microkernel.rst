MicroKernel and Overlays
----------------------------------------------------------

Creating and Modifying Overlays
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Certain operations enabled by the monorail server, primarily node cataloging and
firmware flashing, is performed within small-medium sized linux images that are
booted into RAM (as a tmpfs). To optimize storage and download of these images,
we use [overlayfs]
(https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/Documentation/filesystems/overlayfs.txt),
which allows a machine to mount two filesystems together as one merged filesystem.
What this allows us to do is store the essential components common to every live
image we use in a single image (a squashfs image), and store custom binaries and
scripts in much smaller overlay filesystem archives, saving disk space that has
to be dedicated to image storage by pushing variations into the overlay space.
It also makes it easy to change, re-package and re-distribute the contents of an
overlay on any platform without having to touch the base image.

NOTE: Overlayfs only supports two filesystems layered together, e.g. you
cannot have 3 or 4 filesystem layers.

**Example**

Let's say we have a base image representing a standard Linux file structure:

.. code-block:: bash

    bin    boot    dev    etc    grub.debconf    home
    initrd.img -> boot/initrd.img-3.13.0-32-generic    lib
    lib64    media    mnt    opt    proc    root    run
    sbin    srv    sys    tmp    usr    var
    vmlinuz -> boot/vmlinuz-3.13.0-32-generic


And we have a simple overlay file structure that only contains a couple of scripts:

.. code-block:: bash

    /opt/MegaRAID/MegaCli/libstorelibir-2.so.14.07-0
    /opt/MegaRAID/MegaCli/MegaCli
    /opt/MegaRAID/MegaCli/MegaCli64

If we mount the base image only, and take a look at the /opt directory, here is what
we'll see:

.. code-block:: bash

    /opt/downloads
    /opt/uploads

However, if we mount the base and the overlay as an overlayfs, then the /opt directory
will look like this:

.. code-block:: bash

    /opt/downloads
    /opt/uploads
    /opt/MegaRAID/MegaCli/libstorelibir-2.so.14.07-0
    /opt/MegaRAID/MegaCli/MegaCli
    /opt/MegaRAID/MegaCli/MegaCli64

In this example, the entire overlay is only about 6mb in size (the size of the
MegaRAID binaries). Due to this low footprint, it becomes much easier to make
incremental and quick changes to an image without having to rebuild the entire
thing, and also allows for many slight variations to be stored with very little
duplication across images, since all common files are located in a single base
image.

Base images
~~~~~~~~~~~~~~~~~~~~~~~~

We build almost all of our overlays off a single base image, named
base.trusty.3.13.0-32.squashfs.img. In 99% of use cases, using this image is
sufficient. A base filesystem is compressed into a
[squashfs](http://squashfs.sourceforge.net) image, which is a highly compressed,
read-only filesystem.

Building a base image requires a host machine running the same kernel as the
target kernel for the base image. We use [this script] (*URL to be provided*) to build the filesystem
and create a squashfs .

Creating overlayfs archives
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Simple Modifications**

Simple modifications are modifications that don't extend beyond copying files.
These may include adding scripts, files, directories, statically linked binaries, etc.

Simple modifications don't require mounting the base image. To create a simple
overlay like this, first create a directory:

.. code-block:: bash

    mkdir overlay
    cd overlay

Now add whatever files and directories you want to the overlay. Make sure to add
all required files as well (detailed below in the Required Files section).

To package up the overlay, make sure you are in the top level directory of the
overlay, and run:

.. code-block:: bash

    find . | cpio -H newc -o > ../overlay.cpio
    cd ..
    gzip -c overlay.cpio > ./overlay.cpio.gz


Now rename overlay.cpio.gz, and move it into the monorail server static files
directory in /opt/monorail/static/http. See :doc:`naming_conventions`
for recommendations on what to name the overlay and where to put it.

**Complex Modifications**

Complex modifications are modifications that require access to the OS filesystem
and make more widespread modifications to it. These may include building kernel
modules, installing packages with apt, etc. These modifications can be done only
on a Linux system. If you are building kernel modules, the linux system must also
be running the same kernel version as your base image and target kernel.

In order to make these changes, you must mount the base image along with an
overlay directory first, and run your commands within a chroot jail.

First, install squashfs tooling:

.. code-block:: bash

    sudo apt-get install squashfs-tools

Then, create a directory for your overlay files if it does not already exist:

.. code-block:: bash

    mkdir overlay


Now, create directories to be used as the mount point for the base image and the overlayfs:

.. code-block:: bash

    mkdir lower
    mkdir overlay_mount

Now, mount your filesystem:

.. code-block:: bash

    sudo mount -n -t squashfs -o loop <path to base image> lower
    sudo mount -t overlayfs overlayfs overlay_mount rw,upperdir=<path to overlay>,lowerdir=lower

If you are doing things like building kernel modules, you will need to bind
mount /dev, /proc and /sys:

.. code-block:: bash

    sudo chroot ./overlay_mount mount -t proc none /proc
    sudo chroot ./overlay_mount mount -t sysfs none /sys
    sudo mount --bind /dev ./overlay_mount/dev

Now, chroot into the filesystem:

.. code-block:: bash

    sudo chroot ./overlay_mount

From here, you should have a shell prompt using the root of the overlayfs as its
root. Some examples:

.. code-block:: bash

    sudo apt-get install <package name>
    sudo dpkg -i <path to a copied debian package>

Make sure to add all required files as well (detailed in the Required Files section below).

Finally, when you are done, exit the chroot and unmount everything:

.. code-block:: bash

    exit
    sudo umount ./overlay_mount/proc
    sudo umount ./overlay_mount/sys
    sudo umount ./overlay_mount/dev
    sudo umount overlay_mount
    sudo umount lower

All the modifications you made will be located in your overlay directory
(named overlay in this example). Package up your overlay directory using the below
commands. Depending on the file permissions of the changes you made, you may want
to run these commands as root.

.. code-block:: bash

    cd overlay
    # May need to run this as root
    find . | cpio -H newc -o > ../overlay.cpio
    cd ..
    gzip -c overlay.cpio > <name of zipped overlay> (see [naming conventions](LINK)).

**Required Files**

All overlays should contain the file located at /etc/rc.local, located [here](*URL to be provided*).
This file is necessary for the node to be able to communicate with the monorail
server in order to receive commands.


Modifying Overlayfs Archives
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The method of adding or remove files from an overlay is basically to decompress
the CPIO structure into a directory, modify what you need, and then recreate
another CPIO filesystem from that directory.

To make modifications to existing overlayfs archives, first un-zip and un-archive
the overlay (you may need to run these commands as root):

.. code-block:: bash

    mkdir overlay_src
    cd overlay_src
    gzip -dc <path to zipped overlay archive> | cpio -id


Now, follow the above Simple and Complex Modification sections above, but use
the un-zipped and un-archived overlay directory instead of a newly created
overlay directory.


**Examples**


#### creating the EMC custom overlay with test-eses

Below is the example script/process we used to create the custom overlay
for EMC with test_eses installed.

.. code-block:: bash

    # clean up the workspace
    rm -rf upper/ lower/ root_mount/

    # get the packages you want to install
    apt-get download libxml2 libxml2-dev sgml-base xml-core libxslt1.1

    mkdir upper lower root_overlay
    cd upper
    # In this case we are modifying the existing overlayfs_all_files overlay from the on-static-common package
    gunzip < ../overlayfs_all_files.cpio.gz | cpio -i
    cd ..
    sudo mount -n -t squashfs -o loop ~/base.trusty.3.13.0-32.squashfs.img lower
    sudo mount -t overlayfs overlayfs root_overlay -o rw,upperdir=upper,lowerdir=lower

    sudo chroot ./root_overlay mount -t proc none /proc
    sudo chroot ./root_overlay mount -t sysfs none /sys
    sudo mount --bind /dev ./root_overlay/dev

    sudo mv *.deb ./root_overlay
    sudo chroot ./root_overlay dpkg -i *.deb
    cd ~/emc_test_eses
    ln -s ../root_overlay
    sudo cp ./libtesteses.a ./root_overlay/usr/local/lib/
    sudo chmod 0644 ./root_overlay/usr/local/lib/libtesteses.a
    sudo cp ./libtesteses.la ./root_overlay/usr/local/lib/
    sudo chmod 0755 ./root_overlay/usr/local/lib/libtesteses.la
    sudo cp ./libtesteses.so.0.0.0 ./root_overlay/usr/local/lib/
    sudo chmod 0755 ./root_overlay/usr/local/lib/libtesteses.so.0.0.0
    sudo ln -s -f ./root_overlay/usr/local/lib/libtesteses.so.0.0.0 ./root_overlay/usr/local/lib/libtesteses.so
    sudo ln -s -f ./root_overlay/usr/local/lib/libtesteses.so.0.0.0 ./root_overlay/usr/local/lib/libtesteses.so.0
    sudo cp ./test_eses ./root_overlay/usr/local/bin/
    sudo chmod 0755 ./root_overlay/usr/local/bin/test_eses
    sudo mkdir -p ./root_overlay/usr/local/share/test_eses
    sudo cp ./test_eses.xsl ./root_overlay/usr/local/share/test_eses
    sudo chmod 0644 ./root_overlay/usr/local/share/test_eses/test_eses.xsl

    sudo umount ./root_overlay/proc
    sudo umount ./root_overlay/sys
    sudo umount ./root_overlay/dev
    sudo umount root_overlay
    sudo umount lower

    cd upper
    sudo find ./ | sudo cpio -H newc -o > ../overlay.cpio
    cd ..
    gzip -c ./overlay.cpio > overlayfs.trusty.emc.cpio.gz


The microkernel for tooling is a linux kernel and and a two-stage filesystem
that loads up with it.

The first stage is a standard initramfs that can be loaded by any PXE booting
system. `initrd.img-3.13.0-32-generic` is generated from an ubuntu system
running the kernel assocaited with it (3.13.0-32 in this case, represented by
the file `vmlinuz-3.13.0-32-generic`). The kernel itself has OverlayFS enabled
within it, and the initrd uses that to load a base (read-only) filesystem into
a RAM filesystem and then a single overlay filesystem (readwrite) over the top
of that. The base filesystem is created with debbootstrap and custom commands
to build up a "just enough OS" filesystem based on Ubuntu 14.04 (trusty).

- kernel: `vmlinuz-3.13.0-32-generic`
- initramfs: `initrd.img-3.13.0-32-generic`
- readonly base FS: `base.trusty.3.13.0-32.squashfs.img`

Overlays:

- debug overlay: `overlayfs_debug_files.trusty.cpio.gz`
- general overlay: `overlayfs_all_files.cpio.gz`

The overlay files are CPIO archives with additional "user-space" programs added
into them. The initramfs loads the base OS, and then overlays the CPIO archive,
and the resulting image contains common Linux tooling and immediately loads and
runs a Node.js task-runner that is built and rendered on the fly to the microkernel
to invoke commands on the remote machine as needed. This process is embedded
into the overlay itself, and relies on parameters passed into it through PXE
using `/proc/commandline` and the kernel parameters.

## tweaking the overlay

For instructions on how to create and modify overlays, see :doc:`creating_overlays`
