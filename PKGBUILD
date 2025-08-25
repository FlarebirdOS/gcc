pkgname=(
    gcc
    gcc-libs
    gcc-libs-32bit
    gcc-fortran
    gcc-fortran-32bit
    gcc-objc
    gcc-objc-32bit
    gcc-gnat
    gcc-gnat-32bit
    gcc-go
    gcc-go-32bit
    gcc-gdc
    gcc-gdc-32bit
    gcc-m2
    gcc-m2-32bit
    gcc-gcobol
    lto-dump
)
pkgbase=gcc
pkgver=15.2.0
pkgrel=2
pkgdesc="The GNU Compiler Collection"
arch=('x86_64')
url="https://gcc.gnu.org"
license=(
    'GPL-3.0-with-GCC-exception'
    'GFDL-1.3-or-later'
)
makedepends=(
    'binutils'
    'gcc-gdc'
    'gcc-gdc-32bit'
    'gcc-libs-32bit'
    'glibc-32bit'
    'isl'
    'mpc'
    'zstd'
)
options=('!emptydirs' '!lto')
source=(https://ftp.gnu.org/gnu/${pkgname}/${pkgbase}-${pkgver}/${pkgbase}-${pkgver}.tar.xz
    https://github.com/alire-project/GNAT-FSF-builds/releases/download/gnat-15.1.0-2/gnat-x86_64-linux-15.1.0-2.tar.gz
    gcc-ada-repro.patch
    c89
    c99)
sha256sums=(438fd996826b0c82485a29da03a72d71d6e3541a83ec702df4271f6fe025d24e
    f0e902c010158a79d04c06d3aee5b5999875d9b4c66c876009deffcaa3639df3
    1773f5137f08ac1f48f0f7297e324d5d868d55201c03068670ee4602babdef2f
    7b09ec947f90b98315397af675369a1e3dfc527fa70013062e6e85c4be0275ab
    44ea973558842f3f4bd666bdaf6e810fd7b7c7bd36b5cc4c69f93d2cd0124fc7)

prepare() {
    cd ${pkgbase}-${pkgver}

    sed -e '/m32=/s/m32=.*/m32=..\/lib32$(call if_multiarch,:i386-linux-gnu)/' \
        -i.orig gcc/config/i386/t-linux64

    sed '/STACK_REALIGN_DEFAULT/s/0/(!TARGET_64BIT \&\& TARGET_SSE)/' \
        -i gcc/config/i386/i386.h

    sed -i 's@\./fixinc\.sh@-c true@' gcc/Makefile.in

    sed -i 's|^libdir = $(exec_prefix)/lib|libdir = @libdir@|g' libobjc/Makefile.in

    patch -Np0 < ${srcdir}/gcc-ada-repro.patch

    mkdir flarebird-build
}

build() {
    cd ${pkgbase}-${pkgver}/flarebird-build

    local configure_args=(
        --enable-languages=ada,c,c++,d,fortran,go,lto,m2,objc,obj-c++,cobol
        --enable-default-pie
        --enable-default-ssp
        --enable-host-pie
        --enable-multilib
        --with-multilib-list=m64,m32
        --disable-libssp
        --disable-fixincludes
        --with-system-zlib
        --enable-lto
        --target=${CHOST}
        --with-pkgversion="Flarebird Gcc ${pkgver}"
        ${configure_options}
    )

    local GNAT_DIR=${srcdir}/gnat-x86_64-linux-15.1.0-2

    ln -sfv gnatmake ${GNAT_DIR}/bin/${CHOST}-gnatmake
    ln -sfv gnatbind ${GNAT_DIR}/bin/${CHOST}-gnatbind
    ln -sfv x86_64-pc-linux-gnu-gcc  ${GNAT_DIR}/bin/${CHOST}-gcc
    ln -sfv x86_64-pc-linux-gnu-gcc  ${GNAT_DIR}/bin/${CHOST}-gnat1

    export PATH_HOLD=${PATH}
    export PATH=${GNAT_DIR}/bin:${PATH}

    CFLAGS=${CFLAGS/-Werror=format-security/}
    CXXFLAGS=${CXXFLAGS/-Werror=format-security/}

    ../configure "${configure_args[@]}"

    make -O

    export PATH=${PATH_HOLD}
    unset PATH_HOLD
}

