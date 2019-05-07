---
title: "How to Compile Crystal on Ubuntu"
date: 2019-03-25T20:31:58-07:00
draft: true
categories: ["programming"]
tags: ["crystal", "Ubuntu", "bugs"]
summary: If at first you don't succeed, check this post out.
featuredImage: "images/crystal-ubuntu.png"
---

This was tested on Ubuntu 18.04, but should be applicable to most versions.

If you are trying to compile crystal, and `crystal -v` looks like this:
```shell
~❯❯❯ crystal -v
Crystal 0.27.2 [60760a546] (2019-02-05)

LLVM: 4.0.0
Default target: x86_64-unknown-linux-gnu
```

The "LLVM 4.0" thing is what's important. If you install all of crystal's dependencies:
```shell
~❯❯❯ git clone https://github.com/ivmai/bdwgc.git
~❯❯❯ cd bdwgc
~/bdwgc ❯❯❯ git clone https://github.com/ivmai/libatomic_ops.git
~/bdwgc ❯❯❯ autoreconf -vif
~/bdwgc ❯❯❯ ./configure --enable-static --disable-shared
~/bdwgc ❯❯❯ make -j
~/bdwgc ❯❯❯ make check
~/bdwgc ❯❯❯ sudo make install
```
```shell
sudo apt install \
  libbsd-dev \
  libedit-dev \
  libevent-dev \
  libgmp-dev \
  libgmpxx4ldbl \
  libssl-dev \
  libxml2-dev \
  libyaml-dev \
  libreadline-dev \
  automake \
  libtool \
  git \
  llvm \
  llvm-dev \
  libpcre3-dev \
  build-essential -y
```
And then you get this big ugly error when you try to compile it:
```shell
~/s/c/crystal ❯❯❯ make                                                                                                                                                                                V
Using /usr/bin/llvm-config-6.0 [version=6.0.0]
g++ -c  -o src/llvm/ext/llvm_ext.o src/llvm/ext/llvm_ext.cc -I/usr/lib/llvm-6.0/include -std=c++0x -fuse-ld=gold -Wl,--no-keep-files-mapped -Wl,--no-map-whole-files -fPIC -fvisibility-inlines-hidden -
Werror=date-time -std=c++11 -Wall -W -Wno-unused-parameter -Wwrite-strings -Wcast-qual -Wno-missing-field-initializers -pedantic -Wno-long-long -Wno-maybe-uninitialized -Wdelete-non-virtual-dtor -Wno-
comment -ffunction-sections -fdata-sections -O2 -DNDEBUG  -fno-exceptions -D_GNU_SOURCE -D__STDC_CONSTANT_MACROS -D__STDC_FORMAT_MACROS -D__STDC_LIMIT_MACROS
cc -fPIC    -c -o src/ext/sigfault.o src/ext/sigfault.c
ar -rcs src/ext/libcrystal.a src/ext/sigfault.o
CRYSTAL_CONFIG_PATH="/home/kai/src/crystal/crystal/src" CRYSTAL_CONFIG_BUILD_COMMIT="5282fdf91" ./bin/crystal build -D preview_overflow -D compiler_rt  -o .build/crystal src/compiler/crystal.cr -D wit
hout_openssl -D without_zlib
L-L-V-M-5858P-assR-egistry.o: In function `initialize_inst_combine':
/home/kai/src/crystal/crystal/src/llvm/pass_registry.cr:11: undefined reference to `LLVMInitializeInstCombine'
collect2: error: ld returned 1 exit status
Error: execution of command failed with code: 1: `cc "${@}" -o '/home/kai/src/crystal/crystal/.build/crystal'  -rdynamic  /home/kai/src/crystal/crystal/src/llvm/ext/llvm_ext.o `/usr/bin/llvm-config-6.
0 --libs --system-libs --ldflags 2> /dev/null` -lstdc++ -lpcre -lm -lgc -lpthread /home/kai/src/crystal/crystal/src/ext/libcrystal.a -levent -lrt -ldl -L/usr/lib -L/usr/local/lib`
Makefile:122: recipe for target '.build/crystal' failed
make: *** [.build/crystal] Error 1
```
Then it's probably because you're a bad person who is mean to their computer.
No?
Hmm. Must just be me, then.
In seriousness, the way to fix this is to just use the llvm 4.0 compiler and config.
```shell
~/s/c/crystal ❯❯❯ apt install llvm-4.0{,-dev}
```
This installs `llvm-4.0` and `llvm-4.0-dev`, if the syntax isn't clear. I'm not entirely sure the dev package is necessary: if you find out otherwise, drop me a note.
Compilation proceeds thusly:
```
~/s/c/crystal ❯❯❯ make clean crystal LLVM_CONFIG=/usr/bin/llvm-config-4.0                                                                                                                             V
```
Now you can has Crystal! :D

