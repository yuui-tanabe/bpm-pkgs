# Circle CI Build Passed
pkgname=('llvm33')
pkgver=3.3
pkgrel=2
arch=('i686' 'x86_64')
url="http://llvm.org/"
pkgdesc="LLVM 3.3 (installed in /opt/llvm33/)"
license=('custom:University of Illinois/NCSA Open Source License')
makedepends=('libffi' 'python2' 'ocaml' 'python-sphinx' 'subversion')
source=('llvm::svn+https://llvm.org/svn/llvm-project/llvm/branches/release_33')
sha256sums=('SKIP')

pkgver() {
    cd "${srcdir}/${_pkgname}"

    # This will almost match the output of `llvm-config --version` when the
    # LLVM_APPEND_VC_REV cmake flag is turned on. The only difference is
    # dash being replaced with underscore because of Pacman requirements.
    echo $(awk -F 'MAJOR |MINOR |PATCH |SUFFIX |)' \
            'BEGIN { ORS="." ; i=0 } \
             /set\(LLVM_VERSION_/ { print $2 ; i++ ; if (i==2) ORS="" } \
             END { print "\n" }' \
        CMakeLists.txt)_r$(svnversion | tr -d [A-z])
}

prepare() {


  cd "$srcdir/llvm"
  ./configure \
  --with-python=/usr/bin/python2 \
  --enable-optimized \
  --disable-assertions \
  --enable-targets=x86 
}

build() {
  cd llvm
  REQUIRES_RTTI=1 make -j $(nproc)
}

package() {
  mkdir -p "${pkgdir}/opt"
  mv "${srcdir}/llvm/Release" "${pkgdir}/opt/llvm33"
  mv "${srcdir}/llvm/include" "${pkgdir}/opt/llvm33"
}


# vim:set ts=2 sw=2 et:
