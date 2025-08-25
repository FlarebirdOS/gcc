pkgname=(gcc gcc-libs gcc-libs-32bit)
pkgbase=gcc
pkgver=15.2.0
pkgrel=1
pkgdesc="The GNU Compiler Collection"
arch=('x86_64')
url="https://gcc.gnu.org"
license=(
    'GPL-3.0-with-GCC-exception'
    'GFDL-1.3-or-later'
)
makedepends=(
    'binutils'
    'glibc-32bit'
    'isl'
    'mpc'
    'zstd'
)
options=('!emptydirs' '!lto')
source=(https://ftp.gnu.org/gnu/${pkgname}/${pkgbase}-${pkgver}/${pkgbase}-${pkgver}.tar.xz
    c89
    c99)
sha256sums=(438fd996826b0c82485a29da03a72d71d6e3541a83ec702df4271f6fe025d24e
    7b09ec947f90b98315397af675369a1e3dfc527fa70013062e6e85c4be0275ab
    44ea973558842f3f4bd666bdaf6e810fd7b7c7bd36b5cc4c69f93d2cd0124fc7)

prepare() {
    cd ${pkgbase}-${pkgver}

    sed -e '/m32=/s/m32=.*/m32=..\/lib32$(call if_multiarch,:i386-linux-gnu)/' \
        -i.orig gcc/config/i386/t-linux64

    sed '/STACK_REALIGN_DEFAULT/s/0/(!TARGET_64BIT \&\& TARGET_SSE)/' \
        -i gcc/config/i386/i386.h

    mkdir flarebird-build
}

build() {
    cd ${pkgbase}-${pkgver}/flarebird-build

    local configure_args=(
        LD=ld
        --enable-languages=c,c++
        --enable-default-pie
        --enable-default-ssp
        --enable-host-pie
        --enable-multilib
        --with-multilib-list=m64,m32
        --disable-libssp
        --disable-bootstrap
        --disable-fixincludes
        --with-system-zlib
        --target=${CHOST}
	--with-pkgversion="Flarebird Gcc ${pkgver}"
        ${configure_options}
    )

    CFLAGS=${CFLAGS/-Werror=format-security/}
    CXXFLAGS=${CXXFLAGS/-Werror=format-security/}

    ../configure "${configure_args[@]}"

    make -O
}

package_gcc() {
    pkgdesc="The GNU Compiler Collection - C and C++ frontends"
    groups=('base-devel')
    depends=(
        'binutils'
        "gcc-libs=${pkgver}-${pkgrel}"
        'isl'
        'mpc'
        'zstd'
    )
    options=('!emptydirs')

    cd ${pkgbase}-${pkgver}/flarebird-build

    make DESTDIR=${pkgdir} install

    chown -v -R root:root ${pkgdir}/usr/lib64/gcc/$(gcc -dumpmachine)/${pkgver%%+*}/include{,-fixed}

    ln -s gcc ${pkgdir}/usr/bin/cc
    ln -svr /usr/bin/cpp ${pkgdir}/usr/lib64

    ln -sv gcc.1 ${pkgdir}/usr/share/man/man1/cc.1

    install -dm755 ${pkgdir}/usr/lib64/bfd-plugins/
    ln -sv /usr/libexec/gcc/$(gcc -dumpmachine)/${pkgver%%+*} ${pkgdir}/usr/lib64/bfd-plugins/

    install -d ${pkgdir}/usr/share/gdb/auto-load/usr/lib{32,64}
    mv ${pkgdir}/usr/lib64/libstdc++.so.6.*-gdb.py ${pkgdir}/usr/share/gdb/auto-load/usr/lib64/
    mv ${pkgdir}/usr/lib32/libstdc++.so.6.*-gdb.py ${pkgdir}/usr/share/gdb/auto-load/usr/lib32/

    install -vm755 ${srcdir}/c89 ${pkgdir}/usr/bin/c89
    install -vm755 ${srcdir}/c99 ${pkgdir}/usr/bin/c99

    _pick gcc-libs ${pkgdir}/usr/lib64/libasan.so*
    _pick gcc-libs ${pkgdir}/usr/lib64/libatomic.so*
    _pick gcc-libs ${pkgdir}/usr/lib64/libgcc_s.so
    _pick gcc-libs ${pkgdir}/usr/lib64/libgcc_s.so.1
    _pick gcc-libs ${pkgdir}/usr/lib64/libgomp.so*
    _pick gcc-libs ${pkgdir}/usr/lib64/libhwasan.so*
    _pick gcc-libs ${pkgdir}/usr/lib64/libitm.so*
    _pick gcc-libs ${pkgdir}/usr/lib64/liblsan.so*
    _pick gcc-libs ${pkgdir}/usr/lib64/libquadmath.so*
    _pick gcc-libs ${pkgdir}/usr/lib64/libstdc++.so*
    _pick gcc-libs ${pkgdir}/usr/lib64/libtsan.so*
    _pick gcc-libs ${pkgdir}/usr/lib64/libubsan.so*

    _pick gcc-libs-32bit ${pkgdir}/usr/lib32
    _pick gcc-libs-32bit ${pkgdir}/usr/share/gdb/auto-load/usr/lib32/
    _pick gcc-libs-32bit ${pkgdir}/usr/include/c++/${pkgver%%+*}/${CHOST}/32
}

package_gcc-libs() {
    pkgdesc="Runtime libraries shipped by GCC"
    groups=('base')
    depends=('glibc')
    options=('!emptydirs' '!strip')

    mv ${pkgname}/* ${pkgdir}

    cd ${pkgbase}-${pkgver}/flarebird-build

    make -C $CHOST/libstdc++-v3/po DESTDIR=${pkgdir} install

    for lib in libgomp libitm libquadmath
    do
        make -C ${CHOST}/${lib} DESTDIR=${pkgdir} install-info
    done
}

package_gcc-libs-32bit() {
    pkgdesc="32-bit runtime libraries shipped by GCC"
    depends=('glibc-32bit')
    options=('!emptydirs' '!strip')

    mv ${pkgname}/* ${pkgdir}
}
