---
layout: post
title: Mount root as tmpfs
---
Originally written on 23/10/2009, i'm posting it again since some external links were pointing to it.

So you just got a brand new laptop with an insane chunk of ram, like 4GB or so and you are afraid it won't be ever used at all... You can take advantage of it to have your debian linux box run insanely fast. This process can be easily ported to distributions other than mine but this won't be covered. Let's begin.
Our aim is to load your whole root partition into ram, so every binary, shared library, tmp file and so will be instantly accessed. My advice is to keep `/` and `/home` on different partitions, so you can still save your files and configs with little access to the disk.

Boot into your system and clean up as much as you can, because every useless MB you leave on `/` means less memory available later.
    apt-get clean
    rm -R /usr/src/*
    ...

Here comes the magic: backup the script "/usr/share/initramfs-tools/scripts/local" and hack it to mount root as tmpfs, here's my patch file:

    --- local_orig  2009-03-13 08:46:21.000000000 +0100
    +++ local_rikiji        2009-03-13 08:58:10.000000000 +0100
    @@ -117,8 +117,17 @@
     
            # FIXME This has no error checking
            # Mount root
    -       mount ${roflag} -t ${FSTYPE} ${ROOTFLAGS} ${ROOT} ${rootmnt}
    -
    +       # mount ${roflag} -t ${FSTYPE} ${ROOTFLAGS} ${ROOT} ${rootmnt}
    +       TMPRAMROOT="/ramroot"
    +       mkdir $TMPRAMROOT
    +       mount ${roflag} -t ${FSTYPE} ${ROOTFLAGS} ${ROOT} $TMPRAMROOT
    +       mount -t tmpfs none ${rootmnt}
    +       cd ${rootmnt}
    +       echo "Unpacking root into ram..."
    +       tar -xf "$TMPRAMROOT/packedroot.tar"
    +       echo "Done."
    +       umount $TMPRAMROOT
    +      
            [ "$quiet" != "y" ] && log_begin_msg "Running /scripts/local-bottom"
            run_scripts /scripts/local-bottom
            [ "$quiet" != "y" ] && log_end_msg

Then it should appear like this:

    # Comment out the usual roofs mount
    # mount ${roflag} -t ${FSTYPE} ${ROOTFLAGS} ${ROOT} ${rootmnt}
    TMPRAMROOT="/ramroot"
    mkdir $TMPRAMROOT
    mount ${roflag} -t ${FSTYPE} ${ROOTFLAGS} ${ROOT} $TMPRAMROOT
    mount -t tmpfs none ${rootmnt}
    cd ${rootmnt}
    echo "Unpacking root into ram..."
    tar -xf "$TMPRAMROOT/packedroot.tar"
    echo "Done."
    umount $TMPRAMROOT

Let's build the `initrd` image: 

    mkinitramfs -o /boot/initrd.img-`uname -r`-ramroot

and add an adequate bootloader entry, this is mine for grub

    title RamRoot fscking fast Debian GNU/Linux, kernel 2.6.26-1-686-bigmem
    root (hd0,1)
    kernel /boot/vmlinuz-2.6.26-1-686-bigmem root=/dev/sda2 ro quiet
    initrd /boot/initrd.img-2.6.26-1-686-bigmem-ramroot

Now fire up you favourite live CD and pack your root. I avoided gzipping it to speedup unpacking process (that happens every boot). Something like:

    tar -cf /mnt/sda3/packedroot.tar /mnt/sda2/*
    mv /mnt/sda3/packedroot.tar /mnt/sda2/

And you're done! Reboot and enjoy your amazingly fast system!
