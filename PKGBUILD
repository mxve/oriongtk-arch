# Maintainer: mary
pkgname=oriongtk
pkgver=0.2.0
pkgrel=1
pkgdesc="Orion Web Browser by Kagi (repackaged from Flatpak)"
arch=('x86_64')
url="https://kagi.com/orion/"
license=('custom:commercial')
depends=(
  'gtk4'
  'libadwaita'
  'libsoup3'
  'libsecret'
  'glib2'
  'cairo'
  'pango'
  'gdk-pixbuf2'
  'harfbuzz'
  'vulkan-icd-loader'
  'graphene'
  'icu77'
  'gstreamer'
  'gst-plugins-base'
  'gst-plugins-good'
  'libepoxy'
)
makedepends=('flatpak-extract' 'ostree' 'patchelf')
source=("oriongtk-${pkgver}.flatpak::https://orionbrowser.com/download/oriongtk-early-beta")
sha256sums=('SKIP')
options=('!strip')

prepare() {
  cd "$srcdir"
  rm -rf extracted
  flatpak-extract --outdir extracted "oriongtk-${pkgver}.flatpak"

  # Patch all ELF files: fix rpath and hardcoded /app/ paths
  # Done in prepare() because fakeroot blocks subshell-based find loops
  cd extracted/files
  for f in bin/oriongtk bin/WebKitWebDriver \
           lib64/*.so.* \
           lib64/webkitgtk-6.0/injected-bundle/*.so \
           libexec/webkitgtk-6.0/*; do
    [ -f "$f" ] || continue
    file "$f" | grep -q ELF || continue
    patchelf --set-rpath /opt/lib64 "$f" 2>/dev/null || true
    sed -i 's|/app/|/opt/|g' "$f"
  done
}

package() {
  cd "$srcdir/extracted/files"

  # Install binaries
  install -dm755 "$pkgdir/opt/bin"
  install -Dm755 bin/oriongtk "$pkgdir/opt/bin/oriongtk"
  install -Dm755 bin/WebKitWebDriver "$pkgdir/opt/bin/WebKitWebDriver"

  # Install bundled libraries
  install -dm755 "$pkgdir/opt/lib64"
  for lib in lib64/*.so*; do
    [ -L "$lib" ] && cp -a "$lib" "$pkgdir/opt/lib64/" || install -Dm755 "$lib" "$pkgdir/opt/lib64/$(basename "$lib")"
  done

  # Install injected bundle
  install -dm755 "$pkgdir/opt/lib64/webkitgtk-6.0/injected-bundle"
  install -Dm755 lib64/webkitgtk-6.0/injected-bundle/libwebkitgtkinjectedbundle.so \
    "$pkgdir/opt/lib64/webkitgtk-6.0/injected-bundle/libwebkitgtkinjectedbundle.so"

  # Install GI typelibs
  install -dm755 "$pkgdir/opt/lib/x86_64-linux-gnu/girepository-1.0"
  install -Dm644 lib/x86_64-linux-gnu/girepository-1.0/*.typelib \
    -t "$pkgdir/opt/lib/x86_64-linux-gnu/girepository-1.0/"

  # Install WebKit helper processes
  install -dm755 "$pkgdir/opt/libexec/webkitgtk-6.0"
  for helper in libexec/webkitgtk-6.0/*; do
    install -Dm755 "$helper" "$pkgdir/opt/libexec/webkitgtk-6.0/$(basename "$helper")"
  done

  # Install locale/share files (binary references /opt/share after patching)
  install -dm755 "$pkgdir/opt/share"
  cp -a share/locale "$pkgdir/opt/share/locale"
  find "$pkgdir/opt/share/locale" -type d -exec chmod 755 {} +
  find "$pkgdir/opt/share/locale" -type f -exec chmod 644 {} +

# Install wrapper script
  install -dm755 "$pkgdir/usr/bin"
  cat > "$pkgdir/usr/bin/oriongtk" << 'WRAPPER'
#!/bin/bash
export LD_LIBRARY_PATH="/opt/lib64${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}"
exec /opt/bin/oriongtk "$@"
WRAPPER
  chmod 755 "$pkgdir/usr/bin/oriongtk"

  # Install desktop file (patched for our paths)
  install -Dm644 share/applications/com.kagi.OrionGtk.desktop \
    "$pkgdir/usr/share/applications/com.kagi.OrionGtk.desktop"

  # Install icons
  for size in 16x16 32x32 64x64 128x128 256x256; do
    install -Dm644 "share/icons/hicolor/${size}/apps/com.kagi.OrionGtk.png" \
      "$pkgdir/usr/share/icons/hicolor/${size}/apps/com.kagi.OrionGtk.png"
  done

  # Install metainfo
  install -Dm644 share/metainfo/com.kagi.OrionGtk.metainfo.xml \
    "$pkgdir/usr/share/metainfo/com.kagi.OrionGtk.metainfo.xml"
}
