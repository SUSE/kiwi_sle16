.. _grub_boot_from_iso:

Booting a Live ISO Images from Grub2
====================================

.. sidebar:: Abstract

   This page provides further information for handling
   ISO images built with {kiwi} and references the following
   articles:

   * :ref:`hybrid_iso`

In {kiwi}, all generated ISO images are created to be hybrid. This means,
the image can be used as a CD/DVD or as a disk. This works because
the ISO image also has a partition table embedded. With more and more
computers delivered without a CD/DVD drive this becomes important.
The deployment of such an image onto a disk like an USB stick normally 
destroys all existing data on this device. It is also not possible to use 
USB stick as a data storage device. Most USB sticks are pre-formatted 
with a FAT32 or exFAT Windows File System and to keep the existing data 
on the stick untouched a different deployment needs to be used.

Fortunately Grub2 supports booting directly from ISO files. It does not matter 
whether it is installed on your computer's hard drive or on a USB stick.
The following deployment process copies the ISO image as an additional file 
to the USB stick or hard drive. The ability to boot from the disk is configured 
through a Grub2 feature which allows to loopback mount an ISO file and boot the
kernel and initrd directly from the ISO file.

The initrd loaded in this process must also be able to loopback
mount the ISO file to access the root filesystem and boot the
live system. Almost every Linux distribution supports fat32, 
and more and more of them also support exFAT. For hard drives, 
Linux filesystems are also supported.

The dracut initrd system used by {kiwi} provides this
feature upstream called as "iso-scan/filename". Therefore all {kiwi} generated
live ISO images supports this deployment mode.

The following procedure shows how to setup Grub2 on your hard drive:

1. Copy the ISO image to a folder of your choice on your hard drive.

2. Add the following code to the "grub.cfg" file:

   Be sure to set the path to the ISO image, you can set your own menu name.
   The drive identification for Grub2 can be checked at boot time 
   by pressing the 'c' key and typing 'ls'.

   .. code:: bash

      submenu "Boot from openSUSE ISO" {
	   iso_path="(hd0,gpt2)/path/to/openSUSE.iso"
	   export iso_path
	   loopback loop "$iso_path"
	   root=(loop)
	   source /boot/grub2/loopback.cfg
	   loopback --delete loop
      }

3. Restart your computer and select the added menu.

For USB sticks, the procedure is identical. You would then install 
Grub2 on the USB drive and follow the steps above. 
The use of scripts such as "MultiOS-USB" is strongly recommended.

Booting Install ISO Images from Grub2
-------------------------------------

The install ISO image created for an OEM disk image via the
`installiso="true"` attribute supports the "iso-scan/filename"
feature as well. The procedure is the same as for live ISO images,
only the boot options differ. The following example assumes the
default install ISO volume ID `INSTALL` and the ISO file stored
as :file:`/iso/some-kiwi.install.iso` on the root filesystem of the
current operating system. Replace `UUID` with the UUID of this
filesystem, which can be looked up by running
`findmnt -n -o UUID /`:

.. code:: bash

   menuentry "Install from ISO" {
       load_video
       set gfxpayload=keep
       insmod gzio
       insmod part_gpt
       insmod iso9660
       insmod btrfs
       insmod ext2
       insmod xfs
       insmod lvm
       insmod loopback
       search --no-floppy --fs-uuid --set=root UUID
       set isofile="/iso/some-kiwi.install.iso"
       set kernel="(loop)/boot/x86_64/loader/linux"
       set initrd="(loop)/boot/x86_64/loader/initrd"
       set options="cdinst=1 root=install:CDLABEL=INSTALL"
       loopback loop $isofile
       linux $kernel iso-scan/filename=$isofile $options
       initrd $initrd
   }

Alternatively, the grub configuration of the ISO image itself can be
used through the `loopback.cfg` file provided on all {kiwi} ISO images.
The ISO path must be passed via the `iso_path` variable. Depending
on the distribution, the grub directory on the ISO image is named
`grub2` or `grub`:

.. code:: bash

   menuentry "Install from ISO" {
       insmod loopback
       search --no-floppy --fs-uuid --set=root UUID
       set iso_path="/iso/some-kiwi.install.iso"
       export iso_path
       loopback loop $iso_path
       configfile (loop)/boot/grub2/loopback.cfg
   }

.. note:: ISO File on the Install Target Disk

   If the disk selected as installation target holds the ISO file,
   the installation would overwrite the ISO file while reading from
   it. In this case {kiwi} copies the ISO file into main memory and
   releases the disk before the image is dumped. Make sure the system
   has enough main memory to store the ISO file plus an additional
   512 MB. If there is not enough memory available, the installation
   stops before the target disk is modified.
