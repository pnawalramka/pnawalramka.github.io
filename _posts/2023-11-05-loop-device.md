---
layout: post
title:  "Creating a Loop Device in Linux"
date:   2023-11-05
---
### Overview
If you have ever downloaded a new Linux distribution ISO image, you may have wondered how to view the content of the image prior to repartitioning your disk and installing the operating system onto your local disk. This can be done via a loop mount in Linux.

In Linux and other UNIX-like systems, it is possible to use a regular file as a block device. A loop device is a virtual or pseudo-device which enables a regular file to be accessed as a block device. Say you want to create a Linux file system but do not have a free disk partition available. In such a case, you can create a regular file on the disk and create a loop device using this file. The device node listing for the new pseudo-device can be seen under `/dev`. This loop device can then be used to create a new file system. The file system can be mounted, and its content can be accessed using normal file system APIs.



### Uses of Loop Device
As described above, one of the uses is creating a file system with a regular file when no disk partition is available.

Another common use of a loop device is with ISO images of installable operating systems. The content of an ISO image can be easily browsed by mounting the ISO image as a loop device.



### Creating a Loop Device in Linux
These commands require root privilege.

1. Create a large regular file on disk that will be used to create the loop device.

```
# dd if=/dev/zero of=/loopfile bs=1024 count=51200

51200+0 records in

51200+0 records out

52428800 bytes (52 MB, 50 MiB) copied, 0.114882 s, 456 MB/s
```

This command creates a 50Mb file called loopfile filled with zeros.

If you already have an image file that you want to mount as a loop device, then you can skip this step.


2. Create a loop device with the large file created above.

There may be some loop devices already created. Run the following command to find the first available device node.

```
# losetup -f

/dev/loop1
```

So we can safely use /dev/loop1 to create our loop device. Create the loop device with the following command.

```
# losetup /dev/loop1 /loopfile
```

If you see no errors, the regular file `/loopfile` is now associated with the loop device `/dev/loop1`.


3. Confirm creation of the loop device

```
# losetup /dev/loop1

/dev/loop1: [66309]:214 (/loopfile)
```



### Creating a Linux Filesystem With the Loop Device
You can now create a normal Linux filesystem with this loop device.

1. Create an ext4 filesystem using /dev/loop1.

```
# mkfs -t ext4 -v /dev/loop1

mke2fs 1.45.3 (14-Jul-2019)

fs_types for mke2fs.conf resolution: 'ext4', 'small'

Discarding device blocks: done                            

Filesystem label=

OS type: Linux

Block size=4096 (log=2)

Fragment size=4096 (log=2)

Stride=0 blocks, Stripe width=0 blocks

12800 inodes, 12800 blocks

640 blocks (5.00%) reserved for the super user

First data block=0

Maximum filesystem blocks=14680064

1 block group

32768 blocks per group, 32768 fragments per group

12800 inodes per group



Allocating group tables: done                            

Writing inode tables: done                            

Creating journal (1024 blocks): done

Writing superblocks and filesystem accounting information: done
```


2. Create a mount point for the filesystem.

```
# mkdir /mnt/loopfs
```


3. Mount the newly created filesystem.

```
# mount -t ext4 /dev/loop1 /mnt/loopfs
```
This command mounts the loop device as a normal Linux ext4 filesystem, on which normal filesystem operations can be performed.


4. Check disk usage of the file system.

```
# df -h /dev/loop1

Filesystem      Size  Used Avail Use% Mounted on

/dev/loop1       45M   48K   41M   1% /mnt/loopfs
```

5. Use tune2fs to see the filesystem settings.

```
#  tune2fs -l /dev/loop1

tune2fs 1.45.3 (14-Jul-2019)

Filesystem volume name:   <none>

Last mounted on:          <not available>

Filesystem UUID:          b1b13d6e-c544-45dd-a549-5846371fbde6

Filesystem magic number:  0xEF53

Filesystem revision #:    1 (dynamic)

Filesystem features:      has_journal ext_attr resize_inode dir_index filetype needs_recovery extent 64bit flex_bg sparse_super large_file huge_file dir_nlink extra_isize metadata_csum

Filesystem flags:         signed_directory_hash 

Default mount options:    user_xattr acl

Filesystem state:         clean

Errors behavior:          Continue

Filesystem OS type:       Linux

Inode count:              12800

Block count:              12800

Reserved block count:     640

Free blocks:              11360

Free inodes:              12789

First block:              0

Block size:               4096

Fragment size:            4096

Group descriptor size:    64

Reserved GDT blocks:      6

Blocks per group:         32768

Fragments per group:      32768

Inodes per group:         12800

Inode blocks per group:   400

Flex block group size:    16

Filesystem created:       Sun Mar 19 08:56:47 2023

Last mount time:          Sun Mar 19 09:00:52 2023

Last write time:          Sun Mar 19 09:00:52 2023

Mount count:              1

Maximum mount count:      -1

Last checked:             Sun Mar 19 08:56:47 2023

Check interval:           0 (<none>)

Lifetime writes:          37 kB

Reserved blocks uid:      0 (user root)

Reserved blocks gid:      0 (group root)

First inode:              11

Inode size:              128

Journal inode:            8

Default directory hash:   half_md4

Directory Hash Seed:      e489fd33-4003-4235-9347-144c7a5d4d73

Journal backup:           inode blocks

Checksum type:            crc32c

Checksum:                 0x3b8c797a
```


6. To unmount the filesystem and delete the loop device, run the following commands.

```
# umount /mnt/loopfs/

# losetup -d /dev/loop1
```



*This article was initially published [here](https://dzone.com/articles/loop-device-in-linux).*
