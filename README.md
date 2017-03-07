# rtems
Documentation for the steps taken while building RTEMS in my x86 computer for TI's TM4C arm architecture 

### Create a sandbox environment
```
$ cd
$ mkdir -p development/rtems
$ cd development/rtems
```

### Get the rsb, a builder for toolchains, toolchains required for rtems architecture as shown in the diagram above
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
# Build toolchains for arm and sparc architectures
```
$ cd rtems
$ ../source-builder/sb-set-builder --prefix=$HOME/development/rtems/4.12 4.12/rtems-sparc
$ ../source-builder/sb-set-builder --prefix=$HOME/development/rtems/4.12 4.12/rtems-arm
```
