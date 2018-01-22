# Project: Priority Inversion

## Table of Content
- [Installation for master on posix supported OS (MacOS in my case)](#installation)
- [Summary of Key Concepts from C User Guide for RTEMS](./key_concepts.md)
- [Summary of Task State Transition and Scheduling Manager Directives from C User Guide for RTEMS](./scheduling_concepts.md)
- [Summary of Initailization Manager with Directives from C User Guide for RTEMS](./initialization_manager.md)
- [Summary of Task Manager with Directives from C User Guide for RTEMS](./task_manager.md)

## INSTALLATION

## Build the toolchain
```
# create a sandbox
mkdir -p ~/development/rtems
cd ~/development/rtems

# get the latest rsb
git clone git://git.rtems.org/rtems-source-builder.git rsb

# [OPTIONAL] make sure it is the master branch or whichever you want to work on
git checkout master

cd rsb

# check if your system is fit to be develop rtems on
./source-builder/sb-check

# ONLY CONTINUE IF OUTPUT IS:
RTEMS Source Builder - Check, 4.12 (4c5eb8969451)
Environment is ok

# build the rsb toolchain
../source-builder/sb-set-builder --prefix=$HOME/development/rtems/5 --without-rtems 5/rtems-sparc
```


## Build RTEMS
```
export PATH=$HOME/development/rtems/5/bin:$PATH

cd $HOME/development/rtems
mkdir kernel
cd kernel
git clone git://git.rtems.org/rtems.git rtems

# config the .am or .ac files
cd $HOME/development/rtems/kernel/rtems
./bootstrap -c && ./bootstrap -p && $HOME/development/rtems/rsb/source-builder/sb-bootstrap

cd $HOME/development/rtems/kernel
mkdir erc32
cd erc32
$HOME/development/rtems/kernel/rtems/configure --prefix=$HOME/development/rtems/5 --target=sparc-rtems5 --enable-rtemsbsp=erc32 --enable-posix ENABLE_STRICT_ORDER_MUTEX=1 --enable-tests --disable-networking 

# for 4.10
$HOME/development/rtems/kernel/rtems/configure --prefix=$HOME/development/rtems/4.10 --target=sparc-rtems4.10 --enable-rtemsbsp=erc32 --enable-posix ENABLE_STRICT_ORDER_MUTEX=1 --enable-tests --disable-networking

# for 4.11
$HOME/development/rtems/kernel/rtems/configure --prefix=$HOME/development/rtems/4.11 --target=sparc-rtems4.11 --enable-rtemsbsp=erc32 --enable-posix ENABLE_STRICT_ORDER_MUTEX=1 --enable-tests --disable-networking
```

## Run Tests
```
# run from samples
sparc-rtems5-run -v `find . -name ticker.exe`

# run from sptests
sparc-rtems5-run -v `find . -name sp04.exe`

