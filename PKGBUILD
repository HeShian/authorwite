# SPDX-License-Identifier: 0BSD
pkgname=authorwrite-bin
pkgver=5.24.0
pkgrel=4
pkgdesc='NovelBuddy (作家助手), a desktop writing tool by Yuewen'
arch=('x86_64' 'aarch64')
url='https://zuojia.write.qq.com/'
license=('LicenseRef-proprietary')
depends=('alsa-lib' 'at-spi2-core' 'cairo' 'dbus' 'expat' 'gcc-libs' 'glib2'
         'glibc' 'gtk3' 'hicolor-icon-theme' 'libcups' 'libnotify' 'libsecret'
         'libx11' 'libxcb' 'libxcomposite' 'libxdamage' 'libxext' 'libxfixes'
         'libxkbcommon' 'libxrandr' 'libxss' 'libxtst' 'mesa' 'nspr' 'nss' 'pango'
         'systemd-libs' 'util-linux-libs' 'xdg-utils')
makedepends=('asar' 'jq')
optdepends=('libappindicator-gtk3: system tray integration')
provides=("authorwrite=$pkgver")
conflicts=('authorwrite')
options=('!strip' '!debug')

_deb_x86_64="authorwrite_${pkgver}_amd64.deb"
_deb_aarch64="authorwrite_${pkgver}_linuxarm64.deb"
source=('LICENSE' 'authorwrite-linux.patch')
source_x86_64=("authorwrite://linux_X64-deb/$_deb_x86_64")
source_aarch64=("authorwrite://linux_ARM-deb/$_deb_aarch64")
noextract=("$_deb_x86_64" "$_deb_aarch64")
sha256sums=('29f8133b412608fe8c9f8652f2694cc56c086f9a275b4cf045dbfa69a21749f0'
            '6c2912f965293c929b19037bbe5fb5f4dc10a932653d677ef69e8c51eb720d6a')
sha256sums_x86_64=('b4affcd66059f6aa84a47d6791ecfa6b6bcb60167047a4cf83879a98867cdad8')
sha256sums_aarch64=('335cc56ec73398ce2bf57870d4669f118955320bfd7f5153222429ffae993f2d')

# Yuewen issues short-lived signed links. Fetch one at download time.
_authorwrite_download() {
  # bash -c sets $0 to the URL supplied by makepkg.
  local source_url=$0
  local filename=${source_url##*/}
  local download_type=${source_url#authorwrite://}
  download_type=${download_type%%/*}

  local timestamp signature response download_url
  timestamp=$(date +%s) || return 1
  signature=$(printf '%s%s' "$timestamp" "$download_type" | md5sum) || return 1
  signature=${signature%% *}

  local homepage='https://zuojia.write.qq.com/'
  local endpoint='https://write.qq.com/ccauthorweb/desk/author/getApk'
  response=$(curl -qfsS --proto '=https' --retry 2 --max-time 30 \
    --referer "$homepage" --header 'Origin: https://zuojia.write.qq.com' \
    "$endpoint?downType=$download_type&sign=$signature&timestamp=$timestamp") || return 1
  download_url=$(jq -er 'select(.returnCode == 2000) | .result.downUrl | strings |
    select(startswith("https://author-appapkcdn.yuewen.com/"))' <<< "$response") || {
    printf 'Could not get an official download URL for %s.\n' "$filename" >&2
    return 1
  }
  if [[ ${download_url%%\?*} != */"$filename" ]]; then
    printf 'Yuewen has changed its published release. This package needs %s.\n' "$filename" >&2
    return 1
  fi

  curl -qfLSs --proto '=https' --proto-redir '=https' --retry 2 \
    --connect-timeout 20 --max-time 600 --remove-on-error --referer "$homepage" \
    --output "$filename.part" "$download_url" || return 1
  mv -f -- "$filename.part" "$filename"
}
export -f _authorwrite_download
DLAGENTS+=('authorwrite::/usr/bin/bash -c _authorwrite_download %u')

prepare() {
  local deb="_deb_$CARCH"
  local upstream="$srcdir/upstream"
  local archive="$upstream/opt/作家助手/resources/app/authorwrite_${pkgver}.asar"
  local asar_work="$srcdir/asar-work"

  rm -rf -- "$upstream" "$asar_work"
  mkdir -p "$upstream" "$asar_work"
  bsdtar -xOf "$srcdir/${!deb}" data.tar.xz |
    bsdtar -xpf - --no-same-owner -C "$upstream"

  asar extract "$archive" "$asar_work/app"
  patch --directory="$asar_work/app" --strip=1 --fuzz=0 --no-backup-if-mismatch \
    --input="$srcdir/authorwrite-linux.patch"
  asar pack "$asar_work/app" "$asar_work/authorwrite.asar" --unpack '*.node'
  mv -f "$asar_work/authorwrite.asar" "$archive"
  rm -rf -- "$asar_work"
}

package() {
  cp -a --no-preserve=ownership "$srcdir/upstream/." "$pkgdir/"

  install -d "$pkgdir/usr/bin"
  # A warm Chromium HTTP cache can leave the upstream sign-in frame loading.
  cat > "$pkgdir/usr/bin/authorwrite" <<'EOF'
#!/bin/sh
exec '/opt/作家助手/authorwrite' --disable-http-cache "$@"
EOF
  chmod 755 "$pkgdir/usr/bin/authorwrite"

  # Launch through PATH and use Chromium's default sandbox on Arch.
  sed -i -e '/^Encoding=/d' -e 's|^Exec=.*|Exec=/usr/bin/authorwrite %U|' \
    -e 's/^StartupNotify=.*/StartupNotify=true/' -e 's/^StartupWMClass=.*/StartupWMClass=authorwrite/' \
    "$pkgdir/usr/share/applications/authorwrite.desktop"
}
