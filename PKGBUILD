pkgname=('linux-mainline' 'linux-mainline-headers')
pkgbase=linux-mainline
pkgver=0
pkgrel=1
pkgdesc='Linux Mainline'
arch=('x86_64')
url='https://kernel.org/'
license=('GPL2')
makedepends=(bc kmod libelf pahole cpio perl tar xz git)
options=('!strip')

pkgver() {
	cd "$pkgbase"
	git describe --tags | sed 's/v//;s/-rc/rc/'
}

# prepare() {
# 	cd "$pkgbase"
# 	echo '' > .scmversion
# 	cp ../defconfig out/.config
# 	make O=out olddefconfig
# }

# build() {
# 	cd "$pkgbase"
# 	make O=out
# }

package_linux-mainline() {
	# options and directives overrides
	pkgdesc="The $pkgdesc kernel and modules"
	depends=(coreutils kmod initramfs)

	cd "$pkgbase"
	local MODDIR="$pkgdir/usr/lib/modules/$(make O=out -s kernelversion)"

	echo "Installing boot image..."
	install -Dm644 "out/$(make O=out -s image_name)" "$MODDIR/vmlinuz"

  	# Used by mkinitcpio to name the kernel
  	echo "$pkgbase" | install -Dm644 /dev/stdin "$MODDIR/pkgbase"

	echo "Installing modules..."
	make O=out INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 modules_install

	echo "Removing links..."
	rm -f "$MODDIR/source" "$MODDIR/build"
}

package_linux-mainline-headers() {
	# options and directives overrides
	pkgdesc="Headers and scripts for building modules for the $pkgdesc kernel"
	depends=(pahole)

	cd "$pkgbase"
	local BLDDIR="$pkgdir/usr/lib/modules/$(make O=out -s kernelversion)/build"

	echo "Installing build files..."
	install -Dt "$BLDDIR" -m644 out/.config Makefile out/Module.symvers out/System.map out/vmlinux
	install -Dt "$BLDDIR/kernel" -m644 kernel/Makefile
  	install -Dt "$BLDDIR/arch/x86" -m644 arch/x86/Makefile
	cp -t "$BLDDIR" -a scripts; cp -t "$BLDDIR" -a out/scripts
	install -Dt "$BLDDIR/tools/objtool" out/tools/objtool/objtool
	mkdir -p "$BLDDIR/fs/xfs" "$BLDDIR/mm"

	echo "Installing headers..."
	cp -t "$BLDDIR" -a include; cp -t "$BLDDIR" -a out/include
	cp -t "$BLDDIR/arch/x86" -a arch/x86/include; cp -t "$BLDDIR/arch/x86" -a out/arch/x86/include
	install -Dt "$BLDDIR/arch/x86/kernel" -m644 out/arch/x86/kernel/asm-offsets.s
	install -Dt "$BLDDIR/drivers/md" -m644 drivers/md/*.h
	install -Dt "$BLDDIR/net/mac80211" -m644 net/mac80211/*.h
	install -Dt "$BLDDIR/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h
	install -Dt "$BLDDIR/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
	install -Dt "$BLDDIR/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
	install -Dt "$BLDDIR/drivers/media/tuners" -m644 drivers/media/tuners/*.h

	echo "Installing Kconfig files..."
	find -name 'Kconfig*' -exec install -Dm644 {} "$BLDDIR/{}" \;

	echo "Removing unneeded architectures..."
	local arch
	for arch in "$BLDDIR"/arch/*/; do
		[[ $arch = */x86/ ]] && continue
		echo "Removing $(basename "$arch")"
		rm -r "$arch"
	done

	echo "Removing documentation..."
	rm -r "$BLDDIR/Documentation"

	echo "Removing broken symlinks..."
	find -L "$BLDDIR" -type l -printf 'Removing %P\n' -delete

	echo "Removing loose objects..."
	find "$BLDDIR" -type f -name '*.o' -printf 'Removing %P\n' -delete

	echo "Stripping build tools..."
	local file
	while read -rd '' file; do
		case "$(file -bi "$file")" in
			application/x-sharedlib\;*)      # Libraries (.so)
			strip -v $STRIP_SHARED "$file" ;;
			application/x-archive\;*)        # Libraries (.a)
			strip -v $STRIP_STATIC "$file" ;;
			application/x-executable\;*)     # Binaries
			strip -v $STRIP_BINARIES "$file" ;;
			application/x-pie-executable\;*) # Relocatable binaries
			strip -v $STRIP_SHARED "$file" ;;
		esac
	done < <(find "$BLDDIR" -type f -perm -u+x ! -name vmlinux -print0)

	echo "Stripping vmlinux..."
  	strip -v $STRIP_STATIC "$BLDDIR/vmlinux"

	echo "Adding symlink..."
  	mkdir -p "$pkgdir/usr/src"
  	ln -sr "$BLDDIR" "$pkgdir/usr/src/$pkgbase"
}
