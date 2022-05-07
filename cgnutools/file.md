# Cross GNU Toolchain - File 5.41
Build and install as **almaz** user on host system

Configure source with freshly built GCC
```
CC="${ALMAZ_TARGET}-gcc"         \
CXX="${ALMAZ_TARGET}-g++"        \
AR="${ALMAZ_TARGET}-ar"          \
AS="${ALMAZ_TARGET}-as"          \
RANLIB="${ALMAZ_TARGET}-ranlib"  \
LD="${ALMAZ_TARGET}-ld"          \
STRIP="${ALMAZ_TARGET}-strip"    \
./configure --prefix=/cgnutools --disable-libseccomp
```
Build and install
```
make && make install
```