package_gcc() {
    pkgdesc="The GNU Compiler Collection - C and C++ frontends"
    groups=('base-devel')
    depends=("${pkgbase}-libs=${pkgver}-${pkgrel}" 'binutils' 'mpc' 'isl' 'zstd')
    options=('!emptydirs' 'staticlibs')

    cd ${pkgbase}-${pkgver}/flarebird-build

    make -C gcc DESTDIR=${pkgdir} install-driver install-cpp install-gcc-ar \
        c++.install-common install-headers install-plugin install-lto-wrapper

    install -m755 -t ${pkgdir}/usr/bin/ gcc/gcov{,-tool}
    install -m755 -t ${pkgdir}/usr/libexec/gcc/${CHOST}/${pkgver%%+*}/ gcc/{cc1,cc1plus,collect2,lto1}

    make -C ${CHOST}/libgcc DESTDIR=${pkgdir} install

    rm -f ${pkgdir}/usr/lib64/libgcc_s.so*

    make -C ${CHOST}/libstdc++-v3/src DESTDIR=${pkgdir} install
    make -C ${CHOST}/libstdc++-v3/include DESTDIR=${pkgdir} install
    make -C ${CHOST}/libstdc++-v3/libsupc++ DESTDIR=${pkgdir} install
    make -C ${CHOST}/libstdc++-v3/python DESTDIR=${pkgdir} install

    make -C ${CHOST}/32/libstdc++-v3/src DESTDIR=${pkgdir} install
    make -C ${CHOST}/32/libstdc++-v3/include DESTDIR=${pkgdir} install
    make -C ${CHOST}/32/libstdc++-v3/libsupc++ DESTDIR=${pkgdir} install

    make DESTDIR=${pkgdir} install-libcc1

    install -d ${pkgdir}/usr/share/gdb/auto-load/usr/lib64
    mv ${pkgdir}/usr/lib64/libstdc++.so.6.*-gdb.py ${pkgdir}/usr/share/gdb/auto-load/usr/lib64

    rm ${pkgdir}/usr/lib64/libstdc++.so*

    make DESTDIR=${pkgdir} install-fixincludes
    make -C gcc DESTDIR=${pkgdir} install-mkheaders

    make -C lto-plugin DESTDIR=${pkgdir} install

    install -dm755 ${pkgdir}/usr/lib64/bfd-plugins/
    ln -s /usr/libexec/gcc/${CHOST}/${pkgver%%+*} ${pkgdir}/usr/lib64/bfd-plugins/

    make -C ${CHOST}/libitm DESTDIR=${pkgdir} install-nodist_toolexeclibHEADERS
    make -C ${CHOST}/libquadmath DESTDIR=${pkgdir} install-nodist_libsubincludeHEADERS
    make -C ${CHOST}/libsanitizer DESTDIR=${pkgdir} install-nodist_{saninclude,toolexeclib}HEADERS
    make -C ${CHOST}/libsanitizer/asan DESTDIR=${pkgdir} install-nodist_toolexeclibHEADERS
    make -C ${CHOST}/libsanitizer/tsan DESTDIR=${pkgdir} install-nodist_toolexeclibHEADERS
    make -C ${CHOST}/libsanitizer/lsan DESTDIR=${pkgdir} install-nodist_toolexeclibHEADERS

    make -C gcc DESTDIR=${pkgdir} install-man install-info

    rm ${pkgdir}/usr/share/man/man1/{gccgo,gfortran,lto-dump,gdc,gm2,gcobol}.1
    rm ${pkgdir}/usr/share/man/man3/gcobol-io.3
    rm ${pkgdir}/usr/share/info/{gccgo,gfortran,gnat-style,gnat_rm,gnat_ugn,gdc,m2}.info

    make -C libcpp DESTDIR=${pkgdir} install
    make -C gcc DESTDIR=${pkgdir} install-po

    # many packages expect this symlink
    ln -s gcc ${pkgdir}/usr/bin/cc

    # Create a symlink required by the FHS for "historical" reasons.
    ln -svr /usr/bin/cpp ${pkgdir}/usr/lib64

    # POSIX conformance launcher scripts for c89 and c99
    install -Dm755 ${srcdir}/c89 ${pkgdir}/usr/bin/c89
    install -Dm755 ${srcdir}/c99 ${pkgdir}/usr/bin/c99

    # install the libstdc++ man pages
    # make -C ${CHOST}/libstdc++-v3/doc DESTDIR=${pkgdir} doc-install-man

    # byte-compile python libraries
    python3 -m compileall ${pkgdir}/usr/share/gcc-${pkgver%%+*}/
    python3 -O -m compileall ${pkgdir}/usr/share/gcc-${pkgver%%+*}/

    _pick gcclibs32bit ${pkgdir}/usr/lib32
    _pick gcclibs32bit ${pkgdir}/usr/include/c++/${pkgver%%+*}/${CHOST}/32
}

