# RTEMS
This is my documentation as I am exploring developing for RTEMS. 
The original documentation can be found in [Original Documentation](https://ftp.rtems.org/pub/rtems/people/chrisj/docs/user/start/index.html).

[All documentation for RTEMS](https://docs.rtems.org) are also available under one link.


Everything done here is done in a Linux machine. 

## Table of Content
- ### [Introduction and Building]
  - [What is rtems and how it works](#theory)
  - [Configure the dev environment](#configure-the-development-environment)
    - [Create a sandbox environment](#create-a-sandbox-environment)
    - [Prerequisite](#prerequisite)
  - [Setting up with RTEMS Source Builder RSB](#setting-up-with-rtems-source-builder-rsb)
    - [Download the source for rsb](#download-the-source-for-rsb)
    - [Check if your computer can support building the toolchain](#check-if-your-computer-can-support-building-the-toolchain)
    - [Build Toolchains](#build-toolchains)
  - [Build RTEMS](#build-rtems)
    - [Download the RTEMS OS](#download-the-rtems-os)
    - [Set up the path](#set-up-the-path)
    - [Build BSPs](#build-bsps)
  - [Running programs](#running-programs)
  - [Useful commands](#useful-commands)
  - [Troubleshooting](#troubleshooting)
    - [TypeError: cannot concatenate str and general objects](#typeerror-cannot-concatenate-str-and-general-objects)
    - [Missing Packages or Broken Mirror](#missing-packages-or-broken-mirror)
- ### [Project: Paravirtuatization](./Topics/paravirtualization.md)
- ### [Project: Priority Inversion](./Topics/project_priority_inversion/README.md)
- ### [Project: Moving on to compiling bsp for tm4c129e TI's TM4C arm architecture ](#moving-on-to-compiling-bsp-for-tm4c129e)

## Diagram of how rtems is organized
![RTEMS Organization](Topics/images/architecture_diagram_dr_bloom_office_hours.jpg)

The `<root-dir>/rsb` directory contains the RSB.
The `<root-dir>/4.12` directory contains the build set for each architecture we are running RTEMS on.

## Theory
RTEMS Source Builder (RSB): 
- is a tool to aid building packages from source used by the RTEMS project.
- Has two types of configuration data
    - A build set:
        -  A build set describes a collection of packages that define a set of
           tools you would use when developing software for RTEMS. For example
           the basic GNU tool set is binutils, gcc, and gdb and is the typical
           base suite of tools you need for an embedded cross-development type
           project. 
    - Configuration Files:
        - They define how a package is built.
        - They are scripts that: 
            - Prepare
            - Build
            - Install
            packages

RSB installs into a build *prefix*.
Prefix: 
    - the top of a group of directories containing all the files needed to develop RTEMS applications. 

RSB is a builder for toolchains. Toolchains are required for rtems architecture as shown in the RTEMS Organization diagram above

## Configure the development environment
### Create a sandbox environment
```
$ cd
$ mkdir -p development/rtems
$ cd development/rtems
```
### Prerequisite
Download the following packages. Debian packages are mentioned here.
```
sudo apt update
sudo apt upgrade
sudo apt install bison texinfo flex make binutils gcc g++ gdb unzip git python2.7-dev zlib1g-dev libncurses5-dev
```
## Setting Up With RTEMS Source Builder RSB
### Download the source for rsb 
```
$ git clone git://git.rtems.org/rtems-source-builder.git rsb
```

### Check if your computer can support building the toolchain
```
$ cd rsb
$ ./source-builder/sb-check
```
Continue only if your output is similar to:
```
RTEMS Source Builder - Check, 4.12 (4c5eb8969451)
Environment is ok
```
### Build Toolchains
We build toolchains (the required tools/programs to build a runnable RTEMS OS on an computer architecture) for any architecture we want.
Here we build the toolchains for sparc, arm, powerpc, and i386 architecture, and also a qemu virtualization so we can work with a virtual machine that has RTEMS OS on top of qemu virtualization.
```
$ cd $HOME/development/rtems/rsb/rtems
$ ../source-builder/sb-set-builder --prefix=$HOME/development/rtems/4.12 4.12/rtems-sparc
$ ../source-builder/sb-set-builder --prefix=$HOME/development/rtems/4.12 4.12/rtems-arm
../source-builder/sb-set-builder --prefix=$HOME/development/rtems/4.12 4.12/rtems-powerpc
$ ../source-builder/sb-set-builder --prefix=$HOME/development/rtems/4.12 4.12/rtems-i386
$ ../source-builder/sb-set-builder --prefix=$HOME/development/rtems/4.12 --without-rtems devel/qemu
```

Now, we have the required tools (gcc, gdb, ...) for the architecture mentioned above. Let's get the RTEMS source code so we can build a system with RTEMS running on a specific architecture.

## Build RTEMS
### Download the RTEMS OS
```
cd $HOME/development/rtems
mkdir kernel
cd kernel
git clone git://git.rtems.org/rtems.git rtems
```

### Set up the path
```
export PATH=$HOME/development/rtems/4.12/bin:$PATH #Or add to your PATH variable.
```

### Initialize the default files (.ac and .am to executables) in the rtems source
This step initializes all the default files required. **Only run this the first time, or when we add new files. Do not run this when we update any file in RTEMS source.**
```
cd $HOME/development/rtems/kernel/rtems
./bootstrap -c && ./bootstrap -p && $HOME/development/rtems/rsb/source-builder/sb-bootstrap
```
bootstrap -c is for cleaning all the remaining make and config files
bootstrap -p is for doing smt with the preinstalled things
and we use the sb-bootstrap from the previously received rsb for building the make and config files from .ac and .am files

### Build BSPs
```
# for erc/sparc
cd $HOME/development/rtems/kernel
# build the sparc/erc32 machine
# create a directory to keep the bsp for sparc/erc32
mkdir erc32
cd erc32
$HOME/development/rtems/kernel/rtems/configure --prefix=$HOME/development/rtems/4.12 --target=sparc-rtems4.12 --enable-rtemsbsp=erc32 --enable-posix
make all
make install

# build for arm/b_realview_arm
cd $HOME/development/rtems/kernel
mkdir b-realview_pbx_a9_qemu
cd b-realview_pbx_a9_qemu
# Compile the realview-pbx-arm machine's kernel files
$HOME/development/rtems/kernel/rtems/configure --target=arm-rtems4.12 --disable-networking --enable-rtemsbsp=realview_pbx_a9_qemu --enable-posix --prefix=$HOME/development/rtems/4.12
make all
make install

# build for i386/pc386
cd $HOME/development/rtems/kernel
mkdir pc386
cd pc386
# compile the pc386 machine's kernel files (build rtems for pc386)
HOME/development/rtems/kernel/rtems/configure --target=i386-rtems4.12 --prefix=$HOME/development/rtems/4.12 --enable-rtemsbsp=pc386 USE_COM1_AS_CONSOLE=1 BSP_PRESS_KEY_FOR_RESET=0

# Notes on the USE_COM1_AS_CONSOLE=1 and BSP_PRESS_KEY_FOR_RESET=0 options can be found in https://devel.rtems.org/wiki/Developer/Simulators/QEMU#Usingthertems-testingModule

# build for powerpc/psim
cd $HOME/development/rtems/kernel
mkdir psim
cd psim
# Compile the psim machine's kernel files
$HOME/development/rtems/kernel/rtems/configure --target=powerpc-rtems4.12 --enable-rtemsbsp=psim --enable-posix --prefix=$HOME/development/rtems/4.12
make all
make install
```

> In english, the command above says (I am guessing here), compile RTEMS for arm-rtems4.12, using the tools from --prefix with the added options of disabling networking.

We need these to confirm that nothing is wrong with the rtmes or toolchain in case we have errors for tm4c129e


### Running Programs

Running `ticker.exe` for sparc
```
# run from the root directory for the rtems build for sparc
sparc-rtems4.12-run -v `find . -name ticker.exe`
```

Running `ticker.exe` for arm
```
# run from the root directory for the rtems build for arm
QEMU_AUDIO_DRV=none qemu-system-arm -no-reboot -net none -M realview-pbx-a9 -m 256M -serial stdio -kernel `find . -name ticker.exe`
```

Running `ticker.exe` for i386
```
# run from the root directory for the rtems build for i386
qemu-system-i386 -kernel `find . -name ticker.exe` -append "--console=/dev/com1" -serial stdio
```

Since I was not able to run `ticker.exe`, I ran `rtems-test` in `rtems-tools`. 
[Link to RTEMS tester](ftp://ftp.rtems.org/pub/rtems/people/chrisj/rtems-tester/rtems-tester.html) - ftp://ftp.rtems.org/pub/rtems/people/chrisj/rtems-tester/rtems-tester.html
```
cd $HOME/development/rtems/rsb/.../rtems-tools/
rtems-test --rtems-bsp=psim-run --rtems-tools=$HOME/development/rtems/4.12 \ /home/rtemsuser/development/rtems/kernel/psim/powerpc-rtems4.12/c/psim/testsuites
```


# Moving on to compiling bsp for tm4c129e
What I have done so far:

### Create a new directory and get Habeebs code that claims to have built a bsp for TM4C129E board
```
cd ~/development/
git clone https://github.com/Dipupo/rtems.git haheeb-rtems
```

### Create the patch files of the commits that habeeb implemented
```
cd ~/development/habeeb-rtems
git format-patch b30ab25 --stdout > ~/development/rtems/kernel/rtems/habeeb_diff.diff
```
### apply the patch files created
```
cd ~/development/rtems/kernel/rtems
git apply --reject habeeb_diff.diff
```

In my case, I got two .rej files. On inspection, I only needed to delete them because they were implemented, probably in another commit as well.

### compile the bsp
```
cd ~/development/rtems/kernel
cd rtems
./bootstrap -c && ./bootstrap -p && $HOME/development/rtems/rsb/source-builder/sb-bootstrap

cd ..
mkdir tm4c129e
cd tm4c129e
$HOME/development/rtems/kernel/rtems/configure --target=arm-rtems4.12 --disable-networking --enable-rtemsbsp=tm4c129e --enable-posix --prefix=$HOME/development/rtems/4.12

make all
make install
```
Here, ```make all``` gives the error as mentioned in [Error file](https://github.com/sammanthp007/build-rtems-documentation/blob/master/make%20error%20tm4c129e)

## Useful Commands

List all bsps
```
$ $HOME/development/rtems/kernel/rtems/rtems-bsps
```

List all architectures
```
$ $HOME/development/rtems/rsb/source-builder/sb-set-builder --list-bsets
```

## Troubleshooting
#### TypeError: cannot concatenate str and general objects
```
TypeError cannot concatenate 'str' and 'general' objects

# Soln:
export the path
```

### For installing 4.10
#### error: building sparc-rtems4.10-gcc-4.4.7-newlib-1.18.0-x86_64-linux-gnu-1
```
package: sparc-rtems4.10-gcc-4.4.7-newlib-1.18.0-x86_64-linux-gnu-1
building: sparc-rtems4.10-gcc-4.4.7-newlib-1.18.0-x86_64-linux-gnu-1
error: building sparc-rtems4.10-gcc-4.4.7-newlib-1.18.0-x86_64-linux-gnu-1
Build FAILED
  See error report: rsb-report-sparc-rtems4.10-gcc-4.4.7-newlib-1.18.0-x86_64-linux-gnu-1.txt
error: building sparc-rtems4.10-gcc-4.4.7-newlib-1.18.0-x86_64-linux-gnu-1
```
Solution:
I changed the version of my computer's gcc to 4.7 (not 4.4.7, because i could not find a way to get it through apt, and I did not want to install the binaries and install).
```
# add the trusty universe to apt. Open /etc/apt/sources.list and add the following two lines:
deb http://archive.ubuntu.com/ubuntu/ trusty universe
deb-src http://archive.ubuntu.com/ubuntu/ trusty universe

# then run
sudo apt install gcc-4.7

# then update-alternative
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.7 20
sudo update-alternatives --install /usr/bin/cc cc /usr/bin/gcc 30
sudo update-alternatives --set cc /usr/bin/gcc

sudo update-alternatives --config gcc


# Now try to build the toolchain
```
- I was able to understand the error in the error file about missing [toplev.o] [here](https://unix.stackexchange.com/a/219709)
- I was able to then know how to add a repo to apt from [here](https://askubuntu.com/a/227785) which in my case is mentioned above
- And eventhough it was 4.7 but i needed 4.4.7, I was confident that this would work because it was mentioned [here](https://stackoverflow.com/a/29187423/4867689)
- And I learned about updating alternatives of a tool [here](https://askubuntu.com/a/26518)

Furthermore, if this had not worked, I would have tried to run the pathes mentioned in [this email thread](https://lists.rtems.org/pipermail/devel/2016-May/015127.html) which was referred to in [this ticker[(https://devel.rtems.org/ticket/2724)

And one last resource for [gcc newlib debugging(https://devel.rtems.org/wiki/Debugging/Newlib) that I did not know what to make of was also found.


#### Missing Packages or Broken Mirror
If some packages are missing, you may need to download the zip from somewhere. I have some save in google drive since the mirrors seem to be broken. You can download the required packages at [Drive](https://drive.google.com/file/d/0B_ZAkr8A2WagRkJEQUZhOWVtY3M/view?usp=sharing)
