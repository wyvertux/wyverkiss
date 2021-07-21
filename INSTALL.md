This installation guide is adapted from KISS Linux's installation
guidelines. It's been modified to fit Wyverkiss' whole situation.

Welcome to the installation guide for Wyverkiss.

The installation utilizes a tarball which is unpacked to '/'. This same
tarball is also directly usable as a chroot from any Linux distribution.

A live-CD from a another distribution is required to bootstrap
Wyverkiss. It does not matter which distribution is used so long as it
includes 'tar', 'xz' and other basic utilities.

The guide assumes you have booted a live-CD, are logged in as root, have
your disks, partitions and filesystems setup and have an internet
connection.

## The Process

- Install Wyverkiss
- Download the latest release
- Verify the checksums
- Verify the signature
- Unpack the tarball
- Enter the chroot 

- Setup repositories
- KISS_PATH
- Official repositories

- Enable repository signing 
- Build and install gnupg1
- Import My Key 
- Enable Signature Verification

- Rebuild KISS  
- Modify Compiler Options  
- Update All Base Packages
- Rebuild All Packages  

- Userspace tools 
- Filesystems
- WiFi
- Dynamic IP addressing

- The hostname  

- The kernel
- Install required packages
- Download the kernel sources
- Download firmware blobs 
- Configure the kernel
- Build the kernel
- Install the kernel 

- The bootloader 
- Build and install grub  
- Setup grub 

- Install init scripts
- Set a root password 
- Add a normal user
- Install graphical session (wayland) 
- Further steps


## Install Wyverkiss

Start by declaring two variables.

```sh
$ url=https://github.com/wyvertux/wyverkiss/releases/download/2021.7-3
$ file=kiss-chroot-2021.7-3.tar.xz
```


### Download the latest release

```sh
$ curl -fLO "$url/$file
```


### Verify checksums (recommended)

```sh
$ curl -fLO "$url/$file.sha256"
$ sha256sum -c < "$file.sha256"
```


### Verify signature (recommended)

This step verifies that the release was signed by its creator. If the
live OS of your choice does not include GPG, this step can also be done
on another machine (with the same tarball). There are two ways to do
this. By using minisign (or perhaps any `signify`-compatible tool), or
GnuPG. Yes, I'm aware of the irony, using GNU tools to verify a
self-proclaimed GNU-free KISS flavor.

```sh
## Download the armored ASCII file
$ curl -fLO "$url/$file.asc"## for GnuPG
$ curl -fLO "$url/$file.minisig"## for minisign

## The following step applies only for minisign
$ minisign -Vm "$file" -P RWRX6BWR+kgO3kLVuWgBezKRR9IdFiXVabcicNlmEU+qKmYeP82ZtFMb

## The following steps apply only for GnuPG.

## Import my public key (if this fails, try another keyserver).
$ gpg --keyserver keys.gnupg.net --recv-keys 

## Verify the signature
$ gpg --verify "$file.asc"
```


### Unpack the tarball

This step effectively installs Wyverkiss to /mnt. The tarball contains a
working system minus the bootloader and kernel.

$ cd /mnt
$ tar xvf "$OLDPWD/$file"


### Enter the chroot

This is a simple script to chroot into /mnt and set up the environment.
The script handles mounting pseudo filesystems (/proc, /dev, /sys, etc),
enabling network inside the chroot and clean up upon exit.

On execution of this step you will be running Wyverkiss! The next steps
involve the kernel, software compilation and system setup.

```sh
$ /mnt/bin/kiss-chroot /mnt
```


## Setup repositories

The repository system is quite different to that of other distributions.
The system is controlled via an environment variable called KISS_PATH.
This variable is analogous to PATH, a colon separated list of absolute
paths.

A repository is merely a directory (repo) containing directories
(packages) and can be located anywhere on the file-system. The full path
to the directory is the value to KISS_PATH.

The chroot/installation tarball does not come with any repositories by
default, nor does the package manager expect or assume that any exist in
a given location. This is entirely up to the user.


### Setting KISS_PATH

The variable can be set system-wide, per-user, per-session, per-command,
and even programmatically. This guide will cover setting it for the
current user with an example repository layout.

Take this layout:

```
  ~/repo
  |
  +- wyverkiss/
  |  - .git/
  |  - core/
  |  - extra/
  |  - wayland/
  |  - gnu/
  |
  +- personal/
  |  - games/
  |  - web/
```

- Repositories are stored in ~/repos/ which is a per-user configuration.
- There is a git repository called 'wyverkiss' containing four `kiss`
  repositories.
- There is a directory called personal, not tracked by git.
- The personal directory contains two `kiss` repositories.

- This user's `KISS_PATH` could look like this:

```sh
## This is the contents for ~/.profile, the numbers on the left indicate the line number
1 export KISS_PATH=''
2 KISS_PATH=$KISS_PATH:$HOME/repos/personal/games
3 KISS_PATH=$KISS_PATH:$HOME/repos/personal/web
4 KISS_PATH=$KISS_PATH:$HOME/repos/repo/core
5 KISS_PATH=$KISS_PATH:$HOME/repos/repo/extra
6 KISS_PATH=$KISS_PATH:$HOME/repos/repo/wayland
```

