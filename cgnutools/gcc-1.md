# Cross GNU Toolchain - GCC 11.2.1 rev 20220219 - pass 1
Build and install as **almaz** user on host system

GCC now requires the GMP, MPFR and MPC packages to build.
Unpack them in-tree.

```
xz -cd ../pkgs/mpfr-4.1.0.tar.xz | tar -xf -
mv -v mpfr-4.1.0 mpfr
xz -cd ../pkgs/gmp-6.2.1.tar.xz | tar -xf -
mv -v gmp-6.2.1 gmp
gunzip -cd ../pkgs/mpc-1.2.1.tar.gz | tar -xf -
mv -v mpc-1.2.1 mpc
```

The GCC documentation recommends building GCC in 
a dedicated build directory
```
mkdir -v build && cd  build
```
Set the configure flags
```
export  CARGS="--prefix=/cgnutools "
export CARGS+="--build=${ALMAZ_HOST} "
export CARGS+="--host=${ALMAZ_HOST} "
export CARGS+="--target=${ALMAZ_TARGET} "
export CARGS+="--with-sysroot=/cgnutools/${ALMAZ_TARGET} "
export CARGS+="--with-arch=${ALMAZ_CPU} "
export CARGS+="--with-newlib "
export CARGS+="--disable-nls "
export CARGS+="--disable-libvtv "
export CARGS+="--disable-libitm "
export CARGS+="--disable-libssp "
export CARGS+="--disable-shared "
export CARGS+="--disable-libgomp "
export CARGS+="--without-headers "
export CARGS+="--disable-threads "
export CARGS+="--disable-multilib "
export CARGS+="--disable-libatomic "
export CARGS+="--disable-libstdcxx "
export CARGS+="--disable-libquadmath "
export CARGS+="--disable-libsanitizer "
export CARGS+="--disable-decimal-float "
export CARGS+="--enable-clocale=generic "
export CARGS+="--enable-languages=c "
export CARGS+="--enable-lto "
export CARGS+="--with-ppl --with-cloog "
```
Configure source 
```
CFLAGS='-g0 -O0' \
CXXFLAGS=$CFLAGS \
../configure ${CARGS}
```

Build only the minimum of gcc, full version we will make after that we would have built musl
```
make all-gcc all-target-libgcc
```
Install to toolchain
```
make install-gcc install-target-libgcc && unset CARGS
```
