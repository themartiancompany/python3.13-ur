# SPDX-License-Identifier: AGPL-3.0

#    ----------------------------------------------------------------------
#    Copyright © 2024, 2025, 2026  Pellegrino Prevete
#
#    All rights reserved
#    ----------------------------------------------------------------------
#
#    This program is free software: you can redistribute it and/or modify
#    it under the terms of the GNU Affero General Public License as
#    published by the Free Software Foundation, either version 3 of the
#    License, or (at your option) any later version.
#
#    This program is distributed in the hope that it will be useful,
#    but WITHOUT ANY WARRANTY; without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#    GNU Affero General Public License for more details.
#
#    You should have received a copy of the GNU Affero General Public
#    License along with this program.
#    If not, see <https://www.gnu.org/licenses/>.

# Maintainers:
#   Truocolo
#     <truocolo@aol.com>
#     <truocolo@0x6E5163fC4BFc1511Dbe06bB605cc14a3e462332b>
#   Pellegrino Prevete (dvorak)
#     <pellegrinoprevete@gmail.com>
#     <dvorak@0x87003Bd6C074C713783df04f36517451fF34CBEf>
#   sneekyfoxx
#     <sneekyfoxx09@gmail.com>

_os="$(
  uname \
    -o)"
if [[ "${_os}" == "GNU/Linux" ]]; then
  _c_compilers=(
    "gcc"
  )
  _libc="gcc-libs"
elif [[ "${_os}" == "Android" ]]; then
  _c_compilers=(
    "clang"
    "llvm"
  )
  _libc="llvm-libs"
fi
_Pkg=Python
_pkg=python
_pymajver=3
_pyminver=13
_pkgver="${_pymajver}.${_pyminver}"
pkgname="${_pkg}${_pkgver}"
pkgver="${_pkgver}.0"
pkgrel=0
_pyver="${pkgver}"
_pybasever="${_pkgver}"
_pkgdesc=(
  "${_Pkg} programming language interpreter"
)
arch=(
  'aarch64'
  'arm'
  'i686'
  'mips'
  'x86_64'
)
license=(
  'PSF-2.0'
)
url="https://www.${_pkg}.org"
depends=(
  'bzip2'
  'expat'
  "${_c_compilers[@]}"
  "${_libc}"
  'gdbm'
  'libffi'
  'libnsl'
  'libxcrypt'
  'openssl'
  'zlib'
)
makedepends=(
  'bluez-libs'
  'gdb'
  'mpdecimal'
)
_mpdecimal_optdepends=(
  'mpdecimal:'
    'for decimal' 'xz: for lzma' 'tk: for tkinter'
)
optdepends=(
  'sqlite'
  "${_mpdecimal_optdepends[@]}"
)
_tarname="${_Pkg}-${pkgver}"
_tarfile="${_tarname}.tar.xz"
source=(
  "${url}/ftp/${_pkg}/${_pyver}/${_tarfile}"
)
md5sums=(
  '726e5b829fcf352326874c1ae599abaa'
)

prepare() {
  local \
    _modules_dir \
    _rm_args=()
  _modules_dir="Modules"
  _rm_args+=(
    "${_modules_dir}/expat"
    "${_modules_dir}/zlib"
    "${_modules_dir}/_ctypes/"{"darwin","libffi"}*
    "${_modules_dir}/_decimal/libmpdec"
  )
  cd \
    "${srcdir}/${_tarname}"
  # Ensure that we are using the system copy of various libraries (expat, zlib and libffi),
  # rather than copies shipped in the tarball
  rm \
    -rf \
    "${_rm_args[@]}"
}

build() {
  local \
    _cflags=() \
    _configure_opts=()
  _cflags+=(
    ${CFLAGS}
    -fno-semantic-interposition
    -fno-omit-frame-pointer
    -mno-omit-leaf-frame-pointer
  )
  _configure_opts+=(
    ax_cv_c_float_words_bigendian="no"
    --prefix="/usr"
    --enable-shared
    --with-computed-gotos
    --with-lto
    --enable-ipv6
    --with-system-expat
    --with-dbmliborder="gdbm:ndbm"
    --with-mimalloc
    --with-system-libmpdec
    --enable-loadable-sqlite-extensions
    --with-ensurepip
    --enable-optimizations
    --disable-gil
    --enable-experimental-jit="yes"
    --with-tzpath="/usr/share/zoneinfo"
  )
  cd \
    "${srcdir}/${_tarname}"
  CFLAGS="${_cflags[*]}"
  ./configure \
    "${_configure_opts[@]}"
  make \
    EXTRA_CFLAGS="$CFLAGS"
}

package() {
  cd \
    "${srcdir}/${_tarname}"
  # altinstall:
  #   /usr/bin/pythonX.Y but
  #   not /usr/bin/python or /usr/bin/pythonX
  make \
    DESTDIR="${pkgdir}" \
    altinstall \
    maninstall
  # Split tests
  rm \
    -r \
    "${pkgdir}/usr/lib/python"*"/"{"test","idlelib/idle_test"}
  # Avoid conflicts with the main 'python' package.
  rm \
    -f \
    "${pkgdir}/usr/lib/libpython${_pymajver}.so" \
    "${pkgdir}/usr/share/man/man1/python${_pymajver}.1"
  # Clean-up reference to build directory
  sed \
    -i \
    "s|$srcdir/${_tarname}:||" \
    "${pkgdir}/usr/lib/${_pkg}${_pybasever}/config-${_pybasever}-${CARCH}-linux-gnu/Makefile"
  # Add useful scripts FS#46146
  install \
    -vdm755 \
    "${pkgdir}/usr/lib/python${_pybasever}/Tools/"{"i18n","scripts"}
  install \
    -vm755 \
    "Tools/i18n/"{"msgfmt","pygettext"}.py \
    "${pkgdir}/usr/lib/${_pkg}${_pybasever}/Tools/i18n/"
  install \
    -vm755 \
    "Tools/scripts/"{"README",*"py"} \
    "${pkgdir}/usr/lib/${_pkg}${_pybasever}/Tools/scripts/"
  # License
  install \
    -vDm644 \
    "LICENSE" \
    "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}
