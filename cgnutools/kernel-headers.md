# Cross GNU Toolchain - Linux 5.17.0 Kernel Headers
Build as **almaz** user on host system

Kernel version greater than 5.9.16 will not compile under musl 
```
patch -Np1 -i ../patches/kernel/include-uapi-linux-swab-Fix-potentially-missing-__always_inline.patch
```
Clean the source

```
make mrproper
```

Build the headers

```
ARCH=${ALMAZ_ARCH} make headers
```
Install the headers

```
find usr/include -name ".*" -exec rm -vf {} \;
mkdir -pv /cgnutools/${ALMAZ_TARGET}/include
cp -rv usr/include/* /cgnutools/${ALMAZ_TARGET}/include/
rm -v /cgnutools/${ALMAZ_TARGET}/include/Makefile
```
