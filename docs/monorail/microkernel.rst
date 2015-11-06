MicroKernel and Overlays
----------------------------------------------------------

RackHD uses microkernal images and overlay filesystems to deploy operating systems on managed hardware.

The `on-imagebuilder`_ repository contains a set of ansible playbooks and roles used for building
debian-based linux microkernel images and overlay filesystems. They are primarily used with
the `on-taskgraph`_ workflow engine.

.. _on-imagebuilder: https://github.com/rackhd/on-imagebuilder
.. _on-taskgraph: https://github.com/rackhd/on-taskgraph

There are three ansible playbooks included, which build mountable
`squashfs`_ images, `overlay`_ filesystems, and initrd images.

.. _squashfs: https://en.wikipedia.org/wiki/SquashFS
.. _overlay: https://en.wikipedia.org/wiki/OverlayFS

Requirements
~~~~~~~~~~~~~~~~~~~~~~~~~~

- Any Debian/Ubuntu-based system (support for other distributions coming soon, however simply installing debootstrap and it should work).
- ansible is installed (`apt-get install ansible`)
- Internet access OR network access to an apt cache/proxy server from the build machine


Terms
~~~~~~~~~~~~~~~~~~~~~~~~~

.. list-table::
   :widths: 20 80
   :header-rows: 1

   * - Term
     - Description
   * - Base Image
     - This is a lightweight debian filesystem packaged up as a mountable squashfs image. Essentially, it's just a `debootstrap`_ minbase filesystem with some added configurations and packages. It is ~50mb squashed and occupies ~120mb of space when mounted. The base image[s] are used as a shared image that different overlays can be built and mounted with, and take 3-5 minutes to build.
   * - overlay filesystem
     - The overlay filesystem is a gzipped cpio archive of copy-on-write changes made to a mounted base image. Usually an overlay filesystem just contains a few packages and/or shell scripts, and is often under 10mb in size and takes under a minute to build.
   * - provisioner
     - An ansible role used to specify changes that should be made to the filesystem of an initrd, base image or overlay (e.g. extra packages, scripts, files).

.. _debootstrap: https://wiki.debian.org/Debootstrap




Bootstrap Process
~~~~~~~~~~~~~~~~~~~~~~~~~

The images produced by these playbooks are intended to be netbooted and run in RAM.
The typical flow for how these images are used/booted is this:

1. Netboot the kernel and initrd via PXE/iPXE.
2. The custom-built initrd runs a startup script (roles/initrd/provision_initrd/files/local) that requests a base squashfs image and an overlay filesystem from the boot server.
3. The initrd mounts both images together (union mount) into a tmpfs and boots into that as the root.

The basefs and initrd images are not intended to be changed very often. It's more likely
that one will add new provisioner roles to build custom overlays that can then be mounted
with the base image built by the existing ansible roles in this repository.



Building Images
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To build images, define an imagebuilding playbook (see example.yml for an example) or
use the default one (all.yml)::

    sudo ansible-playbook -i hosts <playbook.yml>


Adding Provisioner Roles and Configuration Files
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The provisioner role specifies how the filesystem of an initrd, base image, or overlay should be customized.

To add a new provisioner, do the following:

1. Make a new directory in `roles/<initrd|basefs|overlay>/tasks`, depending on the image type.
2. Create and edit a main.yml file in the above directory to do the tasks you want (see `ansible modules`_).
3. Add a new config_file into the vars directory. This will be included in the Ansible run as a set of top-level variables (via vars_files) to be included/used by tasks in the role.

   **NOTE:** The config_file must have as a bare minimum a provisioner variable that points to the role::

                      provisioner: roles/overlay/provision_discovery_overlay


4. Configure a new playbook (see example.yml) to run the appropriate wrapper playbook with the config_file, for example::

              - include: common/overlay_wrapper.yml
                 vars:
                   - config_file: vars/my_overlay.yml


The wrapper playbooks handle all the setup and cleanup required to run a provisioner, such as filesystem mounting and creation, and build file creation.

.. _ansible modules: http://docs.ansible.com/ansible/modules_intro.html


Changing the Global Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

All playbooks and roles depend on the variables defined in hosts and group_vars/on_imagebuild.
These variables specifies the location of the build roots and the apt server/package repositories that are used.

Changing the Build Root
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Update the paths in hosts to the desired build root. The build root paths must be updated in specified in both the *[overlay_build_chroot]* and the *[on_imagebuild:vars]* sections.


Changing the Repository URLs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


It is highly recommended that an apt-cacher-ng server be used rather than the upstream
archive.ubuntu.com server that is specified by default. Depending on the network connection,
this can reduce the build times for the basefs and initrd images by 50%.

To do this:

1. Run *apt-get install apt-cacher-ng*.
2. Edit the apt_server variable in group_vars/on_imagebuild to equal the address of your apt cache server, for example::

             apt_server: 192.168.100.5:3142


The first build will still be slow, because no packages are cached, but subsequent builds will be much faster.


**Note:** The playbooks can only be run LOCALLY -- not against remote hosts as
is usual with Ansible. This is because the chroot ansible_connection type
used for most builders and provisioners is not supported over ssh and
other remote ansible_connection types.


Why Not Containers?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The goal is to optimize for size on disk and modularity. By creating many different overlays that share a base image, we avoid data
duplication on the boot server (50mb base image + 10 * 5mb overlay archives vs. 10 * 55mb container images).

Additionally, it gives us flexibility to update the base image
and any system dependencies/scripts/etc. on it without having to rebuild
any overlays. For example, we use a custom rc.local script in the base image
that is used to receive commands from `workflows`_ on startup. Making
changes to this script should only have to be done in one place.

.. _workflows: https://github.com/rackhd/on-tasks

Please send us a note if you think this is incorrect! So log as our design contraints are preserved, we are more than open to leveraging
existing container technology.
