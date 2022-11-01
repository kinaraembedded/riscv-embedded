# SiFive Freedom Unleashed SDK

The new experimental Freedom Unleashed (FU) SDK is based on OpenEmbedded (OE). Using OE you will be able to:

- build predefined disk images for QEMU, [SiFive HiFive Unleashed](https://www.sifive.com/boards/hifive-unleashed) development board and [SiFive HiFive Unmatched](https://www.sifive.com/boards/hifive-unmatched);
	+ __Note__: The support for HiFive Unleashed Expansion board from Microsemi is now removed from SiFive OpenEmbedded layer (i.e. [meta-sifive](https://github.com/sifive/meta-sifive)). If you have the expansion board we advice you to switch to [Microchip PolarFire SoC Yocto BSP](https://github.com/polarfire-soc/meta-polarfire-soc-yocto-bsp/) which includes support for MPFS-DEV-KIT (HiFive Unleashed Expansion Board) directly from the manufacturer. You are also welcome to use older releases (2021.02.00 or older) from SiFive OpenEmbedded layer.
	+ __Note:__  2021.02.00 release introduces the support for the SiFive HiFive Unmatched board __(pre-production 8GB variant)__. Contact your SiFive representative before using disk images built for `unmatched` machine on your particular board. If you received the __final board (16GB) variant__ via Mouser or CrowdSupply you should skip 2021.02.00 release and use 2021.04.00 (or newer).
- build custom disk images with additional software packages from various third-party OE layers;
- quickly launch QEMU VM instance with your built disk image;
- build bootloader binaries (OpenSBI, U-Boot, U-Boot SPL);
- build Device Tree Binary (DTB);
- build Linux kernel images;
- easily modify disk partition layout.

For more information on particular release see `ReleaseNotes` directory in [freedom-u-sdk](https://github.com/sifive/freedom-u-sdk) repository on GitHub.

For advanced OE usage we advice to look into [Yocto Project Documentation](http://docs.yoctoproject.org/) and [A practical guide to BitBake](https://a4z.gitlab.io/docs/BitBake/guide.html).


## Quick Start

Install `repo` command from Google if not available on your host system. Please follow [the official instructions](https://source.android.com/setup/develop#installing-repo) by Google.

Then install a number of packages for BitBake (OE build tool) to work properly on your host system. BitBake itself depends on Python 3. Once you have Python 3 installed BitBake should be able to tell you most of the missing packages.

> For Ubuntu 18.04 (or newer) install python3-distutils package.

Detailed instructions for various distributions can be found in "[Required Packages for the Build Host](http://docs.yoctoproject.org/ref-manual/system-requirements.html#required-packages-for-the-build-host)" section in Yocto Project Reference Manual.

### Creating Workspace

This needs to be done every time you want a clean setup based on the latest layers.

```bash
mkdir riscv-sifive && cd riscv-sifive 
repo init -u https://github.com/kinaraembedded/riscv-embedded -b kinaraembedded_v1  -m tools/manifests/sifive.xml
repo sync
```

### Creating a Working Branch

If you want to make modifications to existing layers then creating working branches in all repositories is advisable.

```bash
repo start work --all
```

### Getting Build Tools (optional)

OpenEmbedded-Core requires GCC 6 or newer to be available on the host system. Your host system might have an older version of GCC if you use LTS (Long Term Support) Linux distribution (e.g. Ubuntu 16.04.6 has GCC 5.4.0). You could solve this issue by installing build tools. This requires less than 400MB of disk space. You can download pre-built one or build your own build tools.

#### Option 1: Installing OpenEmbedded-Core Build Tools (Pre-Built)

```bash
./openembedded-core/scripts/install-buildtools -r yocto-3.2_M2 -V '3.1+snapshot' -t 20200729
```

The native SDK will be installed under `$BUILDDIR/../openembedded-core/buildtools` prefix.

Finally you should be able to use build tools:

```bash
. ./openembedded-core/buildtools/environment-setup-x86_64-pokysdk-linux
```

#### Option 2: Building Your Own Build Tools

> Your host needs to have GCC 6 (or newer) or build tools installed from Option 1.

> You can find pre-built tools from the same release source in GitHub release assets.

To build your own build tools execute the command below:

```bash
bitbake buildtools-extended-tarball
```

You can find the native SDK under `$BUILDDIR/tmp-glibc/deploy/sdk/` directory.

Now you can install build tools:

```bash
$BUILDDIR/tmp-glibc/deploy/sdk/x86_64-buildtools-extended-nativesdk-standalone-nodistro.0.sh -d $BUILDDIR/../openembedded-core/buildtools -y
```

Finally you should be able to use your build tools:

```bash
. $BUILDDIR/../openembedded-core/buildtools/environment-setup-x86_64-oesdk-linux
```


### Setting up Build Environment

This step has to be done after you modify your environment with toolchain you want to use otherwise wrong host tools might be available in the package build environment. For example, `gcc` from host system will be used for building `*-native` packages.

```bash
. ./freedom-u-sdk/setup.sh
```

### you will observe the below log

> Init OE

> ### Shell environment set up for builds. ###

> You can now run 'bitbake <target>'

> Common targets are:
>   core-image-minimal
>   core-image-full-cmdline
>   core-image-sato
>   core-image-weston
>   meta-toolchain
>   meta-ide-support

> You can also run generated qemu images with a command like 'runqemu qemux86-64'.

> Other commonly useful commands are:
> - 'devtool' and 'recipetool' handle common recipe tasks
> - 'bitbake-layers' handles common layer tasks
> - 'oe-pkgdata-util' handles common target package tasks
> Your build directory already has local configuration file!
> If you want to start from scratch remove old build directory:

>   rm -rf /home/gytechs/kinara/build

> ---------------------------------------------------
> Example: MACHINE=unmatched bitbake demo-coreip-xfce4
> ---------------------------------------------------

> Buildable machine info
> ---------------------------------------------------
> * unmatched         : The SiFive HiFive Unmatched board
> * freedom-u540      : The SiFive HiFive Unleashed board
> * qemuriscv64       : The 64-bit RISC-V machine
> ---------------------------------------------------

> You can verify and fix your host tools by checking symlinks in `$BUILDDIR/tmp-glibc/hosttools` directory.

#### Option 1:Mounting root on Virtual File System  (root = /dev/vda)

### Configuring BitBake Parallel Number of Tasks/Jobs

There are 3 variables that control the number of parallel tasks/jobs BitBake will use: `BB_NUMBER_PARSE_THREADS`, `BB_NUMBER_THREADS` and `PARALLEL_MAKE`. The last two are the most important, and both are set to number of cores available on the system. You can set them in your `$BUILDDIR/conf/local.conf` or in your shell environment similar to how `MACHINE` is used (see next section). Example:

```bash
PARALLEL_MAKE="-j 4" BB_NUMBER_THREADS=4 MACHINE=qemuriscv64 bitbake core-image-minimal
```

### you will observe the below log
> Loading cache: 100%  |#######################################################################################################################################################################| Time: 0:00:01

> Loaded 0 entries from dependency cache.
> Parsing recipes: 100% |#####################################################################################################################################################################| Time: >   0:00:49
> Parsing of 2751 .bb files complete (0 cached, 2751 parsed). 4198 targets, 194 skipped, 0 masked, 0 errors.
> NOTE: Resolving any missing task queue dependencies

> Build Configuration:
> BB_VERSION           = "2.0.0"
> BUILD_SYS            = "x86_64-linux"
> NATIVELSBSTRING      = "universal"
> TARGET_SYS           = "riscv64-oe-linux"
> MACHINE              = "qemuriscv64"
> DISTRO               = "nodistro"
> DISTRO_VERSION       = "2022.04.00"
> TUNE_FEATURES        = "riscv64"
> meta                 = "work:b3114d5d438b7a63a276b4e825b62f3b1ebceed6"
> meta-oe              
> meta-python          
> meta-multimedia      
> meta-filesystems     
> meta-networking      
> meta-gnome           
> meta-xfce            = "work:1888971b1fcff1e86be175c2aac724c28d2840ca"
> meta-riscv           = "work:96b34d09763126b337df363e091d364ebb140bef"
> meta-clang           = "work:d748f542e27d58fcbcee9d1482d71cd9c38bf06b"
> meta-sifive          = "work:07141b482eae20bc13202ff27c59b4dad748bb14"
> freedom-u-sdk        = "work:beedaa10bf143904ca3ec5721086b4d22513c98b"

> Initialising tasks: 100% |##################################################################################################################################################################| Time:   0:00:03
> Sstate summary: Wanted 1061 Local 36 Mirrors 0 Missed 1025 Current 730 (3% match, 42% complete)
> Removing 749 stale sstate objects for arch riscv64: 100% |##################################################################################################################################| Time: 0:00:07
> NOTE: Executing Tasks
> NOTE: Tasks Summary: Attempted 4588 tasks of which 2585 didn't need to be rerun and all succeeded.
> NOTE: Writing buildhistory
> NOTE: Writing buildhistory took: 40 seconds
> NOTE: Build completion summary:
> NOTE:   do_populate_sysroot: 0.0% sstate reuse(0 setscene, 137 scratch)
> NOTE:   do_deploy_source_date_epoch: 0.0% sstate reuse(0 setscene, 186 scratch)
> NOTE:   do_package_qa: 6.7% sstate reuse(12 setscene, 167 scratch)
> NOTE:   do_package: 0.0% sstate reuse(0 setscene, 167 scratch)
> NOTE:   do_packagedata: 6.7% sstate reuse(12 setscene, 167 scratch)
> NOTE:   do_package_write_ipk: 6.7% sstate reuse(12 setscene, 167 scratch)
> NOTE:   do_populate_lic: 0.0% sstate reuse(0 setscene, 27 scratch)


Leaving defaults could cause high load averages, high memory usage, high IO wait and could make your system unresponsive due to resources overuse. The defaults should be changed based on your system configuration.
### After successfull compilation of above command output binaries will be generated  “tmp-glibc/deploy/images/qemuriscv64” directory. 

### Building Disk Images

There are two disk image targets added by FUSDK:

- `demo-coreip-cli` - basic command line image (**recommended**);

- `demo-coreip-xfce4` - basic graphical disk image with [Xfce 4](https://www.xfce.org/) desktop environment.

There are several machine targets defined:

- `qemuriscv64` - RISC-V 64-bit (RV64GC) for QEMU virt machine (**recommended for QEMU target**).
- `freedom-u540` - SiFive HiFive Unleashed development board.
- `unmatched` - SiFive HiFive Unmatched development board.

The QEMU machines with the additional extensions (i.e. beyond RV64GC) do not affect how packages or/and disk images are built. This means the toolchain might not provide support for the new extensions. By default packages are not built with the new instructions enabled.

> 
> Building disk images is CPU intensive, could require <10GB of sources downloaded over the Internet and <200GB of local storage.

Building disk image takes a single command which may take anything from 30 minutes to several hours depending on your hardware. Examples:

```bash
MACHINE=qemuriscv64 bitbake core-image-minimal
MACHINE=freedom-u540 bitbake core-image-minimal
MACHINE=unmatched bitbake core-image-minimal
```

### Running in QEMU

OE provides easy to use wrapper for QEMU:

```bash
MACHINE=qemuriscv64 runqemu nographic slirp
```
#### Option 2:Mounting root on RAMFS  (root = /dev/ram0)

### Configure the below steps 

### Step 1: Create a recipe for initramfs based image

```bash
vim ../openembedded-core/meta/recipes-core/images/gyt-image-minimal.bb
```

**Add below content in file**

> require core-image-minimal.bb 

> IMAGE_FSTYPES = "${INITRAMFS_FSTYPES}" 

### Step 2: Enable initramfs for gyt-image-minimal.bb  

```bash
vim ../openembedded-core/meta/conf/local.conf.sample.extended 
```

**Add below content in file**

> INITRAMFS_IMAGE = "gyt-image-minimal" 

> INITRAMFS_IMAGE_BUNDLE = "1" 

### Step 3: Chage the maximum size of intramfs  

```bash
vim ../openembedded-core/meta/conf$ vim bitbake.conf
```
>   INITRAMFS_MAXSIZE ??= "262144" 

### Step 4: Build the Images   

```bash
PARALLEL_MAKE="-j 4" BB_NUMBER_THREADS=4 MACHINE=qemuriscv64 bitbake gyt-image-minimal 
```

### After success full compilation of above command output binaries will be generated  “tmp-glibc/deploy/images/qemuriscv64” directory. 

### Step 5: Run qemu

```bash
MACHINE=qemuriscv64 runqemu tmp-glibc/deploy/images/qemuriscv64/gyt-image-minimal-qemuriscv64-2022xxxxxxxxxx.rootfs.cpio.gz nographic qemuparams="-m 512"
```

	

