# Maintainer: Benjamin Schneider <ben@bens.haus>

buildarch=8

pkgbase=linux-ebu
pkgver=6.6.7
pkgrel=1
pkgdesc='Linux for Globalscale ESPRESSObin Ultra'
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
)
validpgpkeys=(
  'ABAF11C65A2970B130ABE3C479BE3E4300411886'  # Linus Torvalds
  '647F28654894E3BD457199BE38DBBDC86092693E'  # Greg Kroah-Hartman
)
# https://www.kernel.org/pub/linux/kernel/v6.x/sha256sums.asc
sha256sums=('0ce68ec6019019140043263520955ecd04839e55a1baab2fa9155b42bb6fd841'
         'SKIP'
         'e364434063b0bb60e670e453f02d1bb92b67cd70b235a53b9b59f92213ad21f4'
         '0db0ba677d1acabae3ae444dcbd8b88f88add0e5d2a77a9d53fe7be0f10d8425'
         '9c87dbf165d13879b8c9f4875d87b4122ea765d1833c3511a44fec28a73cddfc'
         'f83e9851133d98814b7ca5fa1e3f66ec191c57bebfd73f9db60da7530da0f992'
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
    'linux-firmware-marvell: wifi driver'
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

  echo "Installing boot image and device tree blobs..."
  install -Dm644 arch/arm64/boot/Image -t "$pkgdir/boot"
  make INSTALL_PATH="$pkgdir/boot" dtbs_install

  echo "Installing modules..."
  make INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 modules_install

  # remove build link
  rm "$modulesdir"/build
}

pkgname=("$pkgbase")
