# Maintainer: Benjamin Schneider <ben@bens.haus>

buildarch=8

pkgbase=linux-ebu
pkgver=6.7.2
pkgrel=1
pkgdesc='Linux for Globalscale ESPRESSObin Ultra (Marvell Armada A3720 SoC)'
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
  install.hook
  install.script
  remove.hook
  cpufreq.patch
)
validpgpkeys=(
  'ABAF11C65A2970B130ABE3C479BE3E4300411886'  # Linus Torvalds
  '647F28654894E3BD457199BE38DBBDC86092693E'  # Greg Kroah-Hartman
)
# https://www.kernel.org/pub/linux/kernel/v6.x/sha256sums.asc
sha256sums=('c34de41baa29c475c0834e88a3171e255ff86cd32d83c6bffc2b797e60bfa671'
         'SKIP'
         '816a9662954c6710cc64431566c0715b5a35b92d7e2afbc9b9b1b6ac6ac11aae'
         'ce1f3e11c73246eea9cd7652d4ba1713ca3670a981c0917879ecfe7314a8fd68'
         'e9e6f62cdb77c8857341458884bfe45d5d37923540e7995ef903884d3793d8e9'
         '4a0e2369200298f62296eeee026cc46743998877443c642c2823990bf0662552'
         'f058667bd7bf1253cdac131138c9005dae0bded66ffb3922bb6802d08c18f46f'
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
    kmod
    mkinitcpio
    uboot-tools
  )
  optdepends=(
    'linux-firmware-marvell: wifi and bluetooth driver (mwifiex)'
    'wireless-regdb: to set the correct wireless channels of your country'
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

  echo "Installing pacman hooks for initramfs generation..."
  sed "${_subst}" ../remove.hook |
    install -Dm644 /dev/stdin "$pkgdir/usr/share/libalpm/hooks/60-$pkgbase-remove.hook"

  sed "${_subst}" ../install.hook |
    install -Dm644 /dev/stdin "$pkgdir/usr/share/libalpm/hooks/90-$pkgbase-install.hook"

  sed "${_subst}" ../install.script |
    install -Dm755 /dev/stdin "$pkgdir/usr/share/libalpm/scripts/$pkgbase"

  echo "Installing boot image and device tree..."
  install -Dm644 arch/arm64/boot/Image -t "$pkgdir/boot/$pkgbase"
  install -Dm644 arch/arm64/boot/dts/marvell/armada-3720-espressobin-ultra.dtb "$pkgdir/boot/$pkgbase/fdt.dtb"

  echo "Installing modules..."
  make INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 modules_install

  # remove build link
  rm "$modulesdir"/build
}

pkgname=("$pkgbase")
