pkgname=wget
pkgver=1.19.5
pkgrel=1
pkgdesc='Network utility to retrieve files from the Web'
url='https://www.gnu.org/software/wget/wget.html'
arch=('i686' 'x86_64')
license=('GPL3')
depends=('glibc' 'gnutls' 'libidn' 'libutil-linux' 'libpsl' 'pcre')
checkdepends=('perl-http-daemon' 'perl-io-socket-ssl' 'python')
optdepends=('ca-certificates: HTTPS downloads')
backup=('etc/wgetrc')
source=(https://ftp.gnu.org/gnu/${pkgname}/${pkgname}-${pkgver}.tar.lz{,.sig})
sha256sums=('29fbe6f3d5408430c572a63fe32bd43d5860f32691173dfd84edc06869edca75'
            'SKIP')
validpgpkeys=('AC404C1C0BF735C63FF4D562263D6DF2E163E1EA'
              '7845120B07CBD8D6ECE5FF2B2A1743EDA91A35B6'
              '1CB27DBC98614B2D5841646D08302DB6A2670428') # Tim Rühsen <tim.ruehsen@gmx.de>

prepare() {
  cd ${pkgname}-${pkgver}
  cat >> doc/sample.wgetrc <<EOF

# default root certs location
ca_certificate=/etc/ssl/certs/ca-certificates.crt
EOF
}

build() {
  cd ${pkgname}-${pkgver}
  ./configure \
    --prefix=/usr \
    --sysconfdir=/etc \
    --disable-rpath \
    --enable-nls \
    --with-ssl=gnutls
  make
}

check() {
  cd ${pkgname}-${pkgver}
  make check < /dev/null
}

package() {
  cd ${pkgname}-${pkgver}
  make DESTDIR="${pkgdir}" install
}

# vim: ts=2 sw=2 et:
