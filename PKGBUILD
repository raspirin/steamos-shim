# Maintainer: aspirin <f1193968991@126.com>
pkgname=steamos-shim
pkgver=1.0.0
pkgrel=1
pkgdesc="Multi-call shim that lets non-SteamOS Linux launch Steam in SteamOS mode via Gamescope"
arch=('any')
url="https://github.com/raspirin/steamos-shim"
license=('MIT')
depends=('bash' 'gamescope' 'steam')
optdepends=('mangohud: performance overlay in SteamOS mode'
            'lib32-mangohud: 32-bit performance overlay support')
conflicts=('gamescope-session-steam'
           'gamescope-session-steam-git')
source=('steamos-shim'
        'steam.desktop'
        'LICENSE')
sha256sums=('SKIP'
            'SKIP'
            'SKIP')

package() {
    cd "$srcdir"

    # Multi-call dispatcher (basename "$0" selects the command)
    install -Dm755 steamos-shim "$pkgdir/usr/bin/steamos-shim"

    # Symlinks in /usr/bin that the Steam client invokes by name
    local cmd
    for cmd in \
        gamescope-session \
        jupiter-biosupdate \
        steamos-select-branch \
        steamos-session-select \
        steamos-update
    do
        ln -s steamos-shim "$pkgdir/usr/bin/$cmd"
    done

    # Polkit-helper subdirectory; Steam client looks here specifically
    install -dm755 "$pkgdir/usr/bin/steamos-polkit-helpers"
    for cmd in \
        jupiter-biosupdate \
        steamos-set-timezone \
        steamos-update
    do
        ln -s ../steamos-shim "$pkgdir/usr/bin/steamos-polkit-helpers/$cmd"
    done

    # Display-manager session entry
    install -Dm644 steam.desktop \
        "$pkgdir/usr/share/wayland-sessions/steam.desktop"

    # License
    install -Dm644 LICENSE \
        "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
