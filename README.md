Compile FreeNAS 9.2 with Xen support
====================================

This repository is based on the following two posts.

[http://forums.freenas.org/threads/how-i-got-a-xenhvm-kernal-and-xen-tools-working-in-freenas.15287/](http://forums.freenas.org/threads/how-i-got-a-xenhvm-kernal-and-xen-tools-working-in-freenas.15287/)
[http://mywiredhouse.net/blog/building-freenas-xen-pvhvm-support/](http://mywiredhouse.net/blog/building-freenas-xen-pvhvm-support/)

## Important Note:
In my instance I have not been able to successfully build FreeNAS 9.2 with Xen due to an checksum error **ataidle** while building. As of the above mentioned posts, it might not necessarily affect 
everyone. I would appreciate your feedback as to whether you encounter the same checksum error?

## Steps 

1. Create machine from scratch by installing FreeBSD 9.2 which has two disks. First disk for the OS (10 GB) and the second disk for the ZFS (20 GB)

2. Create a ZFS pool on the second disk
```$ echo 'zfs_enable="YES"' >> /etc/rc.conf
$ service zfs start
$ zpool create ZFSPool /dev/ada1```

3. Create a new directory on your ZFS volume
```$ mkdir /ZFSPool/myfreenasbuild```

4. Run the following commands to get a build environment
```$ portsnap fetch extract
$ cd /usr/ports/devel/git && make depends install
$ cd /usr/ports/sysutils/cdrtools  && make depends install
$ cd /usr/ports/archivers/pxz && make depends install```

5. Change into the directory on your ZFS volume
```$ cd mkdir /ZFSPool/myfreenasbuild```

6. Download the repository
```$ git clone -b xenserver https://github.com/topler/freenas-9.2-xen.git```

Note: This repository is a copy of the **9.2.0-RELEASE** branch with the changes outlined with the above posts.

7. Change into the repository
```$ cd freenas-9.2-xen```

8. Make external
```$ make git-external```

9. Build
```$ make release```

Note: If the build fails by getting a checksum error with **ataidle**, you can try to run the build without checksum (i.e. potential security risk) 

```$ make NO_CHECKSUM=yes release```

If the build was successful, you should have the *.iso located under */ZFSPool/myfreenasbuild/freenas-9.2-xen/release_stage/x64*.
You can follow the steps provided with [http://mywiredhouse.net/blog/building-freenas-xen-pvhvm-support/](http://mywiredhouse.net/blog/building-freenas-xen-pvhvm-support/) to install FreeNAS on 
Xenserver.

10. After the installation, you need to activate the Xen guest utilities by editing the */conf/base/etc/rc.conf* to include **xenguest_enable="YES"** underneath the *open-vm-tools* and setting the vmware 
tools to *NO*. After a reboot, you should be able to gracefully shutdown or suspend FreeNAS on XenServer along with all those other benefits.
