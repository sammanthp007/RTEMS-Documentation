Writing the tests and running them

## Create the test and config
1. Create a new directory for the test.
2. Create a new Makefile.am. This normally involves a lot of copy and paste from an existing test.
3. Add a new init.c file. Which may be enough for simple tests.
4. Add a test.doc file for the documentation.
5. Add a test.scn file with the output of the test. TODO: probably could add a make-scn to rtems/make/leaf.cfg or rtems/c/src/make/leaf.cfg through a clever call to a simulator and a log file
6. Modify the top level Makefile.am. TODO: probably could add this functionality to rtems/bootstrap
7. Modify the top level configure.ac. TODO: probably could add this functionality to rtems/bootstrap
8. Run bootstrap. TODO: make this an option when running the script in http://git.rtems.org/rtems-testing/tree/rtems-test-template
9. Add a copy and paste .cvsignore. Note: outdated, possibly .gitignore
10. Write a ChangeLog with several entries. Note: replaced ChangeLog with git functionality

The template for test can be found [here](http://git.rtems.org/rtems-testing/tree/rtems-test-template)

## Create the needed files using bootstrap and then config and make
```
cd $HOME/development/rtems/kernel/rtems

./bootstrap -c && ./bootstrap -p && $HOME/development/rtems/rsb/source-builder/sb-bootstrap

cd $HOME/development/rtems/kernel

mkdir erc32

cd erc32

$HOME/development/rtems/kernel/rtems/configure --prefix=$HOME/development/rtems/5 --target=sparc-rtems5 --enable-rtemsbsp=erc32 --enable-posix ENABLE_STRICT_ORDER_MUTEX=1 --enable-tests --disable-networking

gmake all

gmake install
```
