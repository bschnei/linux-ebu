# Maintainer: Benjamin Schneider <ben@bens.haus>

buildarch=8

pkgbase=linux-ebu
pkgver=6.0.19
pkgrel=2
pkgdesc='Linux'
url="https://www.kernel.org/"
arch=(aarch64)
license=(GPL2)
makedepends=(
  bc
  inetutils
  kmod
)
options=('!strip')
_srcname=linux-$pkgver
source=(
  https://cdn.kernel.org/pub/linux/kernel/v${pkgver%%.*}.x/${_srcname}.tar.{xz,sign}
  config
  0001-arm64-dts-marvell-espressobin-ultra-boot.patch
  uboot.env
)
validpgpkeys=(
  'ABAF11C65A2970B130ABE3C479BE3E4300411886'  # Linus Torvalds
  '647F28654894E3BD457199BE38DBBDC86092693E'  # Greg Kroah-Hartman
)
# https://www.kernel.org/pub/linux/kernel/v6.x/sha256sums.asc
sha256sums=('abe37eb0e2e331bdc7c4110115664e180e7d43b7336de6b4cd2bd1b123d30207'
         'SKIP'
         '4731bb6afc4310bec451d2f265b4f39be454453a35866a206d8d75e3c73d7f2b'
         'd0f0e9443b66ed166a1d7440c185d9777ae021c57546f211b1cfb239cf9c25d8'
         'e44de9a49bf1bdba691b0ac92b825972f385df20560975abddd2bf12f0d7c6e8'
)
prepare() {
  cd $_srcname

  echo "Setting version..."
  echo "-$pkgrel" > localversion.10-pkgrel
  echo "${pkgbase#linux}" > localversion.20-pkgname

  local src
  for src in "${source[@]}"; do
    src="${src%%::*}"
    src="${src##*/}"
    src="${src%.zst}"
    [[ $src = *.patch ]] || continue
    echo "Applying patch $src..."
    patch -Np1 < "../$src"
  done

  echo "Setting config..."
  cp ../config .config
  make olddefconfig
  diff -u ../config .config || :

  make -s kernelrelease > version
  echo "Prepared $pkgbase version $(<version)"
}

build() {
  cd ${_srcname}

  unset LDFLAGS
  make ${MAKEFLAGS} Image modules dtbs
}

package() {
  pkgdesc="The $pkgdesc kernel and modules"
  depends=(
    coreutils
    kmod
  )
  optdepends=(
    'wireless-regdb: to set the correct wireless channels of your country'
    'linux-firmware: firmware images needed for some devices'
  )
  provides=(WIREGUARD-MODULE)

  echo "Installing u-boot environment..."
  sed "s|%PKGVER%|${pkgver}|g" uboot.env |
    install -Dm644 /dev/stdin "$pkgdir/boot/linux/$pkgver/uboot.env"

  cd $_srcname
  local modulesdir="$pkgdir/usr/lib/modules/$(<version)"

  echo "Installing boot image and device tree binaries..."
  install -Dm644 arch/arm64/boot/Image -t "$pkgdir/boot/linux/$pkgver"
  make INSTALL_DTBS_PATH="$pkgdir/boot/linux/$pkgver/dtbs" dtbs_install

  echo "Installing modules..."
  make INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 modules_install

  # remove build and source links
  rm "$modulesdir"/{source,build}
}

pkgname=("$pkgbase")