- All repositories are enabled.
- This is a per-user configuration using ~/.profile
- The package manager will search this list in the order it is defined.

TIP: Run '. ~/.profile' for changes to immediately take effect.


### Official repositories

The official repositories contain everything from the base system to a
working web browser (Firefox) and media player (mpv). This includes
Wayland, rust, nodejs and a lot of other useful software.

Clone the repository to the directory of your choosing.

```sh
$ git clone https://github.com/wyvertux/wyverkiss
```

This will be cloned to a directory called 'wyverkiss'. This directory
contains multiple KISS repositories (core, extra, testing, wayland, and
gnu). core, extra, and gnu must be enabled as this guide requires their
use. Wayland is optional.


### Universe

There are many more repositories in existence, each providing a unique
set of software. These are all independently created and managed by
users.

- https://kisslinux.xyz/wiki/community/repositories
- https://github.com/topics/kiss-repo

It may be desirable to save this step for post-installation.

Please note that all of them are probably tested against KISS Linux's
repo and not Wyverkiss. While you can ask them to support Wyverkiss
environment, they are under no obligation to adhere to your request.

Repositories should now be setup and in functioning order. Run 'kiss
search \*' (notice the backslash) to print all repository packages in
the search order of the package manager.

#### TIP!

The install guide (and all documentation) is available via 'kiss help'.

```sh
$ kiss help install
```


## Rebuild Wyverkiss

This step is entirely optional and can also be done post-installation.


### Modify compiler options (optional)

These options have been tested and work with every package in the
repositories. If you'd like to play it safe, use -O2 or -Os instead of
-O3.

If your system has a low amount of memory, omit -pipe. This option
speeds up compilation but may use more memory.

If you intend to transfer packages between machines, omit -march=native.
This option tells the compiler to use optimizations unique to your
processor's architecture.

The `-jX` option should match the number of CPU threads available. You
can omit this, however builds will then be limited to a single thread.

- CFLAGS/CXXFLAGS  

```sh 
## NOTE: The 'O' in '-O3' is the letter O and NOT 0 (ZERO). 
$ export CFLAGS="-O3 -pipe -march=native"  
$ export CXXFLAGS="$CFLAGS"
```  

- MAKEFLAGS

```sh
## NOTE: '4' should be changed to match the number of threads.  
$ export MAKEFLAGS="-j4"
```  


### Update all base packages to the latest version


This is how updates are performed on a Wyverkiss system. This command
uses git to pull down changes from all enabled repositories and will
then optionally handle the build/install process.

```sh  
$ kiss update
```


### Rebuild all packages

We simply cd to the installed packages database and use a glob to grab
the name of every installed package. This glob is then passed to the
package manager as a list of packages to build.


```sh
$ cd /var/db/kiss/installed && kiss build *
```


## Userspace tools

Each kiss action (build, install, etc) has a shorthand alias. From now
on, 'kiss b' will be used in place of 'kiss build' and so on.

The software below is required unless stated otherwise.


### Filesystem utilities

NOTE: Open an issue for additional filesystem support.

- EXT2, EXT3, EXT4 

```sh
$ kiss b e2fsprogs 
```

- FAT, VFAT

```sh  
$ kiss b dosfstools
```


### WiFi (optional)

```sh  
$ kiss b wpa_supplicant
```


### Dynamic IP addressing (optional)

```sh
$ kiss b dhcpcd
```


## The hostname

- Create the /etc/hostname file

```sh 
$ echo HOSTNAME > /etc/hostname 
```

- Update the /etc/hosts file

```
127.0.0.1  HOSTNAME.localdomain  HOSTNAME 
::1        HOSTNAME.localdomain  HOSTNAME  ip6-localhost  
```

NOTE: This step must be done every time the hostname is changed.


## The kernel

This step involves configuring and building your own Linux kernel. If you have
not done this before, below are a few guides to get you started.

- https://wiki.gentoo.org/wiki/Kernel/Gentoo_Kernel_Configuration_Guide
- https://wiki.gentoo.org/wiki/Kernel/Configuration
- https://kernelnewbies.org/KernelBuild

