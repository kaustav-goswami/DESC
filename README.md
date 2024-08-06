# DESC
This is mostly a documentation of DESC.

## Getting Started

DESC is based on the following softwares.
Note that not all of the following softwares/libraries have a valid
installation candidates.

* autoconf
* automake
* autotools-dev
* bc
* bison
* build-essential
* curl **[no installation candidate]**
* expat
* libexpat1-dev
* libexpat-dev
* flex
* gawk
* gcc
* git
* gperf
* libgmp-dev 
* libmpc-dev
* libmpfr-dev
* libtool
* texinfo
* tmux
* patchutils
* zlib1g-dev
* wget
* bzip2
* patch
* vim-common
* lbzip2
* python **[no installation candidate]**
* pkg-config
* libglib2.0-dev
* libpixman-1-dev
* libssl-dev
* screen
* device-tree-compiler
* expect
* makeself **Newer versions are not supported. See below** 
* unzip 
* cpio
* rsync
* cmake
* p7zip-full

The compilation might complain about `cmake` path, if installed using pip.
The gcc version used was 11.4.0.
I also tried using clang but it ran into *some* linking issues.

Follow the following steps to build and test DESC.
1. Clone the original documention for DESC from the link shared via email.
   **Note** See the common issues section before starting to work with the
   instructions.
2. Follow the updated README in that repository.

## Common Issues

There were several issues that I encountered while compiling keystone/DESC.
Here are some of the common ones.
Here are some of the issues and a potential way to fix these issues.
Here is the assumed directory structure.

```
  DESC (documentation, this repository)
  |
  |__ DESC (cloned in step 1)
  |
  |__ Patches
```

1. `qemu` version used for compiling keystone/DESC is old and only gets
   compiled without modifications when using gcc v 9.4.xx (maybe). Therefore,
   once keystone is cloned and `fast-setup.sh` is executed, make sure to
   reconfigure `qemu` by:
   ```sh
   cd DESC/keystone/qemu
   ./configure --disable-werror --target-list=riscv64-softmmu,riscv64-linux-user
   ```
   `riscv` or `riscv64` depends on the value of `BITS` in the `fast-setup.sh`
   script. Make sure to use the correct value there.

2. Make sure that the environment variable `KEYSTONE_SDK_DIR` is set correctly,
   i.e., `</path/to/the/top/DESC/directory/>/DESC/DESC/keystone/sdk/build64`.
   This is important to distinguish between keystone and DESC. The
   `fast-setup.sh` script will fix this value the first time when the
   cross-compiler-toolchain will be setup. Afterwards, it'll just use that
   initial directory for all future builds.

3. Once the initial stuff is set, start building keystone/DESC in the build
   directory. The build is likely to fail when build eyrie-rt stuff. `makeself`
   appends a file path in the /tmp directory, which fails due to permission
   issues (See
   [https://github.com/megastep/makeself/issues/155](https://github.com/megastep/makeself/issues/155)
   for more details). You'd need to downgrade `makeself` to make it work.
   ```sh
   cd DESC/DESC/keystone
   git clone git@github.com:megastep/makeself.git
   cd makeself
   git checkout release-2.2.0
   ```
   You'd manually need to modify the CMake make files to reflect these changes
   while compilation. This can be done by:
   ```sh
   cd DESC/DESC/keystone/build
   # Change all instance of /usr/bin/makeself to </path/to/the/top/DESC/directory/>/DESC/DESC/keystone/makeself/makeself.sh
   # in the files:
   # 1. examples/tests/CMakeFiles/test-package.dir/build.make
   # 2. examples/hello-native/CMakeFiles/hello-native-package.dir/build.make
   # 3. examples/hello/CMakeFiles/hello-package.dir/build.make
   ```
   Continue building.

4. There are a couple of errors that the compiler will complain about, which
   can be fixed usnig the patches in the repository.

   1. `c-stack.h` and `c-stack.c`
      ```sh
      cp Patches/head.patch DESC/DESC/keystone/build/buildroot.build/build/host-m4-1.4.18/
      cp Patches/body.patch DESC/DESC/keystone/build/buildroot.build/build/host-m4-1.4.18/
      cd DESC/DESC/keystone/build/buildroot.build/build/host-m4-1.4.18/
      patch -u -b lib/c-stack.h -i head.patch
      patch -u -b lib/c-stack.c -i body.patch
      ```
   2. `libfakeroot.c`
      ```sh
      cp Patches/lfr.patch DESC/DESC/keystone/build/buildroot.build/build/host-fakeroot-1.25.3/
      cd DESC/DESC/keystone/build/buildroot.build/build/host-fakeroot-1.25.3/
      patch -u -b libfakeroot.c -i lfr.patch
      ```

## Other Issues
None at the moment.
