# Create a container without Moby

## Before we start
This article is derived from [a great sharing](https://youtu.be/sK5i-N34im8) regarding the underlying secrets of the container world by a [Jérôme](https://github.com/jpetazzo) many years ago.  

In this article, we'll go through the demo Jérôme did at the end of that sharing. But

Many thanks to Mr. Jérôme Petazzoni.

## Environment Requirements
* A running linux host with kernel header version 5.10+.
  * [btrfs][btrfs-git-link] is a tool to create COW file systems. Since it's been [deprecated][btrfs-deprecation] from RHEL, the easy yum-install life is not easy anymore. If you want to install it via the source code. The kernel heder version requirement of it is 5.10+, and I believe that having your kernel version matched with the kernel header version is a good idea (maybe not, I don't know).
  * The linux host I picked is from [aws lightsail][aws-lightsail-link] with 4 GB memory, and linux distribution is the [CentOS][centos-link] (seems like RedHat heavily involved in the CNCF universe we live in, so I picked CentOS).
  * Upgrade a linux kernel is very easy. (All 100+ opened tabs on my web browser agreed with me unanimously) (just kidding, 99+ at most)
  
* [btrfs][btrfs-git-link]. If you decide to install it via the source code, make sure to follow the `INSTALL` markdown faile strictly.


## Step by Step Breakdown
1. `mount --make-rprivate /`
   > In short, this line will prevent our container file system from bleeding out.  

   > In depth, this is about [mount propagation][mount-propagation]. According to this [answer][rprivate-implication], mount propagation is default off for the kernel, but `systemd` automatically remounts all mount points as MS_SHARED on system startup so that nspawn and the container tools work out of the box. We turn it back off for our convenience.


[btrfs-deprecation]: https://news.ycombinator.com/item?id=14907771
[btrfs-git-link]: https://github.com/kdave/btrfs-progs/
[aws-lightsail-link]: https://aws.amazon.com/lightsail/
[centos-link]: https://www.centos.org/
[mount-propagation]: https://medium.com/kokster/kubernetes-mount-propagation-5306c36a4a2d
[rprivate-implication]: https://serverfault.com/questions/868682/implications-of-mount-make-private