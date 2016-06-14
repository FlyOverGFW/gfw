官方教程
--
[custom-kernels-distros/run-a-distributionsupplied-kernel-with-pvgrub](https://www.linode.com/docs/tools-reference/custom-kernels-distros/run-a-distributionsupplied-kernel-with-pvgrub/)

过程记录(Centos6 and Newer)
--
我装的是Centos6.5
```
$ yum update -y
$ uname -a
Linux li1192-83.members.linode.com 4.1.5-x86_64-linode61 #7 SMP Mon Aug 24 13:46:31 EDT 2015 x86_64 x86_64 x86_64 GNU/Linux
$ yum install kernel.x86_64 -y #对于64位
```
>Create a file named `/boot/grub/menu.lst` with the following contents. Adjust the `title`, `kernel`, and `initrd` lines to reflect the actual file names found in the `/boot/` directory.

```
$ cd /boot && ls
$ mkdir /boot/grub
$ cd /boot/grub
$ touch menu.lst
$ vi menu.lst
```
根据个人实际情况，添加如下内容
```
timeout 1 #开机倒计时
title CentOS (2.6.32-573.3.1.el6.x86_64)
root (hd0)
kernel /boot/vmlinuz-2.6.32-573.3.1.el6.x86_64 root=/dev/xvda
initrd /boot/initramfs-2.6.32-573.3.1.el6.x86_64.img
```

- In the Linode Manager, edit your Linode’s configuration profile to use either `pv-grub-x86_32` or `pv-grub-x86_64` as the Kernel, depending on the version of CentOS you have deployed (32-bit or 64-bit).
- Make sure the root device is specified as `xvda`.
- Save your changes by clicking `Save Profile` at the bottom of the page.
- Reboot your Linode from the Dashboard tab.
- Once your Linode has rebooted, log in via SSH and issue the following command:
```
uname -a
```

