---
layout     : post
title      : "Docker - LFS Build"
date       : 2017-08-30 13:15:35 +0700
categories : system
tags       : [docker, distro, package manager, lfs]
keywords   : [lfs]
author     : epsi
toc        : toc/2017/08/topics-docker.html

opengraph:
  image: /assets/site/images/topics/docker.png

excerpt:
  Docker flow for LFS package installation,
  first time using LFS experience.
  From build (configure, make, and make install),
  to solving dependency.

related_link_ids: 
  - 17083145  # Docker Summary
  - 17083015  # LFS Build
  - 17082715  # Arch ALPM
  - 17082415  # Debian Portage
  - 17082115  # Slackware Package
  - 17081815  # Fedora DNF
  - 17081515  # openSUSE Zypper
  - 17081315  # Void XBPS
  - 17081145  # Gentoo Portage
  - 17081015  # Crux Ports
---

{% include toc/2017/08/toc-docker-lfs-build.html %}

-- -- --

### Preface

> Goal: Examine Package Manager, Focus on Command Line Interface

	You jump, I jump !

LFS (Linux From Scratch) is once scary for me when I was a beginner,
but do not let the fear intimidate you.
Once you jump to LFS and swim,
you will know that the myth is not true at all.

This LFS is even easier with docker container,
avoid daunting task such as setting up system.
Therefore we can dive right away to compilation.

Here is I share my LFS experience.
Of course this won't we a full blown experience,
since I have decided to postpone LFS learning.

#### Postpone

Unfortunately, I failed in some process,
and I do not even know, whether it is my stupidity as usual,
or by lack of docker limitation knowledge
I have rarely give up,
but I must admit I'm running out of time.

All I can say is, I have to pending this LFS experience,
and continue LFS in chroot environment.
I will write article later about this LFS.

#### Reading

*	<http://www.linuxfromscratch.org/lfs/view/stable/>

{% include toc/2017/08/docker-test-bed.md %}

-- -- --

### Getting Started With Docker

As usual, first, we do attach docker process.

{% highlight bash %}
$ docker pull kevinleptons/lfs-auto
{% endhighlight %}

{% highlight bash %}
$ docker image list 
{% raw %}
  --format 'table {{.Repository}}\t{{.Size}}'
{% endraw %}
REPOSITORY              SIZE
gentoo/stage3-amd64     873MB
vbatts/slackware        86.7MB
voidlinux/voidlinux     202MB
kevinleptons/lfs-auto   753MB
{% endhighlight %}

{% highlight bash %}
$ docker run -it voidlinux/voidlinux bash
bash-4.4# exit
{% endhighlight %}

{% highlight bash %}
$ docker ps -a 
{% raw %}
  --format 'table {{.Image}}\t{{.Names}}\t{{.Status}}'
{% endraw %}
IMAGE                   NAMES               STATUS
kevinleptons/lfs-auto   wonderful_nobel     Exited (127) 31 minutes ago
voidlinux/voidlinux     awesome_davinci     Exited (0) 12 hours ago
gentoo/stage3-amd64     amazing_shirley     Exited (0) 36 hours ago
vbatts/slackware        cranky_keller       Exited (0) 38 hours ago
{% endhighlight %}

{% highlight bash %}
$ docker start wonderful_nobel
wonderful_nobel
{% endhighlight %}

{% highlight bash %}
$ docker attach wonderful_nobel
bash-4.4#
{% endhighlight %}

![Docker LFS: Getting Started][image-ss-lfs-docker]{: .img-responsive }

-- -- --

### No Package Management

LFS use no Package Manager.
You have to download manually and compile by yourself,
sometimes with feeling of being stupid in the corner.
But hey... you are going to feel closer to each package,
and feel the intimacy too, which is good for learning

-- -- --

### Requirement Check

Using guidance from LFS chapter 2.

*	<http://www.linuxfromscratch.org/lfs/view/stable/chapter02/hostreqs.html>

{% highlight bash %}
$ bash version-check.sh
{% endhighlight %}

![Docker LFS: Requirement Check][image-ss-lfs-version]{: .img-responsive }

-- -- --

### No Dependency Resolver

The fact that LFS has no package management,
means you need to compile all the packages directly from source.
Most compilation error that I experience comes from dependency issue.
Most compilation works well after I address the dependency issue.

I was once trying to compile <code>nano</code>
on my first jump to LFS in docker, and failed miserably. 
Then I realize <code>nano</code> needs <code>ncurses</code>.
My second attempt is <code>less</code>, as I predict, 
<code>less</code> does not need any dependency.
My third attempt is compile <code>man-db</code>.
Unfortunately <code>man-db</code> required a few package as dependency.
But after all, the process has been completely done.

Here is the steps, by trial and error.

*	install ncurses: succeed

*	install less: succeed

*	install man:<br/>
	configure depend on pkg-config<br/>
	configure depend on libpipeline<br/>
	configure depend on gdbm<br/>
	make depend on groff<br/>
	make using groff A4 failed<br/>
	make using groff letter succeed<br/>

*	install pkg-config: succeed

*	install libpipeline: succeed

