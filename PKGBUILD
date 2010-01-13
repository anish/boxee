# Contributor: Prurigro
# Maintainer: Prurigro
# Patches/Fixes created from efforts by myself, sdnick484, jaydonoghue, anish, jpf, vrtladept, paulingham and others! (contact me if your work is here but you aren't mentioned)
# fribidi.patch fixes an issue where some people were having missing file issues related to the fribidi library
# gcc44.patch helps boxee compile on gcc 4.4
# boxee64.patch addresses a number of problems getting boxee to compile and run smoothly on x86_64
# flashlib.patch allows flashlib to compile, allowing us to get a little closer to a built-from-source flash solution
# pkgdesc from wikipedia :)

pkgname=boxee-source
pkgver=0.9.20.10263
_flashlib_pkgver=6684
pkgrel=1
pkgdesc="A freeware cross-platform media center software with social networking features that is a fork of the open source XBMC media center"
arch=('i686' 'x86_64')
license=( 'GPL' )
depends=('pulseaudio' 'php' 'alsa-lib' 'freetype2' 'glew' 'hal' 'jasper' 'libcdio' 'sdl_image' 'sdl_mixer' 'sdl_gfx' 'sdl_sound' 'fribidi' 'libgl' 'libmad' 'libxinerama' 'lzo2' 'mesa' 'unrar' 'smbclient' 'sqlite3' 'streamripper' 'libogg' 'python-pysqlite' 'curl' 'gawk' 'libxrandr' 'libxrender' 'pmount' 'libvorbis' 'libmysqlclient' 'pcre' 'dbus' 'fontconfig' 'bzip2' 'boost' 'libtool' 'faac' 'enca' 'libxt' 'libxmu' 'gperf' 'unzip' 'libpng' 'libjpeg' 'python24' 'tre' 'screen' 'bison' 'libsamplerate' 'nspr' 'nss' 'gtk2')
makedepends=( 'autoconf' 'boost' 'pkgconfig' 'gcc' 'make' 'ccache' 'automake' 'cmake' 'nasm' 'coreutils')
options=('!makeflags')
url="http://www.boxee.tv/"
source=(http://dl.boxee.tv/boxee-$pkgver-source.tar.bz2
	http://dl.boxee.tv/flashlib-shared-$_flashlib_pkgver.tar.gz
	boxee.desktop
	fribidi.patch
	boxee64.patch
	flashlib.patch
	anish.patch
)
md5sums=('477f522cb5a4eaeb6d3ea44c580e9b0a'
	'5762c99e2ac7485c7e2876e7c93afdc8'
	'b84c543ac1e5ff0f7d7c4b22b690e0b2'
	'b9ff2928d707321c96ef1ad792c14dda'
	'6fed9bb9ff72bd6bee9c6f75fddcd417'
	'c47f3eec8c8a0ef2a7fbaaca56678177'
	'a07e311b6da020f7e6847d249cf08b66'
)

_src=${srcdir}/boxee-"$pkgver"-source

build() {
	pushd ${_src} || return 1
		#this section contains changes required for x86_64 and thus is only loaded if your arch is x86_64
		if [ $(uname -m) = "x86_64" ]; then
			#boxee64.patch allows boxee to compile on 64bit systems
			patch -p0 < ../boxee64.patch || return 1
			#two symlinks added by paulingham that work with boxee64.patch to allow boxee to compile on x86_64
			pushd xbmc/lib/libsmb || return 1
				ln -s /usr/lib/libtalloc.so.1 libtalloc-x86_64-linux.a
				ln -s /usr/lib/libwbclient.so.0 libwbclient-x86_64-linux.a
			popd || return 1
		fi || return 1

		#anish.patch adds some minor tweaks anish figured out to get the latest sources running
		patch -p0 < ../anish.patch || return 1

		#fribidi.patch fixes the compile issue related to fribidi (big thanks to vrtladept and anish for getting this one rolling)
		patch -p0 < ../fribidi.patch || return 1

		#gcc44.patch makes the few necessary changes to compile on gcc 4.4 (thanks to anish for developing this one!)
#		patch -p0 < ../gcc44.patch || return 1

		#tinyxpath and goom need to be reconfigured so they link against the correct utilities (another thanks to anish for this one)
		pushd xbmc/lib/libBoxee/tinyxpath || return 1
			autoreconf -vif || return 1
			./configure || return 1
		popd || return 1
	
		pushd xbmc/visualizations/Goom/goom2k4-0 || return 1
			aclocal || return 1
			libtoolize --copy --force || return 1
			./autogen.sh --enable-static --with-pic || return 1
		popd || return 1
		
		aclocal || return 1
		autoheader || return 1
		autoconf || return 1
		./configure --prefix=/opt/boxee --enable-mid --disable-debug --enable-external-libass || return 1
	
		#this is another hack to fix an issue with gcc44-- once again I'm using sed because the Makefile is generated in this package
		if [ $(uname -m) = "x86_64" ]; then
			sed -r 's/\(MAKE\)\ -C\ xbmc\/screensavers$/\(MAKE\)\ CFLAGS=\"-march=k8\ -02\ -pipe\"\ -C\ xbmc\/screensavers/g' Makefile > Makefile.sed || return 1
		else
			sed -r 's/\(MAKE\)\ -C\ xbmc\/screensavers$/\(MAKE\)\ CFLAGS=\"-march=i486\ -02\ -pipe\"\ -C\ xbmc\/screensavers/g' Makefile > Makefile.sed || return 1
		fi
		cat Makefile.sed > Makefile || return 1
	
		make || return 1
	popd || return 1

	#language
	install -d ${pkgdir}/opt/boxee/language || return 1
	pushd ${_src}/language/ || return 1
		find . | sed -e 's/\.\///g' | while read file; do
			if [ -d "$file" ]; then
				install -d ${pkgdir}/opt/boxee/language/"$file" || return 1
			else
				install -D "$file" ${pkgdir}/opt/boxee/language/"$file" || return 1
			fi || return 1
		done || return 1
	popd || return 1
	
	#media
	install -d ${pkgdir}/opt/boxee/media || return 1
	pushd ${_src}/media/ || return 1
		find . | sed -e 's/\.\///g' | while read file; do
			if [ $(echo "$file" | grep "icon.png" -i -c) = 0 -a $(echo "$file" | grep "icon32x32.png" -i -c) = 0 -a $(echo "$file" | grep "xbmc.icns" -i -c) = 0 -a $(echo "$file" | grep "Boxee.ico" -i -c) = 0 -a $(echo "$file" | grep "Splash.png" -i -c) = 0 -a $(echo "$file" | grep "Splash_old.png" -i -c) = 0 -a $(echo "$file" | grep "Fonts/arial.ttf" -i -c) = 0 ]; then
				if [ -d "$file" ]; then
					install -d ${pkgdir}/opt/boxee/media/"$file" || return 1
				else
					install -D "$file" ${pkgdir}/opt/boxee/media/"$file" || return 1
				fi || return 1
			fi || return 1
		done || return 1
	popd || return 1
	
	#scripts
	install -d ${pkgdir}/opt/boxee/scripts || return 1
	pushd ${_src}/scripts || return 1
		find . | sed -e 's/\.\///g' | while read file; do
			if [ $(echo "$file" | grep "scripts.zip" -i -c) = 0 -a $(echo "$file" | grep "user_submitted.zip" -i -c) = 0 -a $(echo "$file" | grep "autoexec.py" -i -c) = 0 ]; then
				if [ -d "$file" ]; then
					install -d ${pkgdir}/opt/boxee/scripts/"$file" || return 1
				else
					install -D "$file" ${pkgdir}/opt/boxee/scripts/"$file" || return 1
				fi || return 1
			fi || return 1
		done || return 1
	popd || return 1

	#skin
	install -d ${pkgdir}/opt/boxee/skin/boxee || return 1
	pushd ${_src}/skin/boxee || return 1
		find . | sed -e 's/\.\///g' | while read file; do
#			if [ $(echo "$file" | grep -e "^media" -i -c) = 0 ]; then
				if [ -d "$file" ]; then
					install -d ${pkgdir}/opt/boxee/skin/boxee/"$file" || return 1
				else
					install -D "$file" ${pkgdir}/opt/boxee/skin/boxee/"$file" || return 1
				fi || return 1
#			fi || return 1
		done || return 1
#		install -d ${pkgdir}/opt/boxee/skin/boxee/media || return 1
	popd || return 1

	#system
	pushd ${_src}/system/python/local || return 1
#This isn't indented because whitespace is significant to python
python2.4 -O >/dev/null << EOF
import py_compile
py_compile.compile('mc.py')
EOF
	popd || return 1
	
	install -d ${pkgdir}/opt/boxee/system || return 1
	pushd ${_src}/system/ || return 1
		find . -path "./python/Lib" -prune -o -print | sed -e 's/\.\///g' | while read file; do
			if [ $(echo "$file" | grep "win32" -i -c) = 0 -a $(echo "$file" | grep "spyce" -i -c) = 0 -a $(echo "$file" | grep "DLLs" -i -c) = 0 -a $(echo "$file" | grep "osx" -i -c) = 0 -a $(echo "$file" | grep -e "\.dll$" -i -c) = 0 -a $(echo "$file" | grep -e "\.pyc$" -i -c) = 0 -a $(echo "$file" | grep "xulrunner" -i -c) = 0 -a $(echo "$file" | grep "etc" -i -c) = 0 -a $(echo "$file" | grep "python24.zlib" -i -c) = 0 -a $(echo "$file" | grep "upnpserver.xml" -i -c) = 0 -a $(echo "$file" | grep "IRSSmap.xml" -i -c) = 0 -a $(echo "$file" | grep "X10-Lola-IRSSmap.xml" -i -c) = 0 -a $(echo "$file" | grep "fontconfig_readme" -i -c) = 0 -a $(echo "$file" | grep "libmpeg2-i486-linux.so" -i -c) = 0 -a $(echo "$file" | grep "bxoverride.so" -i -c) = 0 -a $(echo "$file" | grep "readme.txt" -i -c) = 0 -a $(echo "$file" | grep "simplejson/_speedups.so" -i -c) = 0 ]; then
				if [ -d "$file" ]; then
					install -d ${pkgdir}/opt/boxee/system/"$file" || return 1
				else
					install -D "$file" ${pkgdir}/opt/boxee/system/"$file" || return 1
				fi || return 1
			fi || return 1
		done || return 1
	popd || return 1

	install -d ${pkgdir}/opt/boxee/system/players/flashplayer/xulrunner || return 1
	pushd ${_src}/system/players/flashplayer/xulrunner-i486-linux || return 1
		find . | sed -e 's/\.\///g' | while read file; do
			if [ -d "$file" ]; then
				install -d ${pkgdir}/opt/boxee/system/players/flashplayer/xulrunner/"$file" || return 1
			else
				install -D "$file" ${pkgdir}/opt/boxee/system/players/flashplayer/xulrunner/"$file" || return 1
			fi || return 1
		done || return 1
	popd || return 1
	
	install -d ${pkgdir}/opt/boxee/system/python/lib || return 1
	pushd ${_src}/xbmc/lib/libPython/Python/Lib || return 1
#This isn't indented because whitespace is significant to python
python2.4 -O >/dev/null << EOF
import compileall
compileall.compile_dir(".", force=1)
EOF
	find . | sed -e 's/\.\///g' | while read file; do
		if [ $(echo "$file" | grep -e "^test" -i -c) = 0 ]; then
			if [ -d "$file" ]; then
				install -d ${pkgdir}/opt/boxee/system/python/lib/"$file" || return 1
			elif [ ! $(echo "$file" | grep -e "\.pyo$" -i -c) = 0 ]; then
				install -D "$file" ${pkgdir}/opt/boxee/system/python/lib/"$file" || return 1
			fi || return 1
		fi || return 1
	done || return 1
	popd || return 1

	rmdir ${pkgdir}/opt/boxee/system/python/lib/plat-generic || return 1
	rmdir ${pkgdir}/opt/boxee/system/python/lib/email/test/data || return 1
	rmdir ${pkgdir}/opt/boxee/system/python/lib/plat-next3 || return 1
	rmdir ${pkgdir}/opt/boxee/system/python/lib/idlelib/Icons || return 1
	rmdir ${pkgdir}/opt/boxee/system/python/lib/site-packages || return 1

	#userdata
	install -d ${pkgdir}/opt/boxee/UserData || return 1
	install -D ${_src}/UserData/*linux* ${pkgdir}/opt/boxee/UserData/ || return 1
	ln -s UserData ${pkgdir}/opt/boxee/userdata || return 1
	
	#plugins
	install -d ${pkgdir}/opt/boxee/plugins/music || return 1
	install -d ${pkgdir}/opt/boxee/plugins/pictures || return 1
	install -d ${pkgdir}/opt/boxee/plugins/video || return 1

	#visualisations
	install -d ${pkgdir}/opt/boxee/visualisations/
	pushd ${_src}/visualisations/ || return 1
	for i in *; do
		if [ -d "$i" ]; then
			install -d ${pkgdir}/opt/boxee/visualisations/"$i" || return 1
			if [ $(ls "$i" | wc -l) != "0" ]; then
				install -D "$i"/* ${pkgdir}/opt/boxee/visualisations/"$i"/ || return 1
			fi || return 1
		else
			if [ $(echo "$i" | grep "osx" -c) = "0" -a $(echo "$i" | grep "win32" -c) = "0" -a $(echo "$i" | grep "Goom.vis" -c) = "0" -a $(echo "$i" | grep "xbmc_vis.h" -c) = "0" ]; then
				install -D "$i" ${pkgdir}/opt/boxee/visualisations/ || return 1
			fi || return 1
		fi || return 1
        done || return 1
	popd || return 1

	#screensavers
	install -d ${pkgdir}/opt/boxee/screensavers || return 1
	install -D ${_src}/screensavers/*.xbs ${pkgdir}/opt/boxee/screensavers/ || return 1

	#rtorrent
	install -d ${pkgdir}/opt/boxee/bin || return 1
	install -D ${_src}/bin-linux/boxee-rtorrent ${pkgdir}/opt/boxee/bin/ || return 1

	#boxee binary
	install -D ${_src}/Boxee ${pkgdir}/opt/boxee/ || return 1
	strip ${pkgdir}/opt/boxee/Boxee || return 1
	install -D ${_src}/run-boxee-desktop.in ${pkgdir}/opt/boxee/run-boxee-desktop || return 1

	#give_me_my_mouse_back
	gcc ${_src}/give_me_my_mouse_back.c -o ${_src}/give_me_my_mouse_back -lSDL || return 1
	install -D ${_src}/give_me_my_mouse_back ${pkgdir}/opt/boxee/ || return 2
	strip ${pkgdir}/opt/boxee/give_me_my_mouse_back || return 1
	
	#xbmc-xrandr
	install -D ${_src}/xbmc-xrandr ${pkgdir}/opt/boxee/ || return 1
	strip ${pkgdir}/opt/boxee/xbmc-xrandr || return 1
	
	#freedesktop
	install -d ${pkgdir}/usr/share/applications || return 1
	install -D ${srcdir}/boxee.desktop ${pkgdir}/usr/share/applications/ || return 1
	install -d ${pkgdir}/usr/share/pixmaps || return 1
	install -D ${_src}/media/icon.png ${pkgdir}/usr/share/pixmaps/boxee.png || return 1

	#some symlinks found thanks to the impressive debugging skills of anish, and a few others from me that will allow bxflplayer-linux to run finally!
	install -d ${pkgdir}/usr/lib || return 1
	ln -s /usr/lib/libsmime3.so ${pkgdir}/usr/lib/libsmime3.so.1d || return 1
	ln -s /usr/lib/libssl3.so ${pkgdir}/usr/lib/libssl3.so.1d || return 1
	ln -s /usr/lib/libnss3.so ${pkgdir}/usr/lib/libnss3.so.1d || return 1
	ln -s /usr/lib/libnssutil3.so ${pkgdir}/usr/lib/libnssutil3.so.1d || return 1
	ln -s /usr/lib/libplds4.so ${pkgdir}/usr/lib/libplds4.so.0d || return 1
	ln -s /usr/lib/libplc4.so ${pkgdir}/usr/lib/libplc4.so.0d || return 1
	ln -s /usr/lib/libnspr4.so ${pkgdir}/usr/lib/libnspr4.so.0d || return 1

}