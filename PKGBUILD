# Contributor: Prurigro
# Maintainer: Prurigro
# Initial framework taken from a post by the user 'crom' @ http://forum.boxee.tv/showthread.php?t=2309&highlight=gentoo
# REALLOC hack ported from a fix by [vEX] for xbmc
# Patches/Fixes created from efforts by myself, sdnick484, jaydonoghue, anish, jpf, vrtladept and others! (contact me if your work is here but you aren't mentioned)
# fribidi.patch fixes an issue where some people were having missing file issues related to the fribidi library
# gcc44.patch helps boxee compile on gcc 4.4
# boxee64.patch addresses a number of problems getting boxee to compile and run smoothly on x86_64
# flashlib.patch allows flashlib to compile, allowing us to get a little closer to a built-from-source flash solution
# pkgdesc from wikipedia :)

pkgname=boxee-source
pkgver=0.9.12.6570
pkgver2=0.9.12.6569
_flashlib_pkgver=5765
pkgrel=12
pkgdesc="A freeware cross-platform media center software with social networking features that is a fork of the open source XBMC media center"
arch=('i686' 'x86_64')
license=( 'GPL' )
depends=('php' 'alsa-lib' 'freetype2' 'glew' 'hal' 'jasper' 'libcdio' 'sdl_image' 'sdl_mixer' 'sdl_gfx' 'sdl_sound' 'fribidi' 'libgl' 'libmad' 'libxinerama' 'lzo2' 'mesa' 'unrar' 'smbclient' 'sqlite3' 'streamripper' 'libogg' 'python-pysqlite' 'curl' 'gawk' 'libxrandr' 'libxrender' 'pmount' 'libvorbis' 'libmysqlclient' 'pcre' 'dbus' 'fontconfig' 'bzip2' 'boost' 'libtool' 'faac' 'enca' 'libxt' 'libxmu' 'gperf' 'unzip' 'libpng' 'libjpeg' 'python24' 'tre' 'screen')
makedepends=( 'autoconf' 'boost' 'pkgconfig' 'gcc' 'make' 'ccache' 'automake' 'cmake' 'nasm' 'coreutils' )
options=('!makeflags')
url="http://www.boxee.tv/"
source=(http://dl.boxee.tv/boxee-$pkgver-sources.tar.bz2
	http://dl.boxee.tv/flashlib-shared-$_flashlib_pkgver.tar.gz
	xbmctex.tar.gz
	boxee.desktop
	fribidi.patch
	gcc44.patch
	boxee64.patch
	flashlib.patch
	conf.patch
)
md5sums=('a4ea53c6dfe2ae6f37e58d141312c2ec'
	'510d09c64dcdab1352ee3b441f5bf9a0'
	'12b9dfc3d4bf4b9404793b80d26934f2'
	'b84c543ac1e5ff0f7d7c4b22b690e0b2'
	'ed31f52918b56ed7bfd46c6d5dd76297'
	'e4aef82935c79c3b138cb45384db72fa'
	'c96ddf5e69fe88f9ac9a34c44dd625fe'
	'c47f3eec8c8a0ef2a7fbaaca56678177'
	'0f641e0ea2217d2578dda1e3bd3c8606'
)

_src=${srcdir}/boxee-"$pkgver2"-sources

