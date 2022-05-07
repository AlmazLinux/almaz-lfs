# Cross GNU Toolchain - Musl C Library 1.2.3
Build as **almaz** user on host system

Configure with freshly built GCC
```
./configure \
  CROSS_COMPILE=${ALMAZ_TARGET}- \
  --prefix=/ \
  --target=${ALMAZ_TARGET}
```
Build and install
```
make && make DESTDIR=/cgnutools install
```
GCC will look for system headers in /cgnutools/usr/include. Create directory and symlink:
```
mkdir -pv /cgnutools/usr
ln -sv ../include /cgnutools/usr/include
```
Fix a symlink
```
rm -v /cgnutools/lib/ld-musl-$(uname -m).so.1
ln -sv libc.so /cgnutools/lib/ld-musl-$(uname -m).so.1
```
Create a symlink that can be used to print the required shared objects of a program or shared object
```
mkdir /cgnutools/etc
ln -sv ../lib/libc.so /cgnutools/bin/ldd
```
Configure the dynamic linker
```
cat > /cgnutools/etc/ld-musl-$(uname -m).path <<EOF
/cgnutools/lib
/cgnutools/${ALMAZ_TARGET}/lib
/usr/lib64
/lib64
/usr/lib
/lib
EOF
```
