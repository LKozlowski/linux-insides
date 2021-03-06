Linux kernel development
================================================================================

Introduction
--------------------------------------------------------------------------------

As you already may know, I've started a series of [blog posts](http://0xax.github.io/categories/assembly/) about assembler programming for `x86_64` architecture in the last year. I hadn't written even one line of low-level code before this moment, of course except a couple of toy `Hello World` examples in the university. It was a long time ago and, as I already said, I didn't write low-level code at all. Some time ago I became interested in such things, or in other words, I understood that I can write programs, but actually I didn't understand how my program is arranged.

After writing some assembler code I began to understand how my program looks after compilation, **approximately**. But I didn't understand many different things. For example: what occurs when the `syscall` instruction is executed in my assembler, what occurs when the `printf` function starts to work, how does my program talk with other computers via a network and many many other cases. [Assembler](https://en.wikipedia.org/wiki/Assembly_language#Assembler) programming language didn't give me the answers to my questions, and I decided to go deeper in my research. I started to learn the source code of the Linux kernel and tried to understand things that interest me. The source code of the Linux kernel didn't give me answers on **all** of my questions, but now my knowledge about the Linux kernel and processes around it is much better.

I'm writing this part nine and a half months after I started learning the source code of the Linux kernel and published the first [part](https://0xax.gitbooks.io/linux-insides/content/Booting/linux-bootstrap-1.html) of this book. It now contains forty parts, and that is not the end. I decided to write this series about the Linux kernel mostly for myself. As you know, the Linux kernel is very huge piece of code and it is very easy to forget what it does, what this or that part of the Linux kernel means, and how it implements things. But soon, the [linux-insides](https://github.com/0xAX/linux-insides) repo became popular and after nine months it has `9096` stars:

![github](http://s2.postimg.org/jjb3s4frt/stars.png)

Yeah, seems that people are interested in the internals of the Linux kernel. Besides this, in all that time that I'm writing `linux-inside`, I have received many questions from different people like: how to start with the Linux kernel, what do I need to start contribute to the Linux kernel and and others like these. Generally people are interested contribute to open source project for different reasons and the Linux kernel is not exception:

![google-linux](http://s4.postimg.org/yg9z5zx0d/google_linux.png)

So, seems that people are interested about Linux kernel development process. I thought it will be strange if the book about the Linux kernel will not contain a part that will describe how to take a part in the Linux kernel development and that's why I decided to write it. You will not find information about why you should be interested in contributing to the Linux kernel in this part. I see many benefits to learn source code of the Linux kernel. I don't know how about you, that's why I have no answer on this question. But if you are interested how to start with Linux kernel development, this part is for you.

Let's start.

How to start with Linux kernel
---------------------------------------------------------------------------------

First of all let's look how to get, build and run the Linux kernel. Actually you can run your custom build of the Linux kernel in two ways:

* Run the Linux kernel on a virtual machine;
* Run the Linux kernel on real hardware.

I'll provide descriptions for both methods. Before we will start to do something with the Linux kernel, we need to get it. There are a couple of ways how to do it. All depends on your purpose. If you just want to update the current version of the Linux kernel on your computer, you can use the instructions specific for your Linux [distro](https://en.wikipedia.org/wiki/Linux_distribution).

In the first case you just need to download new version of the Linux kernel with the [package manager](https://en.wikipedia.org/wiki/Package_manager). For example, to upgrade the version of the Linux kernel to `4.1` for [Ubuntu (Vivid Vervet)](http://releases.ubuntu.com/15.04/), you will just need to execute the following commands:

```
$ sudo add-apt-repository ppa:kernel-ppa/ppa
$ sudo apt-get update
```

After this execute this command:

```
$ apt-cache showpkg linux-headers
```

and choose the version of the Linux kernel in which you are interested. In the end execute the next command and replace `${version}` with the version that you chose in the output of the previous command:

```
$ sudo apt-get install linux-headers-${version} linux-headers-${version}-generic linux-image-${version}-generic --fix-missing
```

and reboot your system. After the reboot you will see the new kernel in the [grub](https://en.wikipedia.org/wiki/GNU_GRUB) menu.

In the other way if you are interested in the Linux kernel development, you will need to get the source code of the Linux kernel. You can find it on the [kernel.org](https://kernel.org/) website and download an archive with the Linux kernel source code. Actually the Linux kernel development process is fully built around `git` [version control system](https://en.wikipedia.org/wiki/Version_control). So you can get it with `git` from the `kernel.org`:

```
$ git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
```

I don't know how about you, but I prefer `github`. There is a [mirror](https://github.com/torvalds/linux) of the Linux kernel mainline repository, so you can clone it with:

```
$ git clone git@github.com:torvalds/linux.git
```

Actually I'm using my [fork](https://github.com/0xAX/linux) for development and when I want to pull updates from the main repository I just execute the following command:

```
$ git checkout master
$ git pull upstream master
```

Note that the remote name of the main repository is `upstream`. To add a new remote with the main linux repository you can execute:

```
git remote add upstream git@github.com:torvalds/linux.git
```

After this you will have two remotes:

```
~/dev/linux (master) $ git remote -v
origin	git@github.com:0xAX/linux.git (fetch)
origin	git@github.com:0xAX/linux.git (push)
upstream	https://github.com/torvalds/linux.git (fetch)
upstream	https://github.com/torvalds/linux.git (push)
```

One is of you fork (`origin`) and the second is for the main repository (`upstream`).

Now that we have a local copy of the Linux kernel source code, we need to configure and build it. The Linux kernel can be configured in different ways. The simplest way is to just copy the configuration file of the already installed kernel that is located in the `/boot` directory:

```
$ sudo cp /boot/config-$(uname -r) ~/dev/linux/.config
```

If your current Linux kernel was built with the support for access to the `/proc/config.gz` file, you can copy your actual kernel configuration file with this command:

```
$ cat /proc/config.gz | gunzip > ~/dev/linux/.config
```

If you are not satisfied with the standard kernel configuration that is provided by the maintainers of your distro, you can configure the Linux kernel manually. There are a couple of ways to do it. The Linux kernel root [Makefile](https://github.com/torvalds/linux/blob/master/Makefile) provides a set of targets that allows you to configure it. For example `menuconfig` provides a menu-driven interface for the kernel configuration:

![menuconfig](http://s21.postimg.org/zcz48p7yf/menucnonfig.png)

The `defconfig` argument generates the default kernel configuration file for the current architecture, for example [x86_64 defconfig](https://github.com/torvalds/linux/blob/master/arch/x86/configs/x86_64_defconfig). You can pass the `ARCH` command line argument to `make` to build `defconfig` for the given architecture:

```
$ make ARCH=arm64 defconfig
```

The `allnoconfig`, `allyesconfig` and `allmodconfig` arguments allow you to generate a new configuration file where all options will be disabled, enabled and enabled as modules respectively. The `nconfig` command line arguments that provides `ncurses` based program with menu to configure Linux kernel:

![nconfig](http://s29.postimg.org/hpghikp4n/nconfig.png)

And even `randconfig` to generate random Linux kernel configuration file. I will not write how to configure the Linux kernel, which options to enable and what not, because it makes no sense to do so for two reasons: First of all I do not know your hardware and second, if you know your hardware, the only remaining task is to find out how to use programs for kernel configuration, and all of them are pretty simple to use.

Ok, for this moment we got the source code of the Linux kernel and configured it. The next step is the compilation of the Linux kernel. The simplest way to compile Linux kernel is just execute:

```
$ make
scripts/kconfig/conf  --silentoldconfig Kconfig
#
# configuration written to .config
#
  CHK     include/config/kernel.release
  UPD     include/config/kernel.release
  CHK     include/generated/uapi/linux/version.h
  CHK     include/generated/utsrelease.h
  ...
  ...
  ...
  OBJCOPY arch/x86/boot/vmlinux.bin
  AS      arch/x86/boot/header.o
  LD      arch/x86/boot/setup.elf
  OBJCOPY arch/x86/boot/setup.bin
  BUILD   arch/x86/boot/bzImage
  Setup is 15740 bytes (padded to 15872 bytes).
System is 4342 kB
CRC 82703414
Kernel: arch/x86/boot/bzImage is ready  (#73)
```

command. To increase the speed of kernel compilation you can pass `-jN` command line argument to the `make` util, where `N` specifies the number of commands to run simultaneously:

```
$ make -j8
```

If you want to build Linux kernel for an architecture that differs from your current, the simplest way to do it pass two arguments:

* `ARCH` command line argument and the name of the target architecture;
* `CROSS_COMPILER` command line argument and the cross-compiler tool prefix;

For example if we want to compile the Linux kernel for the [arm64](https://en.wikipedia.org/wiki/ARM_architecture#AArch64_features) with default kernel cnofiguration file, we need to execute following command:

```
$ make -j4 ARCH=arm64 CROSS_COMPILER=aarch64-linux-gnu- defconfig
$ make -j4 ARCH=arm64 CROSS_COMPILER=aarch64-linux-gnu-
```

As result of compilation we can see the compressed kernel - `arch/x86/boot/bzImage`. Now we have compiled kernel and we can either install it on our computer or just run it in an emulator.

Installing Linux kernel
--------------------------------------------------------------------------------

As I already wrote we will consider two ways how to launch new kernel: In the first case we can install and run the new version of the Linux kernel on the real hardware and the second is launch the Linux kernel on a virtual machine. In the previous paragraph we saw how to build the Linux kernel from source code and as a result we have got compressed image:

```
...
...
...
Kernel: arch/x86/boot/bzImage is ready  (#73)
```

After we have got the [bzImage](https://en.wikipedia.org/wiki/Vmlinux#bzImage) we need to install `headers`, `modules` of the new Linux kernel with the:

```
$ sudo make headers_install
$ sudo make modules_install
```

and directly the kernel itself:

```
$ sudo make install
```

From this moment we have installed new version of the Linux kernel and now we must tell the `bootloader` about it. Of course we can add it manually by the editing of the `/boot/grub2/grub.cfg` configuration file, but I prefer to use a script for this purpose. I'm using two differnet Linux distros: Fedora and Ubuntu. There are two different ways to update the [grub](https://en.wikipedia.org/wiki/GNU_GRUB) configuration file. I'm using following script for this purpose:

```shell
#!/bin/bash

source "term-colors"

DISTRIBUTIVE=$(cat /etc/*-release | grep NAME | head -1 | sed -n -e 's/NAME\=//p')
echo -e "Distributive: ${Green}${DISTRIBUTIVE}${Color_Off}"

if [[ "$DISTRIBUTIVE" == "Fedora" ]] ;
then
    su -c 'grub2-mkconfig -o /boot/grub2/grub.cfg' 
else
    sudo update-grub
fi

echo "${Green}Done.${Color_Off}"
```

This is the last step of the new Linux kernel installation and after this you can reboot your computer and select new version of the kernel during boot.

The second case is to launch new Linux kernel in the virtual machine. I prefer [qemu](https://en.wikipedia.org/wiki/QEMU). First of all we need to build initial ramdisk - [initrd](https://en.wikipedia.org/wiki/Initrd) for this. The `initrd` is a temporary root file system that is used by the Linux kernel during initialization process while other filesystems are not mounted. We can build `initrd` with the following commands:

First of all we need to download [busybox](https://en.wikipedia.org/wiki/BusyBox) and run `menuconfig` for its configuration:

```shell
$ mkdir initrd
$ cd initrd
$ curl http://busybox.net/downloads/busybox-1.23.2.tar.bz2 | tar xjf -
$ cd busybox-1.23.2/
$ make menuconfig
$ make -j4
```

The `busybox` is an executable file - `/bin/busybox` that contains a set of standard tools like [coreutils](https://en.wikipedia.org/wiki/GNU_Core_Utilities) and etc. In the `busysbox` menu we need to enable: `Build BusyBox as a static binary (no shared libs)` option:

![busysbox menu](http://s18.postimg.org/sj92uoweh/busybox.png)

We can find this menu in the:

```
Busybox Settings
--> Build Options
```

After this we exit from the `busysbox` configuration menu and execute following commands for building and installation of it:

```
$ make -j4
$ sudo make install
```

Ok, the `busybox` is installed from this moment and we can start to build our `initrd`. Do do this, we go to the previous `initrd` directory and:

```
$ cd ..
$ mkdir -p initramfs
$ cd initramfs
$ mkdir -pv {bin,sbin,etc,proc,sys,usr/{bin,sbin}}
$ cp -av ../busybox-1.23.2/_install/* .
```

copy `busybox` fields to the `bin`, `sbin` and other directories. Now we need to create executable `init` file that will be executed as a first process in the system. My `init` file just mounts [procfs](https://en.wikipedia.org/wiki/Procfs) and [sysfs](https://en.wikipedia.org/wiki/Sysfs) filesystems and executed shell:

```shell
#!/bin/sh
 
mount -t proc none /proc
mount -t sysfs none /sys
 
exec /bin/sh
```

Now we can create an archive that will be our `initrd`:

```
$ find . -print0 | cpio --null -ov --format=newc | gzip -9 > ~/dev/initrd_x86_64.gz
```

We can now run our kernel in the virtual machine. As I already wrote I prefer [qemu](https://en.wikipedia.org/wiki/QEMU) for this. We can run our kernel with the following command:

```
$ qemu-system-x86_64 -snapshot -m 8GB -serial stdio -kernel ~/dev/linux/arch/x86_64/boot/bzImage -initrd ~/dev/initrd_x86_64.gz -append "root=/dev/sda1 ignore_loglevel"
```

![qemu](http://s22.postimg.org/b8ttyigup/qemu.png)

From now we can run the Linux kernel in the virtual machine and this means that we can begin to change and test the kernel.

Getting started with the Linux Kernel Development
---------------------------------------------------------------------------------

The main point of this paragraph is answer on two questions: What to do and what not to do before you will send your first patch to the Linux kernel. Please, do not confuse this `to do` with `todo`. I have no answer what you can fix in the Linux kernel. I just want to tell you my workflow during experimenting with the Linux kernel source code.

First of all I'm trying to pull last updates from the Linus's repo with the following commands:

```
$ git checkout master
$ git pull upstream master
```

After this my local repository with the Linux kernel source code is synced with the [mainline](https://github.com/torvalds/linux) repository. Now we can make some changes in the source code. As I already wrote, I have no advice for you where you can start and what `TODO` in the Linux kernel. But the best place for newbies is `staging` tree. In other words the set of drivers from the [drivers/staging](https://github.com/torvalds/linux/tree/master/drivers/staging). The maintainer of the `staging` tree is [Greg Kroah-Hartman](https://en.wikipedia.org/wiki/Greg_Kroah-Hartman) and the `staging` tree is that place where your trivial patch can be accepted. Let's look on a simple example that describes how to generate patch, check it and send to the [Linux kernel mail listing](https://lkml.org/).

If we will look on the driver for the [Digi International EPCA PCI](https://github.com/torvalds/linux/tree/master/drivers/staging/dgap) based devices, we will see `dgap_sindex` function:

```C
static char *dgap_sindex(char *string, char *group)
{
	char *ptr;

	if (!string || !group)
		return NULL;

	for (; *string; string++) {
		for (ptr = group; *ptr; ptr++) {
			if (*ptr == *string)
				return string;
		}
	}

	return NULL;
}
```

on the `295` line. This function looks for a match of any character in the group, and returns that position. During research of source code of the Linux kernel, I have noted that [lib/string.c](https://github.com/torvalds/linux/blob/master/lib/string.c#L473) source code file contains implementation of the `strpbrk` function that does the same that `dgap_sinidex`. It is not a good idea to use a custom implementation of a function that already exists. So we can remove the `dgap_sindex` function from the [drivers/staging/dgap/dgap.c](https://github.com/torvalds/linux/blob/master/drivers/staging/dgap/dgap.c) source code file and use the `strpbrk` instead.

First of all let's create new `git` branch based on the current master that synced with the Linux kernel mainline repo:

```
$ git checkout -b "dgap-remove-dgap_sindex"
```

And now we can replace the `dgap_sindex` with the `strpbrk`. After we did all changes we need to recompile the Linux kernel or just [dgap](https://github.com/torvalds/linux/tree/master/drivers/staging/dgap) directory. Do not forget to enable this driver in the kernel configuration. You can find it in the:

```
Device Drivers
--> Staging drivers
----> Digi EPCA PCI products
```

![dgap menu](http://s4.postimg.org/d3pozpge5/digi.png)

Now is time to make commit. I'm using following combination for this:

```
$ git add .
$ git commit -s -v
```

After the last command an editor will be openned that will be chosen from `$GIT_EDITOR` or `$EDITOR` environment variable. The `-s` command line argument will add `Signed-off-by` line by the committer at the end of the commit log message. You can find this line in the end of each commit message, for example - [00cc1633](https://github.com/torvalds/linux/commit/00cc1633816de8c95f337608a1ea64e228faf771). The main point of this line is the tracking of who did a change. The `-v` option show unified diff between the HEAD commit and what would be committed at the bottom of the commit message. It is not necessary, but very useful sometimes. A couple of words about commit message. Actually a commit message consists from two parts:

The first part is on the first line and contains short descrption of changes. It starts from the `[PATCH]` prefix followed by a subsystem, driver or architecture name and after `:` symbol short description. In our case it will be something like this:

```
[PATCH] staging/dgap: Use strpbrk() instead of dgap_sindex()
```

After short description usually we have an empty line and full description of the commit. In our case it will be:

```
The <linux/string.h> provides strpbrk() function that does the same that the
dgap_sindex(). Let's use already defined function instead of writing custom.
```

And the `Sign-off-by` line in the end of the commit message. Note that each line of a commit message must no be longer than `80` symbols and commit message must describe your changes in detail. Do not just write a commit message like: `Custom function removed`, you need to describe what you are did and why. The patch reviewers must know what they review. Besides this commit messages in this view are very helpful. Each time when we can't understand something, we can use [git blame](http://git-scm.com/docs/git-blame) to read description of changes.

After we have commited changes time to generate patch. We can do it with the `format-patch` command:

```
$ git format-patch master 
0001-staging-dgap-Use-strpbrk-instead-of-dgap_sindex.patch
```

We've passed name of the branch (`master` in this case) to the `format-patch` command that will generate a patch with the last changes that are in the `dgap-remove-dgap_sindex` branch and not are in the `master` branch. As you can note, the `format-patch` command generates file that contains last changes and has name that is based on the commit short description. If you want to generate a patch with the custom name, you can use `--stdout` option:

```
$ git format-patch master --stdout > dgap-patch-1.patch
```

The last step after we have generated our patch is just to send it to the Linux kernel mail listing. Of course you can use any email client, but the `Git` provides special command for this: `git send-email`. Before you will send your patch, you need to know where to send it. Yes, you can send it just to the Linux kernel mail listing address which is `linux-kernel@vger.kernel.org`, but there is a high probability that the patch will be ignored, because as you may already know there is the large flow of messages on the Linux kernel mail listing. The better way will be send to a maintainer of subsystem where you have made changes. We can find maintainer and other related guys who has touched the code with the `get_maintainer.pl` script. All of you need is just pass file or directory where you wrote a code. Go to the root directory with source code of the Linux kernel and execute it:

```
$ ./scripts/get_maintainer.pl -f drivers/staging/dgap/dgap.c
Lidza Louina <lidza.louina@gmail.com> (maintainer:DIGI EPCA PCI PRODUCTS)
Mark Hounschell <markh@compro.net> (maintainer:DIGI EPCA PCI PRODUCTS)
Daeseok Youn <daeseok.youn@gmail.com> (maintainer:DIGI EPCA PCI PRODUCTS)
Greg Kroah-Hartman <gregkh@linuxfoundation.org> (supporter:STAGING SUBSYSTEM)
driverdev-devel@linuxdriverproject.org (open list:DIGI EPCA PCI PRODUCTS)
devel@driverdev.osuosl.org (open list:STAGING SUBSYSTEM)
linux-kernel@vger.kernel.org (open list)
```

Yout will see the set of the names and related emails. Now we can send our patch with:

```
$ git send-email --to "Lidza Louina <lidza.louina@gmail.com>" \
  --cc "Mark Hounschell <markh@compro.net>"                   \
  --cc "Daeseok Youn <daeseok.youn@gmail.com>"                \
  --cc "Greg Kroah-Hartman <gregkh@linuxfoundation.org>"      \
  --cc "driverdev-devel@linuxdriverproject.org"               \
  --cc "devel@driverdev.osuosl.org"                           \
  --cc "linux-kernel@vger.kernel.org"
```

That's all. The patch is sent and now only have to wait feedback from the Linux kernel developers. After you will sent a patch and a maintainer accepted it, you will find it in the maintainer's repository (for example [patch](https://git.kernel.org/cgit/linux/kernel/git/gregkh/staging.git/commit/?h=staging-testing&id=b9f7f1d0846f15585b8af64435b6b706b25a5c0b) that you saw in this part) and after some time a maintainer will send pull request to Linus and you will see your patch in the mainline repository.

That's all.

Some advice
--------------------------------------------------------------------------------

In the end of this part I want to give you some advice that will describe what to do and what not to do during development of the Linux kernel:

* Think, Think, Think. And think again before you decided to send a patch.

* Each time when you have changed something in the Linux kernel source code - compile it. After any changes. Again and again. Nobody likes changes that don't even compile.

* The Linux kernel has a coding style [guide](https://github.com/torvalds/linux/blob/master/Documentation/CodingStyle) and you need to comply with it. There is great script which can help to check your changes. This script is - [scripts/checkpatch.pl](https://github.com/torvalds/linux/blob/master/scripts/checkpatch.pl). Just pass source code file with changes to it and you will see:

```
$ ./scripts/checkpatch.pl -f drivers/staging/dgap/dgap.c
WARNING: Block comments use * on subsequent lines
#94: FILE: drivers/staging/dgap/dgap.c:94:
+/*
+     SUPPORTED PRODUCTS

CHECK: spaces preferred around that '|' (ctx:VxV)
#143: FILE: drivers/staging/dgap/dgap.c:143:
+	{ PPCM,        PCI_DEV_XEM_NAME,     64, (T_PCXM|T_PCLITE|T_PCIBUS) },

```

Also you can see problematic places with the help of the `git diff`:

![git diff](http://oi60.tinypic.com/2u91rgn.jpg)

* [Linus doesn't accept github pull requests](https://github.com/torvalds/linux/pull/17#issuecomment-5654674)

* If your change consists from some different and not too closely related changes, you need to split your changes. Each change must in a separate commit. The `git format-patch` command will generate patches for each commit and subject of each patch will contain `vN` prefix where the `N` is the number of the patch. If you are planning to send not patch, but series of patches, will be good if you will pass `--cover-letter` option to the `git format-patch` command. This will generate additional file that will contain cover letter that you can use to describe what your patchset changes. Also it is good idea to use `--in-reply-to` option in the `git send-email` command. This option allows you to send your patchseries in reply to the your cover message, so the structure of the your patch will be look like this for a maintainer:

```
|--> cover letter
  |----> patch_1
  |----> patch_2
```

You need to pass `message-id` as value of the `--in-reply-to` option that you can find in the output of the `git send-email`:

![send-email](http://oi60.tinypic.com/2mhd8wo.jpg)

Note one important thing that your email must be in the [plain text](https://en.wikipedia.org/wiki/Plain_text) format. Generally these two `git` commands: `send-email` and `format-patch` are very useful during development, look on the documentation for this commands and you will find many interesting and useful options: [git send-email](http://git-scm.com/docs/git-send-email) and [git format-patch](http://git-scm.com/docs/git-format-patch).

* Do not be surprised if you do not get an answer right away after you will send your patch. Maintainers are people too and people can sometimes be busy.

* The [scripts](https://github.com/torvalds/linux/tree/master/scripts) directory contains many different useful scripts that are related to the Linux kernel development. We already saw two scripts from this directory: the `checkpatch.pl` and the `get_maintainer.pl` scripts. Besides these two, you can find [stackusage](https://github.com/torvalds/linux/blob/master/scripts/stackusage) script that will print usage of the stack as you can understand from the script's name, [extract-vmlinux](https://github.com/torvalds/linux/blob/master/scripts/extract-vmlinux) for extracting uncompressed kernel image, and many others. Besides this `scripts` directory you can find some very useful [scripts](https://github.com/lorenzo-stoakes/kernel-scripts) by [Lorenzo Stoakes](https://twitter.com/ljsloz) for kernel development.

* Subscribe on the Linux kernel mail listing. Yes, there is large flow of letters every day on `lkml`, but it is very useful to read and understand things like current state of the Linux kernel and etc. Besides this there is a [set](http://vger.kernel.org/vger-lists.html) of the mail listings which are related to the different Linux kernel subsystems.

* If your patch is not accepted from the first time and you have got feedback from Linux kernel developers, make changes and resend the patch with the `[PATCH vN]` prefix, where `N` is the number of patch version. For example:

```
[PATCH v2] staging/dgap: Use strpbrk() instead of dgap_sindex()
```

Also it must contain changelog that will describe all changes changes from previous patch versions.

That's all. Of course, these are not all the subtleties of the Linux kernel development collected in this part, but some of the most important.

Happy Hacking!

Conclusion
--------------------------------------------------------------------------------

This is the end of this part and here we saw all steps from the getting source code of the Linux kernel to sending of a patch to the Linux kernel mailing list. Hope it will help you to join to the Linux kernel community.

If you have any questions or suggestions, write me an [email](kuleshovmail@gmail.com) or ping [me](https://twitter.com/0xAX) on twitter.

Please note that English is not my first language, and I am really sorry for any inconvenience. If you find any mistakes please let me know via email or send a PR.

Links
--------------------------------------------------------------------------------

* [blog posts about assembly programming for x86_64](http://0xax.github.io/categories/assembly/)
* [Assembler](https://en.wikipedia.org/wiki/Assembly_language#Assembler)
* [distro](https://en.wikipedia.org/wiki/Linux_distribution)
* [package manager](https://en.wikipedia.org/wiki/Package_manager)
* [grub](https://en.wikipedia.org/wiki/GNU_GRUB)
* [kernel.org](https://kernel.org/)
* [version control system](https://en.wikipedia.org/wiki/Version_control)
* [arm64](https://en.wikipedia.org/wiki/ARM_architecture#AArch64_features)
* [bzImage](https://en.wikipedia.org/wiki/Vmlinux#bzImage)
* [qemu](https://en.wikipedia.org/wiki/QEMU)
* [initrd](https://en.wikipedia.org/wiki/Initrd)
* [busybox](https://en.wikipedia.org/wiki/BusyBox)
* [coreutils](https://en.wikipedia.org/wiki/GNU_Core_Utilities)
* [procfs](https://en.wikipedia.org/wiki/Procfs)
* [sysfs](https://en.wikipedia.org/wiki/Sysfs)
* [Linux kernel mail listing archive](https://lkml.org/)
* [Linux kernel coding style guide](https://github.com/torvalds/linux/blob/master/Documentation/CodingStyle)
* [How to Get Your Change Into the Linux Kernel](https://github.com/torvalds/linux/blob/master/Documentation/SubmittingPatches)
* [Linux Kernel Newbies](http://kernelnewbies.org/)
* [plain text](https://en.wikipedia.org/wiki/Plain_text)