build() {
        pushd ${_src} || return 1
       
	# copy xbmctex because its missing
	cp -r ${srcdir}/XBMCTex ${_src}/tools/.

        #fribidi.patch fixes the compile issue related to fribidi (big thanks to vrtladept for getting this one rolling)
        patch -p0 < ../fribidi.patch || return 1
        
        #gcc44.patch makes the few necessary changes to compile on gcc 4.4 (thanks to anish for developing this one!)
        patch -p0 < ../gcc44.patch || return 1
        
        #boxee64.patch contains changes required for x86_64 and thus is only loaded if your arch is x86_64
        if [ $(uname -m) = "x86_64" ]; then
        	patch -p0 < ../boxee64.patch || return 1
        fi || return 1

	#config patch
	patch -p0 < ../conf.patch || return 1

        pushd xbmc/lib/libBoxee/tinyxpath || return 1
        autoreconf -vif || return 1
	./configure || return 1
        popd || return 1

        # Goom also needs a fixup due to newer autotools
        pushd xbmc/visualizations/Goom/goom2k4-0 || return 1
        aclocal || return 1
        libtoolize --copy --force || return 1
        ./autogen.sh --enable-static --with-pic || return 1
	popd || return 1

        autoconf || return 1
	./configure --prefix=/opt/boxee --disable-debug || return 1

	#this is a dirty hack to fix an issue with realloc, the fix was realized by [vEX] for xbmc on the archlinux forums
	#unfortunately a patch would be a sketchy fix itself as the file needing to be changed is autogenerated
	sed 's|\#define\ HAVE\_REALLOC\ 0|\#define\ HAVE\_REALLOC\ 1|g' config.h > config.h.sed || return 1
	sed 's|\#define\ realloc\ rpl\_realloc|\/\*\#define\ realloc\ rpl\_realloc\*\/|g' config.h.sed > config.h || return 1
	
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
	for i in *; do
		install -d ${pkgdir}/opt/boxee/language/"$i" || return 1
		if [ $(ls "$i" | wc -l) != "0" ]; then
			install -D "$i"/* ${pkgdir}/opt/boxee/language/"$i"/ || return 1
		fi || return 1
        done || return 1
	popd || return 1
	
	#media
	install -d ${pkgdir}/opt/boxee/media || return 1
	pushd ${_src}/media/ || return 1
	for i in *; do
		if [ -d "$i" ]; then
			install -d ${pkgdir}/opt/boxee/media/"$i" || return 1
			if [ $(ls "$i" | wc -l) != "0" ]; then
				install -D "$i"/* ${pkgdir}/opt/boxee/media/"$i"/ || return 1
			fi || return 1
		else
			if [ $(echo "$i" | grep Splash -c) = "0" -a $(echo "$i" | grep xbmc.icns -c) = "0" -a $(echo "$i" | grep icon.png -c) = "0" ]; then
				install -D "$i" ${pkgdir}/opt/boxee/media/ || return 1
			fi || return 1
		fi || return 1
        done || return 1
	popd || return 1
	
	#scripts
	install -d ${pkgdir}/opt/boxee/scripts || return 1
	##Lyrics
	###resources -> language
	install -d ${pkgdir}/opt/boxee/scripts/Lyrics/resources/language/
	pushd ${_src}/scripts/Lyrics/resources/language/ || return 1
	for i in *; do
		install -d ${pkgdir}/opt/boxee/scripts/Lyrics/resources/language/"$i" || return 1
		if [ $(ls "$i" | wc -l) != "0" ]; then
			install -D "$i"/* ${pkgdir}/opt/boxee/scripts/Lyrics/resources/language/"$i"/ || return 1
		fi || return 1
        done || return 1
	popd || return 1
	###resources -> lib
	install -d ${pkgdir}/opt/boxee/scripts/Lyrics/resources/lib || return 1
	install -D ${_src}/scripts/Lyrics/resources/lib/* ${pkgdir}/opt/boxee/scripts/Lyrics/resources/lib/ || return 1
	###resources -> scrapers
	install -d ${pkgdir}/opt/boxee/scripts/Lyrics/resources/scrapers || return 1
	pushd ${_src}/scripts/Lyrics/resources/scrapers/ || return 1
	for i in *; do
		if [ -d "$i" ]; then 
			install -d ${pkgdir}/opt/boxee/scripts/Lyrics/resources/scrapers/"$i" || return 1
			if [ $(ls "$i" | wc -l) != "0" ]; then			
				install -D "$i"/* ${pkgdir}/opt/boxee/scripts/Lyrics/resources/scrapers/"$i"/ || return 1
			fi || return 1
		else 
			install -D "$i" ${pkgdir}/opt/boxee/scripts/Lyrics/resources/scrapers/ || return 1
		fi || return 1
	done || return 1
	popd || return 1
	###resources -> skins -> Boxee
	install -d ${pkgdir}/opt/boxee/scripts/Lyrics/resources/skins/Boxee
	pushd ${_src}/scripts/Lyrics/resources/skins/Boxee || return 1
	install -d ${pkgdir}/opt/boxee/scripts/Lyrics/resources/skins/Boxee/PAL || return 1
	install -D PAL/* ${pkgdir}/opt/boxee/scripts/Lyrics/resources/skins/Boxee/PAL/ || return 1
	install -d ${pkgdir}/opt/boxee/scripts/Lyrics/resources/skins/Boxee/media || return 1
	install -D media/* ${pkgdir}/opt/boxee/scripts/Lyrics/resources/skins/Boxee/media/ || return 1
	ln -s /opt/boxee/scripts/Lyrics/resources/skins/Boxee/PAL ${pkgdir}/opt/boxee/scripts/Lyrics/resources/skins/Boxee/720p || return 1
	popd || return 1
	###resources -> skins -> Default
	install -d ${pkgdir}/opt/boxee/scripts/Lyrics/resources/skins/Default
	pushd ${_src}/scripts/Lyrics/resources/skins/Default || return 1
	install -d ${pkgdir}/opt/boxee/scripts/Lyrics/resources/skins/Default/PAL || return 1
	install -D PAL/* ${pkgdir}/opt/boxee/scripts/Lyrics/resources/skins/Default/PAL/ || return 1
	install -d ${pkgdir}/opt/boxee/scripts/Lyrics/resources/skins/Default/media || return 1
	install -D media/* ${pkgdir}/opt/boxee/scripts/Lyrics/resources/skins/Default/media/ || return 1
	ln -s /opt/boxee/scripts/Lyrics/resources/skins/Default/PAL ${pkgdir}/opt/boxee/scripts/Lyrics/resources/skins/Default/720p || return 1
	popd || return 1
	##base files
	install -D ${_src}/scripts/Lyrics/resources/__init__.py ${pkgdir}/opt/boxee/scripts/Lyrics/resources/ || return 1
	install -D ${_src}/scripts/Lyrics/default* ${pkgdir}/opt/boxee/scripts/Lyrics/ || return 1
	#RTorrent
	##resources -> language
	install -d ${pkgdir}/opt/boxee/scripts/RTorrent/resources/language/english || return 1
	install -D ${_src}/scripts/RTorrent/resources/language/english/* ${pkgdir}/opt/boxee/scripts/RTorrent/resources/language/english/ || return 1
	install -d ${pkgdir}/opt/boxee/scripts/RTorrent/resources/language/spanish || return 1
	install -D ${_src}/scripts/RTorrent/resources/language/spanish/* ${pkgdir}/opt/boxee/scripts/RTorrent/resources/language/spanish/ || return 1
	##resources -> lib
	install -d ${pkgdir}/opt/boxee/scripts/RTorrent/resources/lib || return 1
	install -D ${_src}/scripts/RTorrent/resources/lib/* ${pkgdir}/opt/boxee/scripts/RTorrent/resources/lib/ || return 1
	##resources -> skins -> Boxee
	install -d ${pkgdir}/opt/boxee/scripts/RTorrent/resources/skins/Boxee/720p || return 1
	install -D ${_src}/scripts/RTorrent/resources/skins/Boxee/720p/* ${pkgdir}/opt/boxee/scripts/RTorrent/resources/skins/Boxee/720p/ || return 1
	install -d ${pkgdir}/opt/boxee/scripts/RTorrent/resources/skins/Boxee/PAL || return 1
	install -d ${pkgdir}/opt/boxee/scripts/RTorrent/resources/skins/Boxee/media || return 1
	install -D ${_src}/scripts/RTorrent/resources/skins/Boxee/media/* ${pkgdir}/opt/boxee/scripts/RTorrent/resources/skins/Boxee/media/ || return 1
	##base files
	install -D ${_src}/scripts/RTorrent/default* ${pkgdir}/opt/boxee/scripts/RTorrent/ || return 1
	#OpenSubtitles
	##language
	install -d ${pkgdir}/opt/boxee/scripts/OpenSubtitles/resources/language/english || return 1
	install -D ${_src}/scripts/OpenSubtitles/resources/language/english/* ${pkgdir}/opt/boxee/scripts/OpenSubtitles/resources/language/english/ || return 1
	##lib
	install -d ${pkgdir}/opt/boxee/scripts/OpenSubtitles/resources/lib || return 1
	install -D ${_src}/scripts/OpenSubtitles/resources/lib/* ${pkgdir}/opt/boxee/scripts/OpenSubtitles/resources/lib/ || return 1
	##skins -> Boxee
	install -d ${pkgdir}/opt/boxee/scripts/OpenSubtitles/resources/skins/Boxee/720p || return 1
	install -D ${_src}/scripts/OpenSubtitles/resources/skins/Boxee/720p/* ${pkgdir}/opt/boxee/scripts/OpenSubtitles/resources/skins/Boxee/720p/ || return 1
	install -d ${pkgdir}/opt/boxee/scripts/OpenSubtitles/resources/skins/Boxee/PAL || return 1
	install -D ${_src}/scripts/OpenSubtitles/resources/skins/Boxee/PAL/* ${pkgdir}/opt/boxee/scripts/OpenSubtitles/resources/skins/Boxee/PAL/ || return 1
	install -d ${pkgdir}/opt/boxee/scripts/OpenSubtitles/resources/skins/Boxee/media/flags || return 1
	install -D ${_src}/scripts/OpenSubtitles/resources/skins/Boxee/media/flags/* ${pkgdir}/opt/boxee/scripts/OpenSubtitles/resources/skins/Boxee/media/flags/ || return 1
	install -D ${_src}/scripts/OpenSubtitles/resources/skins/Boxee/media/*.png ${pkgdir}/opt/boxee/scripts/OpenSubtitles/resources/skins/Boxee/media/ || return 1
	##skins -> Default
	install -d ${pkgdir}/opt/boxee/scripts/OpenSubtitles/resources/skins/Default/720p || return 1
	install -D ${_src}/scripts/OpenSubtitles/resources/skins/Default/720p/* ${pkgdir}/opt/boxee/scripts/OpenSubtitles/resources/skins/Default/720p/ || return 1
	install -d ${pkgdir}/opt/boxee/scripts/OpenSubtitles/resources/skins/Default/PAL || return 1
	install -D ${_src}/scripts/OpenSubtitles/resources/skins/Default/PAL/* ${pkgdir}/opt/boxee/scripts/OpenSubtitles/resources/skins/Default/PAL/ || return 1
	install -d ${pkgdir}/opt/boxee/scripts/OpenSubtitles/resources/skins/Default/media/flags || return 1
	install -D ${_src}/scripts/OpenSubtitles/resources/skins/Default/media/flags/* ${pkgdir}/opt/boxee/scripts/OpenSubtitles/resources/skins/Default/media/flags/ || return 1
	install -D ${_src}/scripts/OpenSubtitles/resources/skins/Default/media/*.png ${pkgdir}/opt/boxee/scripts/OpenSubtitles/resources/skins/Default/media/ || return 1
	##base files
	install -D ${_src}/scripts/OpenSubtitles/default* ${pkgdir}/opt/boxee/scripts/OpenSubtitles/ || return 1

	#skin -> Boxee Skin NG
	install -d ${pkgdir}/opt/boxee/skin/Boxee\ Skin\ NG || return 1
	pushd ${_src}/skin/Boxee\ Skin\ NG || return 1
	for i in *; do
		if [ -d "$i" ]; then
			if [ "$i" = "media" ]; then
				install -d ${pkgdir}/opt/boxee/skin/Boxee\ Skin\ NG/"$i"
				pushd "$i" || return 1
				for j in *; do
					if [ "$j" = "guide" ]; then
						install -d ${pkgdir}/opt/boxee/skin/Boxee\ Skin\ NG/"$i"/"$j" || return 1
						pushd "$j" || return 1
						for k in *; do
							install -d ${pkgdir}/opt/boxee/skin/Boxee\ Skin\ NG/"$i"/"$j"/"$k" || return 1
							if [ $(ls "$k" | wc -l) != "0" ]; then						
								install -D "$k"/* ${pkgdir}/opt/boxee/skin/Boxee\ Skin\ NG/"$i"/"$j"/"$k"/ || return 1
							fi || return 1
						done || return 1
						popd || return 1
					else
						if [ -d "$j" ]; then
							install -d ${pkgdir}/opt/boxee/skin/Boxee\ Skin\ NG/"$i"/"$j" || return 1
							if [ $(ls "$j" | wc -l) != "0" ]; then							
								install -D "$j"/* ${pkgdir}/opt/boxee/skin/Boxee\ Skin\ NG/"$i"/"$j"/ || return 1
							fi || return 1
						else
							install -D "$j" ${pkgdir}/opt/boxee/skin/Boxee\ Skin\ NG/"$i"/ || return 1
						fi || return 1
					fi || return 1
				done || return 1
				popd || return 1
			else
				install -d ${pkgdir}/opt/boxee/skin/Boxee\ Skin\ NG/"$i" || return 1
				if [ $(ls "$i" | wc -l) != "0" ]; then
					install -D "$i"/* ${pkgdir}/opt/boxee/skin/Boxee\ Skin\ NG/"$i"/ || return 1
				fi || return 1
			fi || return 1
		else
			install -D "$i" ${pkgdir}/opt/boxee/skin/Boxee\ Skin\ NG/ || return 1
		fi || return 1
        done || return 1
	popd || return 1

	#system
	install -d ${pkgdir}/opt/boxee/system || return 1
	pushd ${_src}/system/ || return 1
	for i in *; do
		if [ -d "$i" ]; then
			install -d ${pkgdir}/opt/boxee/system/"$i" || return 1
			if [ "$i" = "cdrip" ]; then
				pushd "$i" || return 1
				for j in *; do
					if [ $(echo "$j" | grep linux -c) = "1" ]; then
						install -D "$j" ${pkgdir}/opt/boxee/system/"$i"/ || return 1
					fi || return 1
				done
				popd || return 1				
			fi || return 1
			if [ "$i" = "players" ]; then
				pushd "$i" || return 1
				for j in *; do
					if [ -d "$j" ]; then
						install -d ${pkgdir}/opt/boxee/system/"$i"/"$j" || return 1
						pushd "$j" || return 1
						for k in *; do
							if [ -d "$k" ]; then
								install -d ${pkgdir}/opt/boxee/system/"$i"/"$j"/"$k" || return 1
								pushd "$k" || return 1							
									find . | sed -e 's/\.\///g' | while read file; do
										if [ -d "$file" ]; then
											install -d ${pkgdir}/opt/boxee/system/"$i"/"$j"/"$k"/"$file" || return 1
										else
											install -D "$file" ${pkgdir}/opt/boxee/system/"$i"/"$j"/"$k"/"$file" || return 1
										fi || return 1
									done || return 1
								popd || return 1
							else
								if [ $(echo "$k" | grep linux -c) = "1" -o $(echo "$k" | grep so$ -c) = "1" -a $(echo "$k" | grep osx -c) = "0" ]; then
									install -D "$k" ${pkgdir}/opt/boxee/system/"$i"/"$j"/ || return 1
								fi || return 1
							fi || return 1
						done || return 1
						popd || return 1
					fi || return 1
				done || return 1
				popd || return 1
			fi || return 1			
			if [ "$i" = "python" ]; then
				install -d ${pkgdir}/opt/boxee/system/"$i" || return 1
				install -D "$i"/*linux.so ${pkgdir}/opt/boxee/system/"$i"/ || return 1					
				install -d ${pkgdir}/opt/boxee/system/"$i"/spyce || return 1
				pushd "$i"/spyce || return 1
				for k in *; do
					if [ -d "$k" ]; then
						install -d ${pkgdir}/opt/boxee/system/"$i"/spyce/"$k" || return 1
						if [ $(ls "$k" | wc -l) != "0" ]; then
							install -D "$k"/* ${pkgdir}/opt/boxee/system/"$i"/spyce/"$k"/ || return 1
						fi || return 1
					else
						install -D "$k" ${pkgdir}/opt/boxee/system/"$i"/spyce/ || return 1
					fi || return 1
				done || return 1
				popd || return 1
			fi || return 1
			if [ "$i" = "scrapers" ]; then
				install -d ${pkgdir}/opt/boxee/system/"$i"/music || return 1
				install -D ${_src}/system/"$i"/music/* ${pkgdir}/opt/boxee/system/"$i"/music/ || return 1
				install -d ${pkgdir}/opt/boxee/system/"$i"/video || return 1
				install -D ${_src}/system/"$i"/video/* ${pkgdir}/opt/boxee/system/"$i"/video/ || return 1
			fi || return 1
		else
			if [ $(echo "$i" | grep linux -c) = "1" ]; then
				install -D "$i" ${pkgdir}/opt/boxee/system/ || return 1
			fi || return 1
			if [ $(echo "$i" | grep .xml -c) = "1" -a $(echo "$i" | grep IRSSmap.xml -c) = "0" -a $(echo "$i" | grep upnpserver.xml -c) = "0" ]; then
				install -D "$i" ${pkgdir}/opt/boxee/system/ || return 1
			fi || return 1
			if [ $(echo "$i" | grep .conf -c) = "1" ]; then
				install -D "$i" ${pkgdir}/opt/boxee/system/ || return 1
			fi || return 1
		fi || return 1
	done || return 1
	popd || return 1
	#compiling/installing python libs
	install -d ${pkgdir}/opt/boxee/system/python/lib || return 1
	install -D ${_src}/xbmc/lib/libPython/Python/build/lib.linux-$(uname -m)-2.4/*.so ${pkgdir}/opt/boxee/system/python/lib/ || return 1
	pushd ${_src}/xbmc/lib/libPython/Python/Lib || return 1
#This isn't indented because whitespace is significant to python
python -O >/dev/null << EOF
import compileall
compileall.compile_dir(".", force=1)
EOF
	find . | sed -e 's/\.\///g' | while read file; do
		if [ -d "$file" ]; then
			install -d ${pkgdir}/opt/boxee/system/python/lib/"$file" || return 1
		else
			install -D "$file" ${pkgdir}/opt/boxee/system/python/lib/"$file" || return 1
		fi || return 1
	done || return 1
	popd || return 1

	install -D ${_src}/system/python/local/mc.py ${pkgdir}/opt/boxee/system/python/local/mc.py || return 1

	install -d ${pkgdir}/opt/boxee/system/python/local/simplejson || return 1
	pushd ${_src}/system/python/local/simplejson || return 1
		find . | sed -e 's/\.\///g' | while read file; do
			if [ -d "$file" ]; then
				install -d ${pkgdir}/opt/boxee/system/python/local/simplejson/"$file" || return 1
			elif [ $(echo $file | grep .py$ -c) = "1" ]; then
				install -D "$file" ${pkgdir}/opt/boxee/system/python/local/simplejson/"$file" || return 1
			fi || return 1
		done || return 1
	popd || return 1
	
	pushd ${pkgdir}/opt/boxee/system/python/local/simplejson || return 1
#This isn't indented because whitespace is significant to python
python -O >/dev/null << EOF
import compileall
compileall.compile_dir(".", force=1)
EOF
	popd || return 1
	
	pushd ${pkgdir}/opt/boxee/system/python/lib/simplejson/tests || return 1
#This isn't indented because whitespace is significant to python
python -O >/dev/null << EOF
import compileall
compileall.compile_dir(".", force=1)
EOF
	popd || return 1

	#UserData
	install -d ${pkgdir}/opt/boxee/UserData || return 1
	install -D ${_src}/UserData/sources.xml.in.linux ${pkgdir}/opt/boxee/UserData/ || return 1
	install -D ${_src}/UserData/sources.xml.in.diff.linux ${pkgdir}/opt/boxee/UserData/ || return 1
	ln -s ./UserData ${pkgdir}/opt/boxee/userdata || return 1

	#visualizations
	install -d ${pkgdir}/opt/boxee/visualisations/
	pushd ${_src}/visualisations/ || return 1
	for i in *; do
		if [ -d "$i" ]; then
			install -d ${pkgdir}/opt/boxee/visualisations/"$i" || return 1
			if [ $(ls "$i" | wc -l) != "0" ]; then
				install -D "$i"/* ${pkgdir}/opt/boxee/visualisations/"$i"/ || return 1
			fi || return 1
		else
			if [ $(echo "$i" | grep osx -c) = "0" -a $(echo "$i" | grep win32 -c) = "0" -a $(echo "$i" | grep h$ -c) = "0" ]; then
				install -D "$i" ${pkgdir}/opt/boxee/visualisations/ || return 1
			fi || return 1
		fi || return 1
        done || return 1
	popd || return 1

	#screensavers
	install -d ${pkgdir}/opt/boxee/screensavers || return 1
	install -D ${_src}/screensavers/* ${pkgdir}/opt/boxee/screensavers/ || return 1

	#credits
	install -d ${pkgdir}/opt/boxee/credits || return 1
	install -D ${_src}/credits/*.xpr ${pkgdir}/opt/boxee/credits/ || return 1
	install -D ${_src}/credits/*.mod ${pkgdir}/opt/boxee/credits/ || return 1

	#sounds
	install -d ${pkgdir}/opt/boxee/sounds/Bursting\ Bubbles || return 1
	install -D ${_src}/sounds/Bursting\ Bubbles/* ${pkgdir}/opt/boxee/sounds/Bursting\ Bubbles/ || return 1

	#license
	install -d ${pkgdir}/opt/boxee/license || return 1
	install -D ${_src}/license/* ${pkgdir}/opt/boxee/license/ || return 1
	install -D ${_src}/LICENSE.GPL ${pkgdir}/opt/boxee/license/ || return 1

	#rtorrent
	install -d ${pkgdir}/opt/boxee/bin || return 1
	install -D ${_src}/bin-linux/boxee-rtorrent ${pkgdir}/opt/boxee/bin/ || return 1

	#boxee binary
	install -D ${_src}/Boxee ${pkgdir}/opt/boxee/ || return 1
	strip ${pkgdir}/opt/boxee/Boxee || return 1
	install -D ${_src}/run-boxee-desktop.in ${pkgdir}/opt/boxee/run-boxee-desktop || return 1

	#give_me_my_mouse_back
	install -D ${_src}/give_me_my_mouse_back ${pkgdir}/opt/boxee/ || return 1
	strip ${pkgdir}/opt/boxee/give_me_my_mouse_back || return 1
	
	#xbmc-xrandr
	install -D ${_src}/xbmc-xrandr ${pkgdir}/opt/boxee/ || return 1
	strip ${pkgdir}/opt/boxee/xbmc-xrandr || return 1
	
	#boxee-set-initial-resolution
	install -D ${_src}/boxee-set-initial-resolution ${pkgdir}/opt/boxee/ || return 1
	
	#freedesktop
	install -d ${pkgdir}/usr/share/applications || return 1
	install -D ${srcdir}/boxee.desktop ${pkgdir}/usr/share/applications/ || return 1
	install -d ${pkgdir}/usr/share/pixmaps || return 1
	install -D ${_src}/media/icon.png ${pkgdir}/usr/share/pixmaps/boxee.png || return 1
	
	#flashlib - this compiles flashlib and replaces the binary version
	#credits for both the compile options and the patch are to anish
	if [ $(uname -m) != "x86_64" ]; then
		pushd ${srcdir}/flashlib-shared || return 1
			patch -p0 < ../flashlib.patch || return 1
			gcc -c FlashClient.cpp || return 1
			gcc -c FlashLib.cpp || return 1
			g++ -dynamiclib -flat_namespace -shared -o FlashLib-i486-linux.so FlashLib.o FlashClient.o || return 1
			install -D FlashLib-i486-linux.so ${pkgdir}/opt/boxee/system/players/flashplayer/ || return 1
		popd || return 1
	fi || return 1
}
