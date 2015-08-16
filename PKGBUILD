# Maintainer: Lone_Wolf <lonewolf@xs4all.nl>
# Contributor : Jan de Groot <jgc@archlinux.org>
# Contributor: Andreas Radke <andyrtr@archlinux.org>
# Adapted from offical archlinux mesa package

pkgname=mesa-r600g-git
pkgver=20121120
pkgrel=1
_realver=9.1
pkgdesc="Mesa for R300 & R600 chipsets - git version. If you live in the US, you should delete --enable-texture-float \ line."
arch=('x86_64')
depends=('libxt' 'libxxf86vm' 'libxdamage' 'xorg-server' 'libxv' 'libffi' 'gcc-libs')
makedepends=('pkgconfig' 'python2' 'talloc' 'libxml2' 'imake' 'git' 'glproto' 'dri2proto>=2.6' 'llvm-amdgpu-git' 'xorg-server-devel' 'libxvmc' 'resourceproto')
optdepends=('libtxc_dxtn: S3TC support'
            'mesa-demos-git: glxinfo and glxgears')
provides=(mesa=${_realver} libgl=${_realver} ati-dri=${_realver} libglapi=${_realver} libegl=${_realver} khrplatform-devel=${_realver} libgles=${_realver} libgbm=${_realver})
conflicts=('xf86-video-ati<6.9.0-6' mesa libgl ati-dri libglapi libegl khrplatform-devel libgles libgbm)
url="http://mesa3d.sourceforge.net"
license=(custom)
source=(LICENSE)
md5sums=('5c65a0fe315dd347e09b1f2826a1df5a')
_gitroot='git://anongit.freedesktop.org./git/mesa/mesa'
_gitname='mesa'

build() {
  cd "$srcdir"
  msg 'Connecting to git.freedesktop.org GIT server....'
  if [[ -d "$_gitname" ]]; then
    cd "$_gitname" && git pull origin
    msg "The local files are updated."
  else
    git clone --depth=1 "$_gitroot" "$_gitname"
  fi
  msg 'GIT checkout done or server timeout'
  msg 'Starting make...'

  cd "${srcdir}"
 # Cleanup and prepare the build dir
  [ -d $_gitname-build ] && rm -rf $_gitname-build
  mkdir $_gitname-build
  cd "$srcdir/$_gitname" && ls -A | grep -v .git | xargs -d '\n' cp -r -t ../$_gitname-build # do not copy over the .git folder
  cd "$srcdir/$_gitname-build"
    autoreconf -vfi
    ./autogen.sh --prefix=/usr \
    --with-dri-driverdir=/usr/lib/xorg/modules/dri \
    --with-gallium-drivers=r300,r600,swrast \
    --with-dri-drivers=swrast \
    --enable-texture-float \
    --enable-glx-tls \
    --enable-shared-glapi \
    --enable-gles1 \
    --enable-gles2 \
    --enable-gbm \
    --enable-gallium-gbm \
    --enable-xvmc \
    --enable-gallium-g3dvl \
    --enable-osmesa \
    --enable-xorg \
    --enable-gallium-egl \
    --enable-r600-llvm-compiler \
    --enable-xa
# left out compile flags
# default = auto 
#  --enable-32-bit	       build 32-bit libraries [default=auto]
#  --enable-64-bit	       build 64-bit libraries [default=auto]
#  --enable-vdpau          enable vdpau library [default=auto]
#  --disable-driglx-direct enable direct rendering in GLX and EGL for DRI [default=auto]
#  --enable-va             enable va library [default=auto]

#
# default = enabled/yes
#  --enable-shared[=PKGS]       build shared libraries [default=yes]
#  --enable-fast-install[=PKGS] optimize for fast installation [default=yes]
#  --disable-asm	              disable assembly usage [default=enabled on supported plaforms]
#  --disable-pic                compile PIC objects [default=enabled for shared builds on supported platforms]
#  --enable-dri                 enable DRI modules [default=enabled]
#  --enable-glx                 enable GLX library [default=auto]
#  --disable-egl                disable EGL library [default=enabled]
#  --enable-gallium-llvm        build gallium LLVM support [default=enabled on x86/x86_64]
#
#
# default = disabled/no
#  --enable-debug          use debug compiler flags and macros [default=disabled]
#  --enable-mangling       enable mangled symbols and library name [default=disabled]
#  --enable-selinux	       Build SELinux-aware Mesa [default=disabled]
#  --disable-opengl	       disable support for standard OpenGL API [default=no]
#  --enable-opencl         enable OpenCL library [default=no]
#  --enable-xlib-glx       make GLX library Xlib-based instead of DRI-based [default=disable]
#  --enable-gallium-egl    enable optional EGL state tracker (not required for
#                          EGL support in Gallium with OpenGL and OpenGL ES)
#                          (default=disable]
#  --enable-gallium-tests  Enable optional Gallium tests) [default=disable]
#  --enable-gallium-g3dvl  build gallium g3dvl [default=disabled]
#

  make -j1
}