package_gcc-libs() {
    pkgdesc='Runtime libraries shipped by GCC'
    groups=('base')
    depends=('glibc>=2.27')
    options=('!emptydirs' '!strip')

    cd ${pkgbase}-${pkgver}/flarebird-build

    make -C ${CHOST}/libgcc DESTDIR=${pkgdir} install-shared
    rm -f ${pkgdir}/usr/lib64/gcc/${CHOST}/${pkgver%%+*}/libgcc_eh.a

    for lib in libatomic                  \
               libitm                     \
               libquadmath                \
               libsanitizer/{a,l,ub,t}san \
               libstdc++-v3/src           \
               libvtv; do
        make -C ${CHOST}/${lib} DESTDIR=${pkgdir} install-toolexeclibLTLIBRARIES
    done

    make -C ${CHOST}/libstdc++-v3/po DESTDIR=${pkgdir} install

    for lib in libitm libquadmath; do
        make -C ${CHOST}/${lib} DESTDIR=${pkgdir} install-info
    done

    # remove files provided by gcc-libs-32bit
    rm -rf ${pkgdir}/usr/lib32/

}

package_gcc-libs-32bit() {
    pkgdesc='32-bit runtime libraries shipped by GCC'
    depends=('glibc-32bit>=2.27')
    options=('!emptydirs' '!strip')

    mv gcclibs32bit/* ${pkgdir}

    cd ${pkgbase}-${pkgver}/flarebird-build

    make -C ${CHOST}/32/libgcc DESTDIR=${pkgdir} install

    make -C ${CHOST}/32/libgcc DESTDIR=${pkgdir} install-shared

    rm -f ${pkgdir}/usr/lib64/gcc/${CHOST}/${pkgver%%+*}/32/libgcc_eh.a

    make -C ${CHOST}/32/libitm DESTDIR=${pkgdir} install-nodist_toolexeclibHEADERS
    make -C ${CHOST}/32/libsanitizer DESTDIR=${pkgdir} install-nodist_{saninclude,toolexeclib}HEADERS
    make -C ${CHOST}/32/libsanitizer/asan DESTDIR=${pkgdir} install-nodist_toolexeclibHEADERS

    for lib in libatomic                \
               libitm                   \
               libquadmath              \
               libsanitizer/{a,l,ub}san \
               libstdc++-v3/src         \
               libvtv; do
        make -C ${CHOST}/32/${lib} DESTDIR=${pkgdir} install-toolexeclibLTLIBRARIES
    done

    # remove files provided by gcc-libs
    rm -rf ${pkgdir}/usr/lib64
}

package_gcc-fortran() {
    pkgdesc='Fortran front-end for GCC'
    depends=("${pkgbase}=${pkgver}-${pkgrel}" 'isl')

    cd ${pkgbase}-${pkgver}/flarebird-build

    make -C ${CHOST}/libgfortran DESTDIR=${pkgdir} install-toolexeclibLTLIBRARIES

    make -C ${CHOST}/libgfortran DESTDIR=${pkgdir} install-cafexeclibLTLIBRARIES \
        install-{toolexeclibDATA,nodist_fincludeHEADERS,gfor_cHEADERS}

    make -C ${CHOST}/libgomp DESTDIR=${pkgdir} install-nodist_{libsubinclude,toolexeclib}HEADERS

    make -C ${CHOST}/libgomp DESTDIR=${pkgdir} install-toolexeclibLTLIBRARIES
    make -C ${CHOST}/libgomp DESTDIR=${pkgdir} install-nodist_fincludeHEADERS

    make -C ${CHOST}/libgomp DESTDIR=${pkgdir} install-info

    make -C gcc DESTDIR=${pkgdir} fortran.install-{common,man,info}

    install -Dm755 gcc/f951 ${pkgdir}/usr/libexec/gcc/${CHOST}/${pkgver%%+*}/f951

    ln -s gfortran ${pkgdir}/usr/bin/f95
}

package_gcc-fortran-32bit() {
    pkgdesc='Fortran front-end for GCC (32-bit)'
    depends=(
        "${pkgbase}-libs-32bit=${pkgver}-${pkgrel}"
        "${pkgbase}-fortran=${pkgver}-${pkgrel}"
    )

    cd ${pkgbase}-${pkgver}/flarebird-build

    make -C ${CHOST}/32/libgfortran DESTDIR=${pkgdir} install-toolexeclibLTLIBRARIES

    make -C ${CHOST}/32/libgfortran DESTDIR=${pkgdir} install-cafexeclibLTLIBRARIES \
        install-{toolexeclibDATA,nodist_fincludeHEADERS,gfor_cHEADERS}

    make -C ${CHOST}/32/libgomp DESTDIR=${pkgdir} install-nodist_toolexeclibHEADERS
    make -C ${CHOST}/32/libgomp DESTDIR=${pkgdir} install-toolexeclibLTLIBRARIES

    rm -rf ${pkgdir}/usr/lib64/gcc/${CHOST}/${pkgver%%+*}/include
}

package_gcc-objc() {
    pkgdesc='Objective-C front-end for GCC'
    depends=("${pkgbase}=${pkgver}-${pkgrel}" 'isl')

    cd ${pkgbase}-${pkgver}/flarebird-build

    make -C ${CHOST}/libobjc DESTDIR=${pkgdir} install-libs

    make -C ${CHOST}/libobjc DESTDIR=${pkgdir} install-headers

    install -dm755 ${pkgdir}/usr/libexec/gcc/${CHOST}/${pkgver%%+*}
    install -m755 gcc/cc1obj{,plus} ${pkgdir}/usr/libexec/gcc/${CHOST}/${pkgver%%+*}/

    rm -rf ${pkgdir}/usr/lib32
}

package_gcc-objc-32bit() {
    pkgdesc='Objective-C front-end for GCC (32-bit)'
    depends=(
        "${pkgbase}-libs-32bit=${pkgver}-${pkgrel}"
        "${pkgbase}-objc=${pkgver}-${pkgrel}"
    )

    cd ${pkgbase}-${pkgver}/flarebird-build

    make -C ${CHOST}/32/libobjc DESTDIR=${pkgdir} install-libs
}

package_gcc-gnat() {
    pkgdesc='Ada front-end for GCC (GNAT)'
    depends=("${pkgbase}=${pkgver}-${pkgrel}" 'isl')
    options=('!emptydirs' 'staticlibs')

    cd ${pkgbase}-${pkgver}/flarebird-build

    make -C gcc DESTDIR=${pkgdir} ada.install-{common,info}

    install -m755 gcc/gnat1 ${pkgdir}/usr/libexec/gcc/${CHOST}/${pkgver%%+*}

    make -C ${CHOST}/libada DESTDIR=${pkgdir} INSTALL="install" INSTALL_DATA="install -m644" install-libada

    ln -s gcc ${pkgdir}/usr/bin/gnatgcc

    # insist on dynamic linking, but keep static libraries because gnatmake complains
    mv ${pkgdir}/usr/lib64/gcc/${CHOST}/${pkgver%%+*}/adalib/libgna{rl,t}-${pkgver%%.*}.so ${pkgdir}/usr/lib64

    ln -s libgnarl-${pkgver%%.*}.so ${pkgdir}/usr/lib64/libgnarl.so
    ln -s libgnat-${pkgver%%.*}.so ${pkgdir}/usr/lib64/libgnat.so
    rm -f ${pkgdir}/usr/lib64/gcc/${CHOST}/${pkgver%%+*}/adalib/libgna{rl,t}.so
}

package_gcc-gnat-32bit() {
    pkgdesc='Ada front-end for GCC (GNAT) (32-bit)'
    depends=(
        "${pkgbase}-libs-32bit=${pkgver}-${pkgrel}"
        "${pkgbase}-gnat=${pkgver}-${pkgrel}"
    )
    options=('!emptydirs' 'staticlibs')

    cd ${pkgbase}-${pkgver}/flarebird-build

    make -C ${CHOST}/32/libada DESTDIR=${pkgdir} INSTALL="install" INSTALL_DATA="install -m644" install-libada

    install -d ${pkgdir}/usr/lib32

    mv ${pkgdir}/usr/lib64/gcc/${CHOST}/${pkgver%%+*}/32/adalib/libgna{rl,t}-${pkgver%%.*}.so ${pkgdir}/usr/lib32

    ln -s libgnarl-${pkgver%%.*}.so ${pkgdir}/usr/lib32/libgnarl.so
    ln -s libgnat-${pkgver%%.*}.so ${pkgdir}/usr/lib32/libgnat.so
}

package_gcc-go() {
    pkgdesc='Go front-end for GCC'
    depends=("${pkgbase}=${pkgver}-${pkgrel}" 'isl')
    conflicts=('go')

    cd ${pkgbase}-${pkgver}/flarebird-build

    make -C ${CHOST}/libgo DESTDIR=${pkgdir} install-toolexeclibLTLIBRARIES

    make -C ${CHOST}/libgo DESTDIR=${pkgdir} install-exec-am

    make DESTDIR=${pkgdir} install-gotools

    make -C gcc DESTDIR=${pkgdir} go.install-{common,man,info}

    install -Dm755 gcc/go1 ${pkgdir}/usr/libexec/gcc/${CHOST}/${pkgver%%+*}/go1

    rm -rf ${pkgdir}/usr/lib32
}

package_gcc-go-32bit() {
    pkgdesc='Go front-end for GCC (32-bit)'
    depends=(
        "${pkgbase}-libs-32bit=${pkgver}-${pkgrel}"
        "${pkgbase}-go=${pkgver}-${pkgrel}"
    )

    cd ${pkgbase}-${pkgver}/flarebird-build

    make -C ${CHOST}/32/libgo DESTDIR=${pkgdir} install-toolexeclibLTLIBRARIES

    make -C ${CHOST}/32/libgo DESTDIR=${pkgdir} install-exec-am
}

package_gcc-gdc() {
    pkgdesc="D frontend for GCC"
    depends=("${pkgbase}=${pkgver}-${pkgrel}" 'isl')
    options=('staticlibs')

    cd ${pkgbase}-${pkgver}/flarebird-build

    make -C gcc DESTDIR=${pkgdir} d.install-{common,man,info}

    install -Dm755 gcc/gdc ${pkgdir}/usr/bin/gdc
    install -Dm755 gcc/d21 ${pkgdir}/usr/libexec/gcc/${CHOST}/${pkgver%%+*}/d21

    make -C ${CHOST}/libphobos DESTDIR=${pkgdir} install

    _pick gdc32bit ${pkgdir}/usr/lib32
}

package_gcc-gdc-32bit() {
    pkgdesc="D frontend for GCC (32-bit)"
    depends=(
        "${pkgbase}-libs-32bit=${pkgver}-${pkgrel}"
        "gcc-gdc=${pkgver}-${pkgrel}"
    )
    options=('staticlibs')

    mv gdc32bit/* ${pkgdir}
}

package_gcc-m2() {
    pkgdesc='Modula-2 frontend for GCC'
    depends=("${pkgbase}=${pkgver}-${pkgrel}" 'isl')

    cd ${pkgbase}-${pkgver}/flarebird-build

    make -C gcc DESTDIR=${pkgdir} m2.install-{common,man,info}

    install -Dm755 gcc/cc1gm2 ${pkgdir}/usr/libexec/gcc/${CHOST}/${pkgver%%+*}/cc1gm2

    install -Dm755 gcc/gm2 ${pkgdir}/usr/bin/gm2

    make -C ${CHOST}/libgm2 DESTDIR=${pkgdir} install

    _pick m232bit ${pkgdir}/usr/lib32
}

package_gcc-m2-32bit() {
    pkgdesc='Modula-2 frontend for GCC (32-bit)'
    depends=(
        "${pkgbase}-libs-32bit=${pkgver}-${pkgrel}"
        "${pkgbase}-m2=${pkgver}-${pkgrel}"
    )

    mv m232bit/* ${pkgdir}
}

package_gcc-rust() {
    pkgdesc="Rust frontend for GCC"
    depends=("${pkgbase}=${pkgver}-${pkgrel}" 'isl')

    cd ${pkgbase}-${pkgver}/flarebird-build

    make -C gcc DESTDIR=${pkgdir} rust.install-{common,man,info}

    install -Dm755 gcc/gccrs ${pkgdir}/usr/bin/gccrs
    install -Dm755 gcc/crab1 ${pkgdir}/usr/bin/crab1
}

package_gcc-gcobol() {
    pkgdesc="Cobol frontend for GCC"
    depends=("${pkgbase}=${pkgver}-${pkgrel}" 'isl')

    cd ${pkgbase}-${pkgver}/flarebird-build

    make -C gcc DESTDIR=${pkgdir} cobol.install-{common,man,info}

    install -Dm755 gcc/gcobol ${pkgdir}/usr/bin/gcobol
}

package_lto-dump() {
    pkgdesc="Dump link time optimization object files"
    depends=("${pkgbase}=${pkgver}-${pkgrel}" 'isl')

    cd ${pkgbase}-${pkgver}/flarebird-build

    make -C gcc DESTDIR=${pkgdir} lto.install-{common,man,info}
}
