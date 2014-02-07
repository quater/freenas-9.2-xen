Compile FreeNAS 9.2 with Xen support
====================================

This repository is based on the following two posts.

[http://forums.freenas.org/threads/how-i-got-a-xenhvm-kernal-and-xen-tools-working-in-freenas.15287/](http://forums.freenas.org/threads/how-i-got-a-xenhvm-kernal-and-xen-tools-working-in-freenas.15287/)
[http://mywiredhouse.net/blog/building-freenas-xen-pvhvm-support/](http://mywiredhouse.net/blog/building-freenas-xen-pvhvm-support/)

## Important Note:

*This repository is a copy of the **9.2.0-RELEASE** FreeNAS branch with the changes outlined with the above posts.
*In my instance I have not been able to successfully build FreeNAS 9.2 with Xen tools due to an checksum error with the **ataidle** package while building. 
I believe this is due to a problem with my build environment rather due to the Xen related changes present in this repository, thus should not affect 
everyone.

## Steps 

1. Create machine from scratch by installing FreeBSD 9.2 which has two disks. First disk for the OS (10 GB) and the second disk for the ZFS (20 GB)

2. Create a ZFS pool on the second disk

```$ echo 'zfs_enable="YES"' >> /etc/rc.conf```

```$ service zfs start```

```$ zpool create ZFSPool /dev/ada1```

3. Create a new directory on your ZFS volume

```$ mkdir /ZFSPool/myfreenasbuild```

4. Run the following commands to get a build environment

```$ portsnap fetch extract```

```$ cd /usr/ports/devel/git && make depends install```

```$ cd /usr/ports/sysutils/cdrtools  && make depends install```

```$ cd /usr/ports/archivers/pxz && make depends install```

5. Change into the directory on your ZFS volume

```$ cd mkdir /ZFSPool/myfreenasbuild```

6. Download the repository

```$ git clone -b xenserver https://github.com/topler/freenas-9.2-xen.git```

7. Change into the repository

```$ cd freenas-9.2-xen```

8. Make external

```$ make git-external```

9. Build

```$ make release```

Depending on your environment, this build may take several hours to complete. If the build fails due to getting a checksum error with the **ataidle** 
package, you can try to run the build without checksum (i.e. potential security risk).

```$ make NO_CHECKSUM=yes release```

If the build was successful, you should have the *.iso located under */ZFSPool/myfreenasbuild/freenas-9.2-xen/release_stage/x64*.
You can then follow the steps provided with 
[http://mywiredhouse.net/blog/building-freenas-xen-pvhvm-support/](http://mywiredhouse.net/blog/building-freenas-xen-pvhvm-support/) to install FreeNAS on 
Xenserver.
