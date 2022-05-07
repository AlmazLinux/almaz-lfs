# Cross GNU Toolchain - Libexecinfo 1.1
Build and install as **almaz** user on host system
**!** Required for musl-based host

Fix the broken build system
```
patch -Np1 -i ../patches/libexecinfo-alpine/10-execinfo.patch
patch -Np1 -i ../patches/libexecinfo-alpine/20-define-gnu-source.patch
patch -Np1 -i ../patches/libexecinfo-alpine/30-linux-makefile.patch
```
Build with freshly built GCC
```
CC=${ALMAZ_TARGET}-gcc AR=${ALMAZ_TARGET}-ar CFLAGS=" -fno-omit-frame-pointer" make
```
Install to cgnutools
```
install -D -m755 -v execinfo.h           /cgnutools/include/execinfo.h     && \
install -D -m755 -v stacktraverse.h      /cgnutools/include/stacktraverse.h && \
install -D -m755 -v libexecinfo.a        /cgnutools/lib/libexecinfo.a      && \
install -D -m755 -v libexecinfo.so.1     /cgnutools/lib/libexecinfo.so.1   && \
ln -sv /cgnutools/lib/libexecinfo.so.1   /cgnutools/lib/libexecinfo.so
```
