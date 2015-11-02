MicroKernel and Overlays
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. include:: monorail/creating_overlays.rst

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
