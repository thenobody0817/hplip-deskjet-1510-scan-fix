# Maintainer: thenobody0817 <thenobody0817@users.noreply.github.com>
# Based on Arch Linux hplip 1:3.26.6-1, patched for HP Deskjet 1510 scan failure.

pkgname=hplip-deskjet-1510-scan-fix
_pkgbase=hplip
epoch=1
pkgver=3.26.6
pkgrel=1
pkgdesc="HPLIP patched to fix HP Deskjet 1510 (and related) scan buffer failure"
arch=(x86_64)
url="https://github.com/thenobody0817/hplip-deskjet-1510-scan-fix"
license=(
  GPL-2.0-or-later
  MIT
  BSD-3-Clause-HP
  IJG
  GPL-2.0-only
  LGPL-2.1-or-later
  BSD-2-Clause
  LicenseRef-hplip
  python-ldap
)
depends=(
  python-dbus python-distro ghostscript net-snmp foomatic-db-engine
  python-gobject libjpeg-turbo dbus libstdc++ libgcc avahi python
  xdg-utils zlib libcups glibc sh
)
optdepends=(
  'cups: for printing support'
  'sane: for scanner support'
  'python-pillow: for commandline scanning support'
  'python-reportlab: for pdf output in hp-scan'
  'rpcbind: for network support'
  'python-pyqt5: for running GUI and hp-toolbox'
  'libusb: for advanced usb support'
  'wget: for network support'
)
makedepends=(
  python-pyqt5 sane rpcbind cups libusb python-setuptools
)
backup=(
  'etc/hp/hplip.conf'
  'etc/sane.d/dll.d/hpaio'
)
provides=(hplip)
conflicts=(hplip)
options=(!libtool !staticlibs)
source=(
  "https://downloads.sourceforge.net/sourceforge/hplip/hplip-${pkgver}.tar.gz"
  "hplip-3.26.6-http-buffer-fix.patch"
)
sha256sums=(
  'SKIP'
  'SKIP'
)

prepare() {
  cd "hplip-${pkgver}"
  patch -p1 -i "${srcdir}/hplip-3.26.6-http-buffer-fix.patch"
}

build() {
  cd "hplip-${pkgver}"

  # HPLIP 3.26.6 needs older C standard and looser pointer checks on modern GCC
  export CFLAGS="${CFLAGS/-O2/-O2} -std=gnu17 -D_DEFAULT_SOURCE \
    -Wno-error=implicit-function-declaration \
    -Wno-error=incompatible-pointer-types"

  ./configure --prefix=/usr \
    --libdir=/usr/lib \
    --sbindir=/usr/bin \
    --disable-qt4 \
    --disable-qt5 \
    --enable-doc-build=no \
    --disable-cups-drv-install \
    --disable-hpcups-install \
    --disable-hpijs-install \
    --disable-fax-build \
    --enable-network-build \
    --enable-scan-build

  make
}

package() {
  cd "hplip-${pkgver}"
  make DESTDIR="${pkgdir}" install
}
