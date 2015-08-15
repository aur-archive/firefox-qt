# Maintainer: Limao Luo <luolimao@gmail.com>
# Contributor: Ionut Biru <ibiru@archlinux.org>
# Contributor: Jakub Schmidtke <sjakub@gmail.com>
# Contributor: Franco Tortoriello

# You may need to uninstall your current firefox installation before building this package!
pkgname=firefox-qt
pkgver=14.0.1
pkgrel=1
pkgdesc="Web browser from mozilla.org, using Qt (experimental)"
arch=(i686 x86_64)
url="http://www.mozilla.org/projects/firefox"
license=(MPL GPL LGPL)
depends=(qt mozilla-common libxt startup-notification mime-types
    dbus-glib libnotify libvpx libevent pango 'nss>=3.13.3'
    hunspell desktop-file-utils)
makedepends=(unzip python2 wireless_tools yasm mesa
    autoconf2.13 libidl2 imake upx)
optdepends=('wireless_tools: Location detection via available WiFi networks'
    firefox-oxygen-kde)
provides=("firefox=$pkgver")
conflicts=(firefox)
options=(!emptydirs upx)
install=$pkgname.install
source=(
    https://ftp.mozilla.org/pub/mozilla.org/firefox/releases/$pkgver/source/firefox-$pkgver.source.tar.bz2
    mozconfig
    firefox.desktop 
    firefox-install-dir.patch
    vendor.js
    gcc47.patch)
sha256sums=('c21988f0207b678376c3d96f647aadf6d694e836f0b5c933ec15d93b75ea89aa'
    '87de105f83aea399bf8d565c5041d9686e11921dc967e220f8d7cdaa14c7394f'
    'cfb1b3442d94650ee3629d7a0a860f6e0ffd337773e4c25ec39c3a061af673f9'
    '3db7fa9de38b21da92eb3fe2923cd23584c526c6174d984ff7a777fb04a1270b'
    '4b50e9aec03432e21b44d18c4c97b2630bace606b033f7d556c9d3e3eb0f4fa4'
    'a492b183b9ac1d981803adb2c10c491e9986e37ca91f7beba4631115c6b5333b')
sha512sums=('1d7c5f7138858fcc97bdcbc305b2a68f0210d75b9cf2eb1a5cd1825752c6a5d84cdd91fa2f3abdb200f38b7a295f3415d99225e5dd2517aec9522b4628c4f323'
    '0d0724300653135a0546483e1b018928d49ee2b297b1028c14d4c9c681ddb338c68f61272f8252692e0a11db221018c291e0f0fdce702c8726559a48cf420686'
    'c59aa85449fc8ba63860bec688b82749149deb113c3013b92803d326c87d9ca6f16c66529d414cd691c5ab211a844afce0b51befa4b2ac5f47013166675be25e'
    '80729174b40793b9774c5e5618bdcc161df93575c1de6491394e37f45a28127873e97739a97b4d727cbb34420f52f35078816cc807799611cd7b2036d90e74f9'
    'd927e5e882115c780aa0d45034cb1652eaa191d95c15013639f9172ae734245caae070018465d73fdf86a01601d08c9e65f28468621422d799fe8451e6175cb7'
    'e0873328cc5a703af67af559ec94fcb00e5f8da63652192e960fae9491b57871b438ffe73ef1a72f071963827a47b9eeb59de8c7caeaf045eb91e9b0681c690f')

build() {
    cd $srcdir/mozilla-release/

    cp ../mozconfig .mozconfig
    patch -Np1 -i ../firefox-install-dir.patch
    patch -Np1 -i ../gcc47.patch

    # Fix PRE_RELEASE_SUFFIX
    sed -i '/^PRE_RELEASE_SUFFIX := ""/s/ ""//' browser/base/Makefile.in

    export LDFLAGS="$LDFLAGS -Wl,-rpath,/usr/lib/firefox"
    export PYTHON=python2
    export MOZ_MAKE_FLAGS="$MAKEFLAGS"
    unset MAKEFLAGS

    # Enable PGO
    export MOZ_PGO=1
    export DISPLAY=:99
    Xvfb -nolisten tcp -extension GLX -screen 0 1280x1024x24 $DISPLAY &

    make -f client.mk build
    kill $!
}

package() {
    cd $srcdir/mozilla-release
    make -j1 -f client.mk DESTDIR=$pkgdir install

    install -m644 ../vendor.js $pkgdir/usr/lib/firefox/defaults/pref

    for i in 16 22 24 32 48 256; do
	install -Dm644 browser/branding/official/default$i.png $pkgdir/usr/share/icons/hicolor/${i}x${i}/apps/firefox.png
    done

    # Desktop file
    desktop-file-install ../firefox.desktop --dir=$pkgdir/usr/share/applications/
    
    # Use system-provided dictionaries
    rm -rf $pkgdir/usr/lib/firefox/{dictionaries,hyphenation}
    ln -s /usr/share/hunspell $pkgdir/usr/lib/firefox/dictionaries
    ln -s /usr/share/hyphen $pkgdir/usr/lib/firefox/hyphenation

    # We don't want the development stuff
    rm -r $pkgdir/usr/{include,lib/firefox-devel,share/idl}

    #workaround for now
    #https://bugzilla.mozilla.org/show_bug.cgi?id=658850
    ln -sf firefox $pkgdir/usr/lib/firefox/firefox-bin
}
