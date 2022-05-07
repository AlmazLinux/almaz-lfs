# Cross GNU Toolchain - LLVM/Clang 14.0.0
Build as **almaz** user on host system
**!** Build stage0 clang/clang++ with compiler-rt, lld, libunwind, libcxxabi, and libcxx in LLVM source directory

Truncate directory name for easier typing... 
```
cd $ALMAZ/sources       && \
mv llvm-14.0.0.src llvm && \
export LLVMSRC=${ALMAZ}/sources/llvm && \

cd ${LLVMSRC}
```
make sure compiler-rt, libcxx, libcxxabi, libunwind are unpacked in ${{LLVMSRC}/projects and clang & lld in ${LLVMSRC}/tools.
```
cd projects && \
xz -cd ../../pkgs/compiler-rt-14.0.0.src.tar.xz | tar xf - && \
mv compiler-rt-14.0.0.src compiler-rt           	   
xz -cd ../../pkgs/libcxx-14.0.0.src.tar.xz | tar xf -	   && \
mv libcxx-14.0.0.src libcxx                     	   
xz -cd ../../pkgs/libcxxabi-14.0.0.src.tar.xz | tar xf -   && \
mv libcxxabi-14.0.0.src libcxxabi 			   
xz -cd ../../pkgs/libunwind-14.0.0.src.tar.xz | tar xf -   && \
mv libunwind-14.0.0.src libunwind               	   
cd ${LLVMSRC}/tools 					   && \
xz -cd ../../pkgs/clang-14.0.0.src.tar.xz | tar xf -	   && \
mv clang-14.0.0.src clang				   
xz -cd ../../pkgs/lld-14.0.0.src.tar.xz | tar xf - 	   && \
mv lld-14.0.0.src lld
```
Apply patches from Void Linux
```
cd ${LLVMSRC}/projects/compiler-rt
for i in ../../../patches/llvm14-almaz/compiler-rt-sanitizer-ppc64-musl.patch \
	../../../patches/llvm14-almaz/compiler-rt-xray-ppc64-musl.patch; do
	patch -Np1 -i $i 
done
```
```
cd ${LLVMSRC}/projects/libcxx
for i in ../../../patches/llvm14-almaz/libcxx-musl.patch \
	../../../patches/llvm14-almaz/libcxx-ppc.patch \
	../../../patches/llvm14-almaz/libcxx-ssp-nonshared.patch; do
	patch -Np1 -i $i 
done
```
```
cd ${LLVMSRC}/projects/libcxx
for i in ../../../patches/llvm14-almaz/libcxx-dl.patch; do
	patch -Np1 -i $i
done
```
```
cd ${LLVMSRC}/projects/libcxxabi
for i in ../../../patches/llvm14-almaz/libcxxabi-dl.patch; do
	patch -Np1 -i $i
done
```
```
cd ${LLVMSRC}/projects/libunwind
patch -Np1 -i ../../../patches/llvm14-almaz/libunwind-ppc32.patch
```
```
cd ${LLVMSRC}/tools/clang
for i in ../../../patches/llvm14-almaz/clang-001-fix-unwind-chain-inclusion.patch \
	../../../patches/llvm14-almaz/clang-002-add-musl-triples.patch \
	../../../patches/llvm14-almaz/clang-003-ppc64-dynamic-linker-path.patch \
 	../../../patches/llvm14-almaz/clang-004-ppc64-musl-elfv2.patch; do
	patch -Np1 -i $i
done
```
```
cd ${LLVMSRC}
for i in ../patches/llvm14-almaz/llvm-001-musl.patch \
	../patches/llvm14-almaz/llvm-002-musl-ppc64-elfv2.patch \
	../patches/llvm14-almaz/llvm-003-ppc-secureplt.patch \
	../patches/llvm14-almaz/llvm-004-override-opt.patch \
	../patches/llvm14-almaz/llvm-005-ppc-bigpic.patch \
	../patches/llvm14-almaz/llvm-006-aarch64-mf_exec.patch; do
	patch  -Np1 -i $i
done
```

Disable sanitizers for musl systems, per Void Linux... fixes early build failure
```
sed -i 's/set(COMPILER_RT_HAS_SANITIZER_COMMON TRUE)/set(COMPILER_RT_HAS_SANITIZER_COMMON FALSE)/' projects/compiler-rt/cmake/config-ix.cmake
```
Fix missing header for lld, https://bugs.llvm.org/show_bug.cgi?id=49228
```
mkdir -pv tools/lld/include/mach-o
cp -v projects/libunwind/include/mach-o/compact_unwind_encoding.h tools/lld/include/mach-o
```
Set flags to greatly reduce debugging symbols
```
CFLAGS=' -g -g1'
CXXFLAGS=$CFLAGS 
export CFLAGS CXXFLAGS
```
Update host/target triple detection
```
cp -v ../files/config.guess-musl cmake/config.guess
```
Set the build options ..
```
export  CONFIG_OPTIONS="-DCMAKE_BUILD_TYPE=Release "
export CONFIG_OPTIONS+="-DBUILD_SHARED_LIBS=ON "
export CONFIG_OPTIONS+="-DLLVM_ENABLE_LIBCXX=ON "
export CONFIG_OPTIONS+="-DLLVM_TARGET_ARCH=X86 "
export CONFIG_OPTIONS+="-DLLVM_TARGETS_TO_BUILD=X86 "
export CONFIG_OPTIONS+="-DLIBCXX_HAS_MUSL_LIBC=ON "
```
Set the compiler and linker flags...
```
export LINKERFLAGS="-Wl,-dynamic-linker /cgnutools/lib/ld-musl-x86_64.so.1"
export  CONFIG_TOOLS="-DCMAKE_C_COMPILER=${ALMAZ_TARGET}-gcc "
export CONFIG_TOOLS+="-DCMAKE_CXX_COMPILER=${ALMAZ_TARGET}-g++ "
export CONFIG_TOOLS+="-DCLANG_DEFAULT_LINKER=/cgnutools/bin/ld.lld "
```
Set the tuples...
```
export  CONFIG_TUPLES="-DLLVM_DEFAULT_TARGET_TRIPLE=x86_64-pc-linux-musl " 
export CONFIG_TUPLES+="-DLLVM_HOST_TRIPLE=x86_64-pc-linux-musl " 
export CONFIG_TUPLES+="-DCOMPILER_RT_DEFAULT_TARGET_TRIPLE=x86_64-pc-linux-musl " 
```
Set the flags for Compiler-rt...
```
export  CONFIG_CRT="-DCOMPILER_RT_USE_BUILTINS_LIBRARY=ON "
export CONFIG_CRT+="-DCOMPILER_RT_BUILD_SANITIZERS=OFF"
export CONFIG_CRT+="-DCOMPILER_RT_BUILD_XRAY=OFF "
export CONFIG_CRT+="-DCOMPILER_RT_BUILD_PROFILE=OFF "
export CONFIG_CRT+="-DCOMPILER_RT_BUILD_LIBFUZZER=OFF " 
export CONFIG_CRT+="-DCOMPILER_RT_CAN_EXECUTE_TESTS=OFF "
export CONFIG_CRT+="-DCOMPILER_RT_HWASAN_WITH_INTERCEPTORS=OFF "
```
Set the flags for clang:
```
export  CONFIG_CLANG="-DCLANG_DEFAULT_CXX_STDLIB=libc++ "
export CONFIG_CLANG+="-DCLANG_DEFAULT_UNWINDLIB=libunwind "
export CONFIG_CLANG+="-DCLANG_DEFAULT_RTLIB=compiler-rt "
export CONFIG_CLANG+="-DCLANG_ENABLE_STATIC_ANALYZER=OFF "
export CONFIG_CLANG+="-DCLANG_ENABLE_ARCMT=OFF "
```
Set the flags to prevent build static libraries for libunwind, libcxxabi, and libcxx:
```
export CONFIG_LIBUNWIND="-DLIBUNWIND_ENABLE_STATIC=OFF "
export CONFIG_LIBCXXABI="-DLIBCXXABI_ENABLE_STATIC=OFF "
export CONFIG_LIBCXX="-DLIBCXX_ENABLE_STATIC=OFF "
```
Set paths...
```
export  CONFIG_PATHS="-DICONV_LIBRARY_PATH=/cgnutools/lib/libc.so "
export CONFIG_PATHS+="-DCMAKE_INSTALL_PREFIX=/cgnutools "
export CONFIG_PATHS+="-DDEFAULT_SYSROOT=/cgnutools " 
export CONFIG_PATHS+="-DBacktrace_LIBRARY=/cgnutools/lib/libexecinfo.so "
```
Turn off unwanted features, docs and tests
```
export  BUILD_OFF="-DLLVM_BUILD_TESTS=OFF "
export BUILD_OFF+="-DLLVM_INCLUDE_GO_TESTS=OFF "
export BUILD_OFF+="-DLLVM_INCLUDE_TESTS=OFF "
export BUILD_OFF+="-DLLVM_INCLUDE_DOCS=OFF "
export BUILD_OFF+="-DLLVM_INCLUDE_EXAMPLES=OFF "
export BUILD_OFF+="-DLLVM_INCLUDE_BENCHMARKS=OFF "
export BUILD_OFF+="-DLLVM_ENABLE_OCAMLDOC=OFF "
export BUILD_OFF+="-DLLVM_ENABLE_BACKTRACES=OFF "
export BUILD_OFF+="-DLLVM_ENABLE_LIBEDIT=OFF "
export BUILD_OFF+="-DLLVM_ENABLE_LIBXML2=OFF "
export BUILD_OFF+="-DLLVM_ENABLE_LIBPFM=OFF "
export BUILD_OFF+="-DLLVM_ENABLE_TERMINFO=OFF "
export BUILD_OFF+="-DLLVM_ENABLE_ZLIB=OFF "
export BUILD_OFF+="-DLLVM_ENABLE_Z3_SOLVER=OFF "
export BUILD_OFF+="-DLLVM_APPEND_VC_REV=OFF "
export BUILD_OFF+="-DLLVM_ENABLE_CRASH_OVERRIDES=OFF "
export BUILD_OFF+="-DLIBCXX_INCLUDE_BENCHMARKS=OFF "
export BUILD_OFF+="-DLIBCXX_ENABLE_DEBUG_MODE_SUPPORT=OFF "
export DUILD_OFF+="-DCOMPILER_RT_BUILD_ORC=false "
```
Configure source
```
cmake -B build  \
      -DCMAKE_BUILD_TYPE=Release \
      -DCMAKE_INSTALL_PREFIX="/cgnutools" \
      -DCMAKE_C_COMPILER="${ALMAZ_TARGET}-gcc" \
      -DCMAKE_CXX_COMPILER="${ALMAZ_TARGET}-g++" \
      -DLLVM_BUILD_TESTS=OFF \
      -DLLVM_ENABLE_LIBEDIT=OFF \
      -DLLVM_ENABLE_LIBXML2=OFF \
      -DLLVM_INCLUDE_GO_TESTS=OFF \
      -DLLVM_INCLUDE_TESTS=OFF \
      -DLLVM_INCLUDE_DOCS=OFF \
      -DLLVM_INCLUDE_EXAMPLES=OFF \
      -DLLVM_INCLUDE_BENCHMARKS=OFF \
      -DLLVM_DEFAULT_TARGET_TRIPLE="x86_64-pc-linux-musl" \
      -DLLVM_HOST_TRIPLE="x86_64-pc-linux-musl" \
      -DLLVM_TARGET_ARCH="X86" \
      -DLLVM_TARGETS_TO_BUILD="X86" \
      -DCOMPILER_RT_DEFAULT_TARGET_TRIPLE="x86_64-pc-linux-musl" \
      -DCOMPILER_RT_BUILD_SANITIZERS=OFF \
      -DCOMPILER_RT_BUILD_XRAY=OFF \
      -DCOMPILER_RT_BUILD_PROFILE=OFF \
      -DCOMPILER_RT_BUILD_LIBFUZZER=OFF \
      -DCOMPILER_RT_USE_BUILTINS_LIBRARY=ON \
      -DCLANG_DEFAULT_CXX_STDLIB=libc++ \
      -DCLANG_DEFAULT_UNWINDLIB=libunwind \
      -DCLANG_DEFAULT_RTLIB=compiler-rt \
      -DICONV_LIBRARY_PATH="/cgnutools/lib/libc.so" \
      -DDEFAULT_SYSROOT="/cgnutools" \
      -DLIBCXX_HAS_MUSL_LIBC=ON \
      -DLLVM_ENABLE_LIBCXX=ON \
      -DCLANG_DEFAULT_LINKER="/cgnutools/bin/ld.lld" \
      -DBacktrace_HEADER="/cgnutools/include/execinfo.h" \
      -DCMAKE_EXE_LINKER_FLAGS="-Wl,-dynamic-linker /cgnutools/lib/ld-musl-x86_64.so.1" \
      -DCMAKE_SHARED_LINKER_FLAGS="-Wl,-dynamic-linker /cgnutools/lib/ld-musl-x86_64.so.1" \
      -DBacktrace_LIBRARY="/cgnutools/lib/libexecinfo.so.1" \
      -DCOMPILER_RT_BUILD_ORC=false
```
Now ready to build
```
make -C build 
```
Install to /cgnutools
```
cd build && cmake -DCMAKE_INSTALL_PREFIX=/cgnutools -P cmake_install.cmake
```
Cleaning up
```
unset CFLAGS CXFLAGS LINKERFLAGS
cd .. && rm -rf build
```
Set LLD as default toolchain linker
```
ln -sv lld /cgnutools/bin/ld
```
Since LLVM+clang still have GCC dependancies, add the GCC libraries in the search paths of the toolchain's dynamic linker:
```
 echo "/cgnutools/${ALMAZ_TARGET}/lib" >> /cgnutools/etc/ld-musl-$(uname -m).path
```
Configure clang to build binaries with /llvmtools/lib/ld-musl-x86_64.so.1 instead of /lib/ld-musl-x86_64.so.1. This is similar in Musl-LFS/LFS when gcc specs file is modified to set the dynamic linker in /tools instead of host's /lib
```
ln -sv clang-14   /cgnutools/bin/${ALMAZ_TARGET}-clang
ln -sv clang-14   /cgnutools/bin/${ALMAZ_TARGET}-clang++
cat > /cgnutools/bin/${ALMAZ_TARGET}.cfg << "EOF"
-Wl,-dynamic-linker /llvmtools/lib/ld-musl-x86_64.so.1
EOF
```
Configure cross-GCC of cgnutools to match the same output as clang
Dump current specs 
```
export SPECFILE=`dirname $(${ALMAZ_TARGET}-gcc -print-libgcc-file-name)`/specs
${ALMAZ_TARGET}-gcc -dumpspecs > specs
```
Modify dumped specs file 
```
sed -i 's/\/lib\/ld-musl-x86_64.so.1/\/llvmtools\/lib\/ld-musl-x86_64.so.1/g' specs
```
Check:
```
grep "/llvmtools/lib/ld-musl-x86_64.so.1" specs  --color=auto
```
Install modified specs to the cgnutools toolchain
```
mv -v specs $SPECFILE
unset SPECFILE
```
Set the PATH
```
export PATH=/cgnutools/bin:/llvmtools/bin:/bin:/usr/bin
```
Test GCC of cgnutools toolchain:
```
echo "int main(){}" > dummy.c
${ALMAZ_TARGET}-gcc dummy.c -v -Wl,--verbose &> dummy.log
readelf -l a.out | grep ': /llvmtools'
```
Should output:
```
 [Requesting program interpreter: /llvmtools/lib/ld-musl-x86_64.so.1]
```
Check if the correct start files are used
```
grep  'lib.*/crt[1in].*succeeded' dummy.log | cut -d ' ' -f 4-5 | cut -b 5-
```
Should output:
```
 /almaz/cgnutools/bin/../../cgnutools/lib/../lib/crt1.o succeeded
 /almaz/cgnutools/bin/../../cgnutools/lib/../lib/crti.o succeeded
 /almaz/cgnutools/bin/../../cgnutools/lib/../lib/crtn.o succeeded
```
Delete files we made during our check:
```
rm -v dummy.log dummy.c a.out
```
