pkgname=('llvm33')
pkgver=3.3
pkgrel=2
arch=('i686' 'x86_64')
url="http://llvm.org/"
pkgdesc="LLVM 3.3 (installed in /opt/llvm33/)"
license=('custom:University of Illinois/NCSA Open Source License')
makedepends=('libffi' 'python2' 'ocaml' 'python-sphinx')
source=("llvm::http://llvm.org/svn/llvm-project/llvm/branches/release_33")
validpgpkeys=('C13549BB82A17681BF7143C2B4468DF4E95C63DC') # Bill Wendling
sha256sums=('SKIP')

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
