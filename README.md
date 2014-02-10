Compile FreeNAS 9.2 with Xen Support
====================================

This repository is a copy of the FreeNAS **9.2.0-RELEASE** branch with the changes outlined with the below two posts.

[http://forums.freenas.org/threads/how-i-got-a-xenhvm-kernal-and-xen-tools-working-in-freenas.15287/](http://forums.freenas.org/threads/how-i-got-a-xenhvm-kernal-and-xen-tools-working-in-freenas.15287/)
[http://mywiredhouse.net/blog/building-freenas-xen-pvhvm-support/](http://mywiredhouse.net/blog/building-freenas-xen-pvhvm-support/)

## Preface

Initially, I installed a FreeBSD 9.2 machine on my XenServer to perform the below steps but it somehow failed due to some wired errors with the "ataidle" port. Eventually I succeeded by using a FreeBSD 9.2 instance on Amazon EC2. In the light of this, I recommend using an Amazon EC2 instance (i.e. [FreeBSD on EC2](http://www.daemonology.net/freebsd-on-ec2/)) if you encounter wired build issues. Therefore the instructions below presume you are using EC2.

## Steps to Compile

1. Go to [http://www.daemonology.net/freebsd-on-ec2/](http://www.daemonology.net/freebsd-on-ec2/) and get a FreeBSD 9.2 64x macchine.
   * I recommend to use the instance size *m3.xlarge* as it has just enough memory so it won't use the swap.
   * Give this instance 10GB for the main disk.
   * Add an additional disk/ EBS volume of 20GB.
   * Generate a new *pem* file so it will be straight forward to connect to the instance.

2. On EC2 boot the FreeBSD 9.2 instance and connect via SSH by using the *pem* file.
   * Note: Amazon provides an SSH command to copy and paste. Make sure you change it to use the user name **ec2-user** rather than root! Otherwise you won't be able to connect.

3. Switch to root
   ```su root```

4. Check the */var/log/messages* file to figure out what device name the second disk has. In my instance it was *xbd5*
   ```tail -n 1000 /var/log/messages | grep "Virtual Block Device"```

5. Enable ZFS and create a ZFS pool
   ```echo 'zfs_enable="YES"' >> /etc/rc.conf```
   ```service zfs start```
   ```zpool create ZFSPool /dev/xbd5```
   ```cd /ZFSPool/```

6. Create a new directory on your ZFS volume
   ```mkdir /ZFSPool/myfreenasbuild```

7. To prevent that network problems botch your machine, while installing ports and more importantly while compiling, use *screen* (see details [here](https://forums.freebsd.org/viewtopic.php?&t=3599)).
   ```pkg_add -r screen```

8. Start *screen* and then hit enter. If your terminal connection drops now, you don't need to worry as it will not break the compilation.
   ```screen```

9. Run the following commands to create a build environment. Note: There are a few dependencies for this, but just accept the defaults as it goes and you’ll be right.
   ```portsnap fetch extract```
   ```pkg_add -r git```
   ```pkg_add -r cdrtools```
   ```pkg_add -r pxz```
   ```pkg_add -r screen```
   
10. Change into the build directory on your ZFS volume
   ```cd mkdir /ZFSPool/myfreenasbuild```

11. Download the repository
   ```git clone -b xenserver https://github.com/topler/freenas-9.2-xen.git```

12. Change into the downloaded directory
   ```cd freenas-9.2-xen```

13. Make external
   ```make git-external```

14. Build
   ```make release```

## Steps to Convert Image

If you used a *m3.xlarge* instance size, the entire process will take just over 4 hours.

1. If the build was successful, you should have the *.iso and *.img files located under */ZFSPool/myfreenasbuild/freenas-9.2-xen/release_stage/

2. Download the contents of the *release_stage* directory onto your local machine by using SCP

3. On your local machine, extract the *FreeNAS-9.2.0-RELEASE-xen-x64.img.xz* file

4. Convert the *img* to *raw*
   ```qemu-img convert -O raw FreeNAS-9.2.0-RELEASE-xen-x64.img FreeNAS-9.2.0-RELEASE-xen-x64.raw```

5. Use the *vhd-tool* to convert it to *vhd*. Note: Compiling *vhd-tool* is not straight forward. So if you have a Windows machine, simply use the *vhd-tool* provided by Microsoft (i.e. [vhdtool](http://archive.msdn.microsoft.com/vhdtool))

6. Import the resulting *vhd* file onto your XenServer.

## Known Issues

Due to some bugs with the FreeBSD xen-tools;
* Shutdown will only halt the machine rather than powering it off.
* Suspend will successfully suspend, but when resumed it causes the machine to reboot.