package() {

# Mesa
cd "${srcdir}/$_gitname-build" 
  make DESTDIR="${pkgdir}" install
  rm -f "${pkgdir}/usr/lib/libGL.so"*
  rm -f "${pkgdir}/usr/lib/libGLESv"*
  rm -f "${pkgdir}/usr/lib/libEGL"*
  rm -rf "${pkgdir}/usr/lib/egl"
  rm -rf "${pkgdir}/usr/lib/xorg"
  rm -f "${pkgdir}/usr/include/GL/glew.h"
  rm -f "${pkgdir}/usr/include/GL/glxew.h"
  rm -f "${pkgdir}/usr/include/GL/wglew.h"
  rm -f "${pkgdir}/usr/include/GL/glut.h"
# libgl
  install -m755 -d "${pkgdir}/usr/lib/xorg/modules/extensions"
  bin/minstall lib/libGL.so* "${pkgdir}/usr/lib/"
  bin/minstall lib/libdricore*.so* "${pkgdir}/usr/lib/"
#  bin/minstall lib/libglsl.so* "${pkgdir}/usr/lib/"
  cd "src/mesa/drivers/dri"
  make -C "${srcdir}/$_gitname-build/src/gallium/targets/dri-swrast" DESTDIR="${pkgdir}" install
  ln -s libglx.xorg "${pkgdir}/usr/lib/xorg/modules/extensions/libglx.so"
# libglapi
  cd "${srcdir}/$_gitname-build" 
  install -m755 -d "${pkgdir}/usr/lib"
  bin/minstall lib/libglapi.so* "${pkgdir}/usr/lib/"
# libegl
  make -C src/gallium/targets/egl-static DESTDIR="${pkgdir}" install
  bin/minstall lib/libEGL.so* "${pkgdir}/usr/lib/"
  install -m755 -d "${pkgdir}/usr/lib/egl"
  bin/minstall lib/egl/* "${pkgdir}/usr/lib/egl/"
  bin/minstall src/egl/main/egl.pc "${pkgdir}/usr/lib/pkgconfig/"
  bin/minstall include/EGL/* "${pkgdir}/usr/include/EGL/"
  install -m755 -d "${pkgdir}/usr/share/doc"
  install -m755 -d "${pkgdir}/usr/share/doc/libegl"
  bin/minstall docs/egl.html "${pkgdir}/usr/share/doc/libegl/"
# khrplatform-devel
  install -m755 -d "${pkgdir}/usr/include/KHR"
  bin/minstall include/KHR/khrplatform.h "${pkgdir}/usr/include/KHR/"
# ati-dri
  cd "${srcdir}/$_gitname-build/src/mesa/drivers/dri"
  make -C "${srcdir}/$_gitname-build/src/gallium/targets/dri-r300" DESTDIR="${pkgdir}" install
  make -C "${srcdir}/$_gitname-build/src/gallium/targets/dri-r600" DESTDIR="${pkgdir}" install
#license
  install -m755 -d "${pkgdir}/usr/share/licenses/${pkgname}"
  install -m755 "${srcdir}/LICENSE" "${pkgdir}/usr/share/licenses/${pkgname}/"
}
