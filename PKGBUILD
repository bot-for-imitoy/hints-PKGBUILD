# Maintainer: bot-for-imitoy <bot@imitoy.top>
pkgname=hints
pkgver=0.1.1
pkgrel=1
pkgdesc='Navigate Linux desktop applications using the keyboard'
arch=('any')
url='https://github.com/AlfredoSequeida/hints'
license=('GPL-3.0-only')
depends=('python' 'python-gobject' 'python-cairo' 'python-dbus'
         'python-evdev' 'python-pillow' 'python-pyscreenshot' 'python-opencv'
         'python-numpy' 'python-rich' 'gtk3' 'at-spi2-core')
makedepends=('python-build' 'python-installer' 'python-setuptools' 'python-wheel')
optdepends=('libwnck3: X11 window detection'
            'gtk-layer-shell: overlays on supported Wayland compositors'
            'grim: screenshots on supported Wayland compositors')
source=("$pkgname-$pkgver.tar.gz::$url/archive/refs/tags/$pkgver.tar.gz")
sha256sums=('d7d32ce84ff8f0ea06db258e62a4e96373bee6d80ce63f35eb2b7346fc2822f4')

prepare() {
  cd "$pkgname-$pkgver"
  # Package the user unit below instead of writing into the build user's HOME.
  sed -i '/cmdclass={"install": PostInstallCommand},/d' setup.py
  # Use Arch's maintained PyGObject rather than the upstream pip-only pin.
  sed -i 's/PyGObject==3.50.0/PyGObject>=3.50.0/' setup.py
}

build() {
  cd "$pkgname-$pkgver"
  python -m build --wheel --no-isolation
}

check() {
  cd "$pkgname-$pkgver"
  python -m compileall -q hints
  python -m hints.cli --version
}

package() {
  cd "$pkgname-$pkgver"
  python -m installer --destdir="$pkgdir" dist/*.whl
  install -Dm644 README.md "$pkgdir/usr/share/doc/$pkgname/README.md"

  install -dm755 "$pkgdir/usr/lib/systemd/user"
  cat > "$pkgdir/usr/lib/systemd/user/hintsd.service" <<'UNIT'
[Unit]
Description=Hints keyboard navigation daemon
PartOf=graphical-session.target
After=graphical-session.target

[Service]
Type=simple
ExecStart=/usr/bin/hintsd
Restart=on-failure
RestartSec=3

[Install]
WantedBy=graphical-session.target
UNIT

  # Grant the active local desktop session access before 73-seat-late.rules.
  install -dm755 "$pkgdir/usr/lib/udev/rules.d" "$pkgdir/usr/lib/modules-load.d"
  echo 'SUBSYSTEM=="misc", KERNEL=="uinput", TAG+="uaccess", OPTIONS+="static_node=uinput"' \
    > "$pkgdir/usr/lib/udev/rules.d/70-hints-uinput.rules"
  echo uinput > "$pkgdir/usr/lib/modules-load.d/hints.conf"
}
