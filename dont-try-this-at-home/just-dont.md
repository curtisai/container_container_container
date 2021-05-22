# Create a container without [Moby][moby-dock]

## Before we start
This article is derived from [a great sharing](https://youtu.be/sK5i-N34im8) regarding the underlying secrets of the container world by [Jérôme](https://github.com/jpetazzo) many years ago.  

In this article, we'll go through the demo Jérôme did at the end of that sharing.

Many thanks to Mr. Jérôme Petazzoni.

## Environment Requirements
* A running linux host with kernel header version 5.10+.
  * [btrfs][btrfs-git-link] is a tool to create [COW][cow-storage] file systems. Since it's been [deprecated][btrfs-deprecation] from RHEL, the easy yum-install life is not easy anymore. If you want to install it via the source code, kernel heder version 5.10+ is required, and I believe that having your kernel version matches with the kernel header version is a good idea (maybe not, I don't know).
  * The linux host I picked is from [aws lightsail][aws-lightsail-link] with 4 GB memory, and linux distribution is the [CentOS][centos-link] (seems like RedHat heavily involved in the CNCF timeline we live in, so I picked CentOS).
  * Upgrade the linux kernel is very easy. (All 100+ opened tabs on my web browser agreed with me unanimously) (just kidding, 99+ at most)
  
* [btrfs][btrfs-git-link]. If you decide to install it via the source code, make sure to follow the `INSTALL` markdown file strictly.
* [skopeo][skopeo-git] and [umoci][umoci-git]. `skopeo` is for image retrieval, coverting (from, and to [OCI][oci-webpage] image foramt). `umoci` is for image tarball unpacking. Without having a docker daemon running in our system, these two tools are our perfect alternatives.


## Step by Step Breakdown
1. `mount --make-rprivate /`
   > In short, this line will prevent our container file system from bleeding out.  

   > In depth, this is about [mount propagation][mount-propagation]. According to this [answer][rprivate-implication], mount propagation is default off for the kernel, but `systemd` automatically remounts all mount points as **MS_SHARED** on system startup so that `nspawn` and the container tools work out of the box. We turn it back off for our convenience. This change will be lost upon reboot.  
   
   PS. The code mentioned in that answer is moved to [here][systemd-mount-setup].

2. `mkdir -p images containers oci-images unpacked-images`
   > `skopeo` will retrive a docker image (`alpine` in this demo), convert & save it as oci format to the `oci-images` directory.
   > `umoci` will upack the oci image tarballs into `unpacked-images` directory. Then we will copy the `rootfs` of our unpacked image into `images` directory. As a result, the `images` directory actullay stores the flattened `rootfs` a container engine should see while starting a container.

3. `skopeo copy docker://alpine:latest oci:oci-images/alpine:latest`
   > This command will write the alpine image in oci format to the `oci-images` directory.
   > ![](assets/skopeo-copy-to-oci-image.png)

4. `sudo umoci unpack --image oci-images/alpine unpacked-images/alpine`
   > This command will unpack the oci image into two major parts: the `rootfs` and the `config.json`.
   > `rootfs` will be the fs of your container, and the `config.json` contains all magic specs your container engine needs to start the container.
   > Since we are doing the trickes today, the `rootfs` is all we need. But please make sure to take a loot at the `config.json` file, very imformative.
   > ![](assets/umoci-unpack-oci-image.png)

5. `btrfs subvol create images/alpine`
   > [COW][cow-storage] file system is crucial to containers. Jérôme's another [talk][docker-storage-driver-talk] explained it in depth.
   > This command creates a subvolume under `images/alpine`



[moby-dock]: https://www.docker.com/blog/docker-project-announces-open-source-a-thon-to-support-whale-and-marine-wildlife-conservation/
[btrfs-deprecation]: https://news.ycombinator.com/item?id=14907771
[btrfs-git-link]: https://github.com/kdave/btrfs-progs/
[cow-storage]: https://en.wikipedia.org/wiki/Copy-on-write#In_computer_storage
[aws-lightsail-link]: https://aws.amazon.com/lightsail/
[centos-link]: https://www.centos.org/
[skopeo-git]: https://github.com/containers/skopeo
[umoci-git]: https://github.com/opencontainers/umoci
[oci-webpage]: https://opencontainers.org/
[mount-propagation]: https://medium.com/kokster/kubernetes-mount-propagation-5306c36a4a2d
[rprivate-implication]: https://serverfault.com/questions/868682/implications-of-mount-make-private
[systemd-mount-setup]: https://github.com/systemd/systemd/blob/05576809194754989f88f83c7104341c35944546/src/shared/mount-setup.c#L528
[docker-storage-driver-talk]: https://youtu.be/9oh_M11-foU