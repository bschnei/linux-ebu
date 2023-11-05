# Maintainer: Benjamin Schneider <ben@bens.haus>

buildarch=8

pkgbase=linux-ebu
pkgver=6.5.9
pkgrel=1
pkgdesc='Linux'
url="https://www.kernel.org/"
arch=(aarch64)
license=(GPL2)
makedepends=(
  bc
  tar
  xz
)
options=('!strip')
_srcname=linux-$pkgver
source=(
  https://cdn.kernel.org/pub/linux/kernel/v${pkgver%%.*}.x/${_srcname}.tar.{xz,sign}
  config
  u-boot.env
  install.hook
  remove.hook
  linux-ebu.script
)
validpgpkeys=(
  'ABAF11C65A2970B130ABE3C479BE3E4300411886'  # Linus Torvalds
  '647F28654894E3BD457199BE38DBBDC86092693E'  # Greg Kroah-Hartman
)
# https://www.kernel.org/pub/linux/kernel/v6.x/sha256sums.asc
sha256sums=('c6662f64713f56bf30e009c32eac15536fad5fd1c02e8a3daf62a0dc2f058fd5'
         'SKIP'
         '4731bb6afc4310bec451d2f265b4f39be454453a35866a206d8d75e3c73d7f2b'
         '378e9652f075ac7c6a9eb6cfc2cd7986475a516878b045258cc235790c32d537'
         '6817ba045ca75e00a169c10c8a22d712a365c0ecded7403f3fc86db52459654d'
         '4a0e2369200298f62296eeee026cc46743998877443c642c2823990bf0662552'
         '028db5aa0264df7f785a1f7dc42c55958999d4d433e8677a767c51f91426afba'
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
    initramfs
    kmod
    uboot-tools
  )
  optdepends=(
    'wireless-regdb: to set the correct wireless channels of your country'
    'linux-firmware: firmware images needed for some devices'
  )
  provides=(WIREGUARD-MODULE)

  cd $_srcname
  local kernver="$(<version)"
  local modulesdir="$pkgdir/usr/lib/modules/$kernver"

  # sed expression for following substitutions
  local _subst="
    s|%PKGBASE%|${pkgbase}|g
    s|%KERNVER%|${kernver}|g
  "

  echo "Installing u-boot environment..."
  sed "${_subst}" ../u-boot.env |
    install -Dm644 /dev/stdin "$pkgdir/boot/$pkgbase/u-boot.env"

  echo "Installing pacman hooks for initramfs generation..."
  sed "${_subst}" ../remove.hook |
    install -Dm644 /dev/stdin "$pkgdir/usr/share/libalpm/hooks/60-$pkgbase-remove.hook"

  sed "${_subst}" ../install.hook |
    install -Dm644 /dev/stdin "$pkgdir/usr/share/libalpm/hooks/90-$pkgbase-install.hook"

  sed "${_subst}" ../linux-ebu.script |
    install -Dm755 /dev/stdin "$pkgdir/usr/share/libalpm/scripts/$pkgbase"

  echo "Installing kernel image and device tree binary..."
  install -Dm644 arch/arm64/boot/Image -t "$pkgdir/boot/$pkgbase"
  install -Dm644 arch/arm64/boot/dts/marvell/armada-3720-espressobin-ultra.dtb "$pkgdir/boot/$pkgbase/fdt.dtb"

  # mkinitcpio looks for the kernel here
  ln -s "$pkgdir/boot/$pkgbase/Image" "$pkgdir/boot/Image-$pkgbase" 

  echo "Installing modules..."
  make INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 modules_install

  # remove build and source links
  rm "$modulesdir"/{source,build}
}

pkgname=("$pkgbase")
