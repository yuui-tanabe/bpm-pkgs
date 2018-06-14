pkgname=pacman
pkgver=5.1.0
pkgrel=2
pkgdesc="A library-based package manager with dependency support"
arch=('x86_64')
url="http://www.archlinux.org/pacman/"
license=('GPL')
groups=('base' 'base-devel')
depends=('bash' 'glibc' 'libarchive' 'curl'
         'gpgme' 'pacman-mirrorlist' 'archlinux-keyring')
makedepends=('asciidoc')
checkdepends=('python2' 'fakechroot')
backup=(etc/pacman.conf etc/makepkg.conf)
options=('strip' 'debug')
validpgpkeys=('6645B0A8C7005E78DB1D7864F99FFE0FEAE999BD'  # Allan McRae <allan@archlinux.org>
              'B8151B117037781095514CA7BBDFFC92306B1121') # Andrew Gregory (pacman) <andrew@archlinux.org>
source=(https://sources.archlinux.org/other/pacman/$pkgname-$pkgver.tar.gz{,.sig}
        0001-makepkg-Clear-ERR-trap-before-trying-to-restore-it.patch
        0002-makepkg-Don-t-use-parameterless-return.patch
        pacman.conf
        makepkg.conf)
sha256sums=('9f5993fc8923530713742f15df284677f297b3eca15ed7a24758c98ac7399bd3'
            'SKIP'
            '9b2304141582a421e812c76760a74f360a3cbd780472cbb60cf023a34d6fcb3d'
            '2a31d4db5f6e19e0148d4892de14317514f2b2dfb5369c7972a641ca8be89e5a'
            '95b3b2416402059cf6acf3e046082e7ce261e2b88629231dbf579a4200d8a63b'
            '650ddad24cad6afc4aecb4829cb8f06b9acb70c10a44f756fe8bd279949b518e')

prepare() {
  cd "$pkgname-$pkgver"
  # Fix install_packages failure exit code, required by makechrootpkg
  patch -Np1 -i ../0001-makepkg-Clear-ERR-trap-before-trying-to-restore-it.patch
  patch -Np1 -i ../0002-makepkg-Don-t-use-parameterless-return.patch
}

build() {
  cd "$pkgname-$pkgver"

  ./configure --prefix=/usr --sysconfdir=/etc \
    --localstatedir=/var --enable-doc \
    --with-scriptlet-shell=/usr/bin/bash \
    --with-ldconfig=/usr/bin/ldconfig
  make V=1
}

check() {
  make -C "$pkgname-$pkgver" check
}

package() {
  cd "$pkgname-$pkgver"

  make DESTDIR="$pkgdir" install

  # install Arch specific stuff
  install -dm755 "$pkgdir/etc"
  install -m644 "$srcdir/pacman.conf" "$pkgdir/etc"
  install -m644 "$srcdir/makepkg.conf" "$pkgdir/etc"

  # put bash_completion in the right location
  install -dm755 "$pkgdir/usr/share/bash-completion/completions"
  mv "$pkgdir/etc/bash_completion.d/pacman" "$pkgdir/usr/share/bash-completion/completions"
  rmdir "$pkgdir/etc/bash_completion.d"

  for f in makepkg pacman-key; do
    ln -s pacman "$pkgdir/usr/share/bash-completion/completions/$f"
  done
}
