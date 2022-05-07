# Cross GNU Toolchain - Binutils 2.38
Build and install as almaz user on host system

Required for building GCC

Unpack GNU Binutils sources, and then create build directory

```
mkdir build
cd build
``` 

Configure binutils:

```
export  CARGS="--prefix=/cgnutools "
export CARGS+="--target=${ALMAZ_TARGET} "
export CARGS+="--with-sysroot=/cgnutools/${ALMAZ_TARGET} "
export CARGS+="--disable-compressed-debug-sections "
export CARGS+="--enable-deterministic-archives "
export CARGS+="--disable-multilib "
export CARGS+="--disable-werror "
export CARGS+="--disable-nls "
export CARGS+="--enable-lto --enable-plugins --enable-gold "

../configure ${CARGS}
```

Make binutils working on host system:
```
make configure-host
```

Make and install binutils to cgnutools:
```
make && make install && unset CARGS
```
