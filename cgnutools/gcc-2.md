# Cross GNU Toolchain - GCC 11.2.1 rev 20220219 - pass 2
Build as **almaz** user on host system

```
xz -cd ../pkgs/mpfr-4.1.0.tar.xz | tar -xf -
mv -v mpfr-4.1.0 mpfr
xz -cd ../pkgs/gmp-6.2.1.tar.xz | tar -xf -
mv -v gmp-6.2.1 gmp
gunzip -cd ../pkgs/mpc-1.2.1.tar.gz | tar -xf -
mv -v mpc-1.2.1 mpc
```
Apply patches (from Alpine-linux)
```
for patch in 0001-posix_memalign.patch \
0002-gcc-poison-system-directories.patch \
0003-specs-turn-on-Wl-z-now-by-default.patch \
0004-Turn-on-D_FORTIFY_SOURCE-2-by-default-for-C-C-ObjC-O.patch \
0005-On-linux-targets-pass-as-needed-by-default-to-the-li.patch \
0006-Enable-Wformat-and-Wformat-security-by-default.patch \
0007-Enable-Wtrampolines-by-default.patch \
0009-Ensure-that-msgfmt-doesn-t-encounter-problems-during.patch \
0010-Don-t-declare-asprintf-if-defined-as-a-macro.patch \
0011-libiberty-copy-PIC-objects-during-build-process.patch \
0012-libitm-disable-FORTIFY.patch \
0013-libgcc_s.patch \
0014-nopie.patch \
0015-libffi-use-__linux__-instead-of-__gnu_linux__-for-mu.patch \
0016-dlang-update-zlib-binding.patch \
0017-dlang-use-libucontext-on-mips64.patch \
0018-dlang-libdruntime-define-fcntl.h-constants-for-mips6.patch \
0019-ada-fix-shared-linking.patch \
0020-build-fix-CXXFLAGS_FOR_BUILD-passing.patch \
0021-add-fortify-headers-paths.patch \
0024-mips64-disable-multilib-support.patch \
0025-aarch64-disable-multilib-support.patch \
0026-s390x-disable-multilib-support.patch \
0027-ppc64-le-disable-multilib-support.patch \
0028-x86_64-disable-multilib-support.patch \
0029-riscv-disable-multilib-support.patch \
0030-always-build-libgcc_eh.a.patch \
0031-ada-libgnarl-compatibility-for-musl.patch \
0032-ada-musl-support-fixes.patch \
0033-gcc-go-Fix-handling-of-signal-34-on-musl.patch \
0034-There-are-more-than-one-st_-a-m-c-tim-fields-in-stru.patch \
0035-gcc-go-signal-34-is-special-on-musl-libc.patch \
0036-gcc-go-undef-SETCONTEXT_CLOBBERS_TLS-in-proc.c.patch \
0037-gcc-go-link-to-libucontext.patch \
0038-Use-generic-errstr.go-implementation-on-musl.patch \
0039-configure-Add-enable-autolink-libatomic-use-in-LINK_.patch \
0040-configure-fix-detection-of-atomic-builtins-in-libato.patch \
0041-libgo-Recognize-off64_t-and-loff_t-definitions-of-mu.patch \
0042-Fix-attempt-to-use-poisoned-calloc-error-in-libgccji.patch \
0043-stddef.h-add-support-for-musl-typedef-macro-guards.patch \
0044-gcc-go-Use-int64-type-as-offset-argument-for-mmap.patch \
0045-libgo-include-asm-ptrace.h-for-pt_regs-definition-on.patch \
0046-Disable-fsplit-stack-support-on-non-glibc-targets.patch \
0047-x86-Properly-disable-fsplit-stack-support-on-non-gli.patch \
0048-gdc-unconditionally-link-libgphobos-against-libucont.patch \
0049-properly-disable-fsplit-stack-on-non-glibc-targets-P.patch \
0050-x86-Fix-fsplit-stack-feature-detection-via-TARGET_CA.patch \
0051-go-gospec-forcibly-disable-fsplit-stack-support.patch \
0023-DP-Use-push-state-pop-state-for-gold-as-well-when-li.patch ; do
patch -Np1 -i ../patches/gcc-11.2.1-alpine/$patch ;
done
```
Configure in a dedicated build directory
```
mkdir build && cd  build
```
Set configure flags
```
export  CARGS="--prefix=/cgnutools "
export CARGS+="--build=${ALMAZ_HOST} "
export CARGS+="--host=${ALMAZ_HOST} "
export CARGS+="--target=${ALMAZ_TARGET} "
export CARGS+="--enable-shared "
export CARGS+="--enable-threads=posix "
export CARGS+="--enable-libstdcxx-time "
export CARGS+="--enable-clocale=generic "
export CARGS+="--enable-languages=c,c++ "
export CARGS+="--enable-fully-dynamic-string "
export CARGS+="--with-sysroot=/cgnutools "
export CARGS+="--disable-libsanitizer "
export CARGS+="--disable-multilib "
export CARGS+="--disable-symvers "
export CARGS+="--disable-libsspc "
export CARGS+="--disable-libssp "
export CARGS+="--disable-static "
export CARGS+="--disable-nls "
export CARGS+="--enable-lto "
export CARGS+="--with-ppl --with-cloog "
```
Configure source
```
AR=ar LDFLAGS="-Wl,-rpath,/cgnutools/lib" \
../configure ${CARGS}
```
Build
```
make AS_FOR_TARGET="${ALMAZ_TARGET}-as" \
    LD_FOR_TARGET="${ALMAZ_TARGET}-ld"
```
Install
```
make install
```
Now adjust GCC to produce programs and libraries that will use musl libc in /cgnutools...
```
export SPECFILE=`dirname $(${ALMAZ_TARGET}-gcc -print-libgcc-file-name)`/specs
${ALMAZ_TARGET}-gcc -dumpspecs > specs
sed -i 's/\/lib\/ld-musl-x86_64.so.1/\/cgnutools\/lib\/ld-musl-x86_64.so.1/g' specs
```
Check specs file
```
grep "/cgnutools/lib/ld-musl-x86_64.so.1" specs  --color=auto
```
Install specs file
```
mv -v specs $SPECFILE
unset SPECFILE
```
Check if GCC compiles as intended:
```
echo "int main(){}" > dummy.c
${ALMAZ_TARGET}-gcc dummy.c
readelf -l a.out | grep Requesting
```
Should output:
```
 [Requesting program interpreter: /cgnutools/lib/ld-musl-x86_64.so.1]
```
