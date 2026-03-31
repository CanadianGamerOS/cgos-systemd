# Maintainer: canadiangamerOS <https://gitlab.com/canadiangamerOS/cgos-systemd>
# Based on the Arch Linux systemd PKGBUILD

pkgbase=cgos-systemd
pkgname=(
  cgos-systemd
  cgos-systemd-libs
  cgos-systemd-resolvconf
  cgos-systemd-sysvcompat
)
pkgver=261
pkgrel=1
pkgdesc="System and Service Manager (CGOS fork)"
arch=(x86_64)
url="https://gitlab.com/canadiangamerOS/cgos-systemd"
license=(
  GPL-2.0-or-later
  LGPL-2.1-or-later
)
makedepends=(
  acl
  audit
  cryptsetup
  curl
  dbus
  elfutils
  gnutls
  git
  gperf
  iproute2
  kbd
  kmod
  libcap
  libgcrypt
  libidn2
  libp11-kit
  libseccomp
  libxkbcommon
  lz4
  meson
  openssl
  pam
  libarchive
  python-jinja
  tpm2-tss
  util-linux-libs
  xz
  zlib
  zstd
)
source=("cgos-systemd::git+https://gitlab.com/canadiangamerOS/cgos-systemd.git#branch=main")
sha256sums=('SKIP')

build() {
  local meson_options=(
    -Dbootloader=disabled
    -Dbpf-framework=disabled
    -Ddefault-dnssec=no
    -Dfirstboot=false
    -Dinstall-tests=false
    -Dldconfig=false
    -Dman=disabled
    -Drpmmacrosdir=no
    -Dsysusers=false
    -Dtests=false
    # disable optional features not needed for a live ISO
    -Dapparmor=disabled
    -Dhomed=disabled
    -Dimportd=disabled
    -Dlibfido2=disabled
    -Dmicrohttpd=disabled
    -Dpasswdqc=disabled
    -Dpcre2=disabled
    -Dpolkit=disabled
    -Dpwquality=disabled
    -Dqrencode=disabled
    -Dremote=disabled
    -Dselinux=disabled
    -Dsysupdate=disabled
    -Dvmspawn=disabled
    -Dxenctrl=disabled
    -Db_lto=true
    -Db_pie=true
  )

  if command -v artix-meson &>/dev/null; then
    artix-meson "$pkgbase" build "${meson_options[@]}"
  elif command -v arch-meson &>/dev/null; then
    arch-meson "$pkgbase" build "${meson_options[@]}"
  else
    meson setup "$pkgbase" build --prefix=/usr "${meson_options[@]}"
  fi

  meson compile -C build
}

package_cgos-systemd-libs() {
  pkgdesc="cgos-systemd client libraries"
  depends=(
    gcc-libs
    libcap
    libgcrypt
    lz4
    xz
    zstd
  )
  provides=(
    libsystemd
    libsystemd.so
    libudev.so
    systemd-libs="${pkgver}"
  )
  conflicts=(systemd-libs libsystemd)

  meson install -C build --no-rebuild --destdir "$pkgdir" \
    --tags libudev,libsystemd,devel
}

package_cgos-systemd() {
  pkgdesc="$pkgdesc"
  depends=(
    "cgos-systemd-libs=${pkgver}"
    acl audit bash cryptsetup curl dbus elfutils glib2 gnutls
    iproute2 kbd kmod libcap libgcrypt libidn2 libp11-kit libseccomp
    lz4 openssl pam libarchive tpm2-tss util-linux-libs
    xz zlib zstd
  )
  provides=(
    nss-myhostname
    systemd="${pkgver}"
    udev
  )
  conflicts=(systemd udev)
  replaces=(udev)
  backup=(
    etc/pam.d/systemd-user
    etc/systemd/coredump.conf
    etc/systemd/homed.conf
    etc/systemd/journald.conf
    etc/systemd/logind.conf
    etc/systemd/networkd.conf
    etc/systemd/resolved.conf
    etc/systemd/sleep.conf
    etc/systemd/system.conf
    etc/systemd/timesyncd.conf
    etc/systemd/user.conf
    etc/udev/rules.d/99-default.rules
    etc/udev/udev.conf
  )

  meson install -C build --no-rebuild --destdir "$pkgdir"

  # remove libs/devel handled by cgos-systemd-libs
  rm -rf \
    "$pkgdir"/usr/lib/libsystemd* \
    "$pkgdir"/usr/lib/libudev* \
    "$pkgdir"/usr/include \
    "$pkgdir"/usr/lib/pkgconfig/libsystemd.pc \
    "$pkgdir"/usr/lib/pkgconfig/libudev.pc \
    "$pkgdir"/usr/share/pkgconfig/systemd.pc \
    "$pkgdir"/usr/share/pkgconfig/udev.pc

  # remove sysvcompat binaries handled by cgos-systemd-sysvcompat
  rm -f "$pkgdir"/usr/bin/{halt,init,poweroff,reboot,shutdown}

  # remove resolvconf handled by cgos-systemd-resolvconf
  rm -f "$pkgdir"/usr/bin/resolvconf

  # remove files that conflict with filesystem package
  rm -f \
    "$pkgdir"/usr/share/factory/etc/issue \
    "$pkgdir"/usr/share/factory/etc/nsswitch.conf
  sed -i \
    -e '/^C \/etc\/nsswitch\.conf/d' \
    -e '/^C \/etc\/issue/d' \
    "$pkgdir"/usr/lib/tmpfiles.d/etc.conf

  install -Dm644 "$pkgbase/LICENSE.GPL2" -t "$pkgdir/usr/share/licenses/$pkgname/"
  install -Dm644 "$pkgbase/LICENSE.LGPL2.1" -t "$pkgdir/usr/share/licenses/$pkgname/"
}

package_cgos-systemd-resolvconf() {
  pkgdesc="cgos-systemd resolvconf replacement (passthrough to resolved)"
  depends=("cgos-systemd=${pkgver}")
  provides=(resolvconf openresolv)
  conflicts=(resolvconf)

  install -d "$pkgdir/usr/bin"
  ln -s /usr/bin/resolvectl "$pkgdir/usr/bin/resolvconf"

  install -d "$pkgdir/usr/share/licenses/$pkgname"
  ln -s /usr/share/licenses/cgos-systemd "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_cgos-systemd-sysvcompat() {
  pkgdesc="cgos-systemd SysV compatibility scripts"
  depends=("cgos-systemd=${pkgver}")
  provides=(systemd-sysvcompat)
  conflicts=(sysvinit systemd-sysvcompat)

  install -d "$pkgdir/usr/bin"
  ln -s ../lib/systemd/systemd "$pkgdir/usr/bin/init"
  for tool in halt poweroff reboot shutdown; do
    ln -s systemctl "$pkgdir/usr/bin/$tool"
  done

  install -d "$pkgdir/usr/share/licenses/$pkgname"
  ln -s /usr/share/licenses/cgos-systemd "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
