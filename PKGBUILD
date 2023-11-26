# Maintainer: Benjamin Schneider <ben@bens.haus>

buildarch=8

pkgbase=linux-ebu
pkgver=6.6.2
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
  install.script
  remove.hook
)
validpgpkeys=(
  'ABAF11C65A2970B130ABE3C479BE3E4300411886'  # Linus Torvalds
  '647F28654894E3BD457199BE38DBBDC86092693E'  # Greg Kroah-Hartman
)
# https://www.kernel.org/pub/linux/kernel/v6.x/sha256sums.asc
sha256sums=('73d4f6ad8dd6ac2a41ed52c2928898b7c3f2519ed5dbdb11920209a36999b77e'
         'SKIP'
         '2732b49ee66e8827cbf566408c4007055bf1bcd5151c55f4232b29ed5eda4288'
         '378e9652f075ac7c6a9eb6cfc2cd7986475a516878b045258cc235790c32d537'
         'e26dcc0c8bc2341c9624dcd0368629fce641cb0665f6782a3765465bc893b75d'
         '028db5aa0264df7f785a1f7dc42c55958999d4d433e8677a767c51f91426afba'
         '4a0e2369200298f62296eeee026cc46743998877443c642c2823990bf0662552'
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

  sed "${_subst}" ../install.script |
    install -Dm755 /dev/stdin "$pkgdir/usr/share/libalpm/scripts/$pkgbase"

  echo "Installing kernel image and device tree binary..."
  install -Dm644 arch/arm64/boot/Image -t "$pkgdir/boot/$pkgbase"
  install -Dm644 arch/arm64/boot/dts/marvell/armada-3720-espressobin-ultra.dtb "$pkgdir/boot/$pkgbase/fdt.dtb"

  # mkinitcpio looks for the kernel here
  ln -s "$pkgbase/Image" "$pkgdir/boot/Image-$pkgbase"

  echo "Installing modules..."
  make INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 modules_install

  # remove build link
  rm "$modulesdir"/build
}

pkgname=("$pkgbase")