*	install gdbm: succeed

*	install groff: succeed

*	install man: succeed

Nomore fear now. I realized that I can do it by examining each error.
In fact, I can cheat, by using other distro to identify the dependency.

-- -- --

### Example 1: less

<code>less</code> is my first successful install.
<code>less</code> is also an easy example.
Just follow the procedure, and you are done.

Reading:

*	<http://www.linuxfromscratch.org/lfs/view/development/chapter06/less.html>

Source:

*	<http://www.greenwoodsoftware.com/less/less-487.tar.gz>

#### Detail

{% highlight bash %}
$ cd ~

$ wget -c http://www.greenwoodsoftware.com/less/less-487.tar.gz

$ tar -xvf less-487.tar.gz

$ cd less-487
{% endhighlight %}

{% highlight bash %}
$ ./configure --prefix=/usr --sysconfdir=/etc
...
regular expression library:  posix
configure: creating ./config.status
config.status: creating Makefile
config.status: creating defines.h
{% endhighlight %}

{% highlight bash %}
$ make
...
gcc -I. -c -DBINDIR=\"/usr/bin\" -DSYSDIR=\"/etc\"  -g -O2 lesskey.c
gcc  -o lesskey lesskey.o version.o
gcc -I. -c -DBINDIR=\"/usr/bin\" -DSYSDIR=\"/etc\"  -g -O2 lessecho.c
gcc  -o lessecho lessecho.o version.o
{% endhighlight %}

{% highlight bash %}
$ make install
./mkinstalldirs /usr/bin /usr/share/man/man1
/usr/bin/install -c less /usr/bin/less
/usr/bin/install -c lesskey /usr/bin/lesskey
/usr/bin/install -c lessecho /usr/bin/lessecho
/usr/bin/install -c -m 644 ./less.nro /usr/share/man/man1/less.1
/usr/bin/install -c -m 644 ./lesskey.nro /usr/share/man/man1/lesskey.1
/usr/bin/install -c -m 644 ./lessecho.nro /usr/share/man/man1/lessecho.1
{% endhighlight %}

![Docker LFS: less: make install][image-ss-lfs-less-install]{: .img-responsive }

Commonly the steps are 

*	<code>./configure</code>, mostly with a few parameter.

*	<code>make</code>, sometimes with parameter.

*	maybe require <code>make check</code>.

*	and finally <code>make install</code>.

-- -- --

### Example 2: man-db

#### Dependency

As mentioned, <code>man-db</code> depend on a few packages.
This package should be installed first:
<code>pkg-config</code>,
<code>libpipeline</code>,
<code>gdbm</code>,
<code>groff</code>,
<code>man</code>.

Basic package dependency can be found here.

*	<http://www.linuxfromscratch.org/lfs/view/stable/chapter03/packages.html>

#### Detail

Reading:

*	<http://www.linuxfromscratch.org/lfs/view/development/chapter06/man-db.html>

Source:

*	<http://download.savannah.gnu.org/releases/man-db/man-db-2.7.6.1.tar.xz>

{% highlight bash %}
$ cd ~

$ wget -c http://download.savannah.gnu.org/releases/man-db/man-db-2.7.6.1.tar.xz

$ tar -xvf man-db-2.7.6.1.tar.xz

$ cd man-db-2.7.6.1
{% endhighlight %}

This is what it looks like, for beginer who never compile anything.

{% highlight bash %}
$ ./configure --prefix=/usr                        \
            --docdir=/usr/share/doc/man-db-2.7.6.1 \
            --sysconfdir=/etc                    \
            --disable-setuid                     \
            --enable-cache-owner=bin             \
            --with-browser=/usr/bin/lynx         \
            --with-vgrind=/usr/bin/vgrind        \
            --with-grap=/usr/bin/grap            \
            --with-systemdtmpfilesdir=
...
configure: creating ./config.status
config.status: creating Makefile
...
config.status: creating po/POTFILES
config.status: creating po/Makefile
config.status: creating gnulib/po/POTFILES
config.status: creating gnulib/po/Makefile
{% endhighlight %}

[![Docker LFS: man: configure][image-ss-lfs-man-conf]{: .img-responsive }][photo-ss-lfs-man-conf]

{% highlight bash %}
$ make 
...
Making all in tools
make[2]: Entering directory '/root/man-db-2.7.6.1/tools'
make[2]: Nothing to be done for 'all'.
make[2]: Leaving directory '/root/man-db-2.7.6.1/tools'
make[2]: Entering directory '/root/man-db-2.7.6.1'
make[2]: Leaving directory '/root/man-db-2.7.6.1'
make[1]: Leaving directory '/root/man-db-2.7.6.1'
{% endhighlight %}

{% highlight bash %}
$ make check
...
Making check in tools
make[1]: Entering directory '/root/man-db-2.7.6.1/tools'
make[1]: Nothing to be done for 'check'.
make[1]: Leaving directory '/root/man-db-2.7.6.1/tools'
make[1]: Entering directory '/root/man-db-2.7.6.1'
make[1]: Leaving directory '/root/man-db-2.7.6.1'
{% endhighlight %}