The Linux kernel is not managed by the package manager. The kernel is managed
manually by the user. (Rationale: @/faq#5.3)

KISS does not support booting using an initramfs (see @/faq#5.2). When
configuring your kernel ensure that all required file-system, disk controller
and USB drivers are built with [*] (=y) and not [m] (=m).

  
TIP: The Wiki contains a basic kernel configuration page.  
https://kisslinux.xyz/wiki/kernel/config  


### Install required packages

- libelf (required in most if not all cases).  

```sh
$ kiss b libelf
```  

- netbsd-curses (required only for 'gmake menuconfig').

```sh
$ kiss b netbsd-curses
``

- perl (required in nearly all cases). 

```sh
$ kiss b perl  
```

- GNU make

```sh
$ kiss b gmake
```

- GNU as (might be required if `LLVM_IAS` fails later)

```sh
$ kiss b gnu-as
```

TIP: A patch can be applied to remove this requirement.
- https://kisslinux.xyz/wiki/kernel/config#5.0
- /usr/share/doc/kiss/wiki/kernel/patches/kernel-no-perl.patch


### Download the kernel sources and required patches

Kernel releases:

- https://kernel.org (vanilla)
- https://www.fsfla.org (libre)

A larger list of kernels can be found here:

- https://wiki.archlinux.org/index.php/Kernel


- Download the kernel sources. 

```sh
$ curl -fLO KERNEL_SOURCE  
```

- Extract the kernel sources. 

```sh
$ tar xvf KERNEL_SOURCE
$ cd linux-*
```


### Download firmware blobs (if required)

To keep the KISS (and subsequently, Wyverkiss) repositories entirely
FOSS, the proprietary kernel firmware is omitted. This also makes sense
as the kernel itself is manually managed by the user. This step is only
required if your hardware needs firmware.

https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git


- Download and extract the firmware.

```sh
$ curl -fLO FIRMWARE_SOURCE.tar.gz 
$ tar xvf FIRMWARE_SOURCE.tar.gz
```

- Copy the required drivers to `/usr/lib/firmware`.

```sh  
$ mkdir -p /usr/lib/firmware
$ cp -R ./path/to/driver /usr/lib/firmware
```

### Patch the kernel

As for kernel 5.13, the following patches need to be applied:

- Fix `objtool` failing with musl systems

```sh
$ sed 's/$(LIBELF_FLAGS)/$(LIBELF_FLAGS) -D__always_inline=inline/' tools/objtool/Makefile > _
$ mv -f _ tools/objtool/Makefile  
```

- `byacc` support

```sh
$ curl -fLo byacc.patch https://patchwork.kernel.org/project/linux-kbuild/patch/20200130162314.31449-1-e5ten.arch@gmail.com/raw/
$ patch -p1 < byacc.patch
```


### Configure the kernel

You can determine which drivers you need by searching the web for your
hardware and the Linux kernel. If you require firmware blobs, the
drivers you enable must be enabled as `[m]` (modules). You can also
optionally include the firmware in the kernel itself.

- Generate a default config with most drivers built into the kernel.

```sh  
$ gmake LLVM=1 LLVM_IAS=1 YACC=byacc defconfig
```

- Open an interactive menu to edit the generated .config and enable 
  anything extra you may need. 

```sh  
$ gmake LLVM=1 LLVM_IAS=1 YACC=byacc menuconfig
```


- Store the generated config for reuse later.  

```sh  
$ cp .config /path/to/somewhere
```
  
TIP: The kernel can backup its own .config file.
https://kisslinux.xyz/wiki/kernel/config#2.0  
  

### Build the kernel

This may take a while to complete. The compilation time depends on your
hardware and kernel configuration.

```sh
$ gmake LLVM=1 LLVM_IAS=1 YACC=byacc
```


### Install the kernel

- Install the built modules (to /usr/lib). (Ignore the GCC errors).
 
```sh
$ gmake INSTALL_MOD_STRIP=1 modules_install 
```

- Install the built kernel (to /boot). (Ignore the LILO error).
  
```sh
$ gmake install 
```

Rename the kernel/system.map (vmlinuz -> vmlinuz-VERSION).

```sh  
$ mv /boot/vmlinuz/boot/vmlinuz-VERSION
$ mv /boot/System.map /boot/System.map-VERSION 
```



## The bootloader (UNDER CONSTRUCTION)

Since currently the only bootloader available for KISS is GNU GRUB, and
I don't like most of the alternatives, we cannot use it by default
although there's literally nothing preventing you to use it (we don't
block it, unlike GCC). Consider using EFISTUB instead (Sorry,
prefer-to-use-BIOS guys).

See KISS Linux’s wiki for EFISTUB for details.

- https://kisslinux.xyz/wiki/boot/efistub


## Install init scripts

The default init is busybox init (though nothing ties you to it). The
below commands install the bootup and shutdown scripts as well as the
default inittab config.

Source code: https://github.com/kisslinux/init

```sh  
$ kiss b baseinit
```


## Change the root password (recommended)

```sh  
$ passwd root  
```


## Add a normal user (recommended)

```sh  
$ adduser USERNAME 
```


### Install graphical session (wayland) (optional)

See:

- https://kisslinux.xyz/wiki/wayland/install
- https://kisslinux.xyz/wiki/wayland

#### TIP! 
  
The Wiki is available offline via 'kiss help wiki'.

```sh
$ kiss help wiki
  
$ kiss help wiki/wayland
$ kiss help wiki/wayland/install
  
$ kiss help wiki/software  
$ kiss help wiki/software/firefox  
  
```


## Further steps

The installation is complete!

If everything was done correctly, you should now be able to reboot into
your Wyverkiss installation. Typical post-installation steps should
follow, you can use the KISS wiki for that.

If you encountered any issues or have any questions, get in touch.

See: https://github.com/wyvertux/wyverkiss/issues