[![Docker LFS: man: make][image-ss-lfs-man-make]{: .img-responsive }][photo-ss-lfs-man-make]

{% highlight bash %}
$ make install
...
Making install in tools
make[1]: Entering directory '/root/man-db-2.7.6.1/tools'
make[2]: Entering directory '/root/man-db-2.7.6.1/tools'
make[2]: Nothing to be done for 'install-exec-am'.
make[2]: Nothing to be done for 'install-data-am'.
make[2]: Leaving directory '/root/man-db-2.7.6.1/tools'
make[1]: Leaving directory '/root/man-db-2.7.6.1/tools'
make[1]: Entering directory '/root/man-db-2.7.6.1'
make[2]: Entering directory '/root/man-db-2.7.6.1'
make[2]: Nothing to be done for 'install-exec-am'.
make[2]: Nothing to be done for 'install-data-am'.
make[2]: Leaving directory '/root/man-db-2.7.6.1'
make[1]: Leaving directory '/root/man-db-2.7.6.1'
{% endhighlight %}

[![Docker LFS: man: install][image-ss-lfs-man-install]{: .img-responsive }][photo-ss-lfs-man-install]

-- -- --

### Resolving Dependency

Since I use Docker in not an official way to practice LFS,
I also cheat using the host. I find a way to resolve dependency.
You can use <code>pactree</code> using arch based distribution (ALPM).
Your package manager, might have similar tool.


Here is my screenshot using Artix Linux,
the host of my docker.

[![Docker LFS: man: install][image-ss-artix-pactree]{: .img-responsive }][photo-ss-artix-pactree]

With Tree

*	<code>pactree</code>

*	<code>rpmreaper</code>

*	<code>prt-get deptree</code>

*	<code>slpkg deps-status --tree</code>

Without Tree

*	<code>dnf repoquery --requires man-db</code>

*	<code>zypper info --requires man</code>

*	<code>pacman -[S|Q]i man-db</code>

*	<code>xbps-query -R -x man-db</code>

*	<code>emerge -ep man-db</code>

*	<code>apt-cache showpkg man-db</code>

*	<code>prt-get depends</code>

Next time, I'll check the dependency first before building my LFS package.

-- -- --

### Conclusion

	It has been an honour, being (almost) a LFS user.

This is just building LFS at a glance.
Of course there are more than my limited knowledge.
There is also more advance topic such as BLFS, ALFS, and CLFS.

Thank you for reading

[//]: <> ( -- -- -- links below -- -- -- )

{% assign system_path = 'https://epsi-rns.github.io/assets-system' %}
{% assign asset_post  = system_path | append: '/2017/08/docker-lfs' %}

[image-ss-lfs-docker]:       {{ asset_post }}/00-getting-started.png
[image-ss-lfs-version]:      {{ asset_post }}/01-bash-version.png

[image-ss-lfs-less-install]: {{ asset_post }}/02-less-make-install.png

[image-ss-lfs-man-conf]:    {{ asset_post }}/02-man-configure-half.png
[photo-ss-lfs-man-conf]:    https://photos.google.com/share/AF1QipMO53TtSJVXrkn8R0s4wre4QWgX7_G5CoaSkFMneVHFp9Tu5STBmdjW3M3fpA2eEw/photo/AF1QipNAY08WmXWjBas2LKuL7Youj7oaKUYOiG0QLlYe?key=WGIySDVOaVpibkJCRkV5NWVZUUs3UnNLNHR1MVpn

[image-ss-lfs-man-make]:    {{ asset_post }}/02-man-make-half.png
[photo-ss-lfs-man-make]:    https://photos.google.com/share/AF1QipMO53TtSJVXrkn8R0s4wre4QWgX7_G5CoaSkFMneVHFp9Tu5STBmdjW3M3fpA2eEw/photo/AF1QipMMyskHqXSsNX-mEdciEbqmBEML1Xvu64H1ZIY2?key=WGIySDVOaVpibkJCRkV5NWVZUUs3UnNLNHR1MVpn

[image-ss-lfs-man-install]: {{ asset_post }}/02-man-make-install-half.png
[photo-ss-lfs-man-install]: https://photos.google.com/share/AF1QipMO53TtSJVXrkn8R0s4wre4QWgX7_G5CoaSkFMneVHFp9Tu5STBmdjW3M3fpA2eEw/photo/AF1QipP0Z6AvDlEG_MdmlNzCzpUt86uyKTQ7gGnZSBNU?key=WGIySDVOaVpibkJCRkV5NWVZUUs3UnNLNHR1MVpn

[image-ss-artix-pactree]:   {{ asset_post }}/artix-pactree-man.png
[photo-ss-artix-pactree]:   https://photos.google.com/share/AF1QipMO53TtSJVXrkn8R0s4wre4QWgX7_G5CoaSkFMneVHFp9Tu5STBmdjW3M3fpA2eEw/photo/AF1QipPYsJbSP0SyZEAebz4NPfNMDdKb6BHnv7ZbL6UD?key=WGIySDVOaVpibkJCRkV5NWVZUUs3UnNLNHR1MVpn
