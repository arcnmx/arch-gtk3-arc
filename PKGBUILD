# Maintainer: twilinx <twilinx@mesecons.net>

pkgname=gtk3-arc
pkgver=3.22.24
pkgrel=1
conflicts=(gtk3)
provides=("gtk3=$pkgver")
pkgdesc="GTK+ 3 with typeahead feature enabled for the file chooser widget"
arch=(i686 x86_64)
url="http://www.gtk.org/"
install=gtk3.install
depends=(atk cairo libxcursor libxinerama libxrandr libxi libepoxy gdk-pixbuf2 dconf
         libxcomposite libxdamage pango shared-mime-info at-spi2-atk wayland libxkbcommon
         adwaita-icon-theme json-glib librsvg wayland-protocols desktop-file-utils mesa
         cantarell-fonts gtk-update-icon-cache)
optdepends=('libcanberra: gtk3-widget-factory demo'
            'gtk3-print-backends: Printing')
makedepends=(gobject-introspection libcanberra gtk-doc git colord rest libcups glib2-docs
             sassc)
license=(LGPL)
_commit=e72d54c8a7bdf5f41feccbcc0b78522a8b50d79e  # tags/3.22.24^0
source=("git://git.gnome.org/gtk+#commit=$_commit"
        settings.ini
        gtk-query-immodules-3.0.hook
        typeahead.patch
        paste-selection.patch
        paste-selection-fix.patch
        shift-insert.patch
        kde-server-decoration.patch)
sha256sums=('SKIP'
            '01fc1d81dc82c4a052ac6e25bf9a04e7647267cc3017bc91f9ce3e63e5eb9202'
            'de46e5514ff39a7a65e01e485e874775ab1c0ad20b8e94ada43f4a6af1370845'
            '5006fa1dcea9aa74766196ec5c18e5172d7287195c2a49ffcd0adc13bc6e62c1'
            '4273c017f3dd061dccf8c4b1da8a5e13c697a10f9c143cd3bcac5a1c2c99f51a'
            '495a8e588e3a454fc94e417e0be91ffc5176389b4e590e94c6017d9e12de5297'
            'e5ae764587512a39ac3c84f01205ae458ab94cf22468beafcab841578ad38744'
            '58642d55853cbe112942aa6bf90527a83aab33965bf59ac67b25b9bfca5e7f60')

prepare() {
    cd "$srcdir/gtk+"

    # Typeahead-specific changes
    patch gtk/gtkfilechooserwidget.c -i "$srcdir/typeahead.patch"

    # Shift-Insert selection paste
    patch -p0 < "$srcdir/paste-selection.patch"
    patch -p0 < "$srcdir/paste-selection-fix.patch"
    patch -p0 < "$srcdir/shift-insert.patch"

	# KDE wayland server decoration protocol
    patch -p1 < "$srcdir/kde-server-decoration.patch"

    NOCONFIGURE=1 ./autogen.sh
}

build() {
    cd "$srcdir/gtk+"

    # Fix for gdbus-codegen
    export PYTHONPATH="/usr/share/glib-2.0"

    CXX=/bin/false ./configure --prefix=/usr \
        --sysconfdir=/etc \
        --localstatedir=/var \
        --disable-schemas-compile \
        --enable-x11-backend \
        --enable-broadway-backend \
        --enable-wayland-backend \
        --enable-gtk-doc=no \
        --enable-gtk-doc-html=no

    #https://bugzilla.gnome.org/show_bug.cgi?id=655517
    sed -i -e 's/ -shared / -Wl,-O1,--as-needed\0/g' libtool

    make
}

package() {
    install=gtk3.install

    cd "$srcdir/gtk+"
    make DESTDIR="$pkgdir" install
    install -Dm644 ../settings.ini "$pkgdir/usr/share/gtk-3.0/settings.ini"
    install -Dm644 ../gtk-query-immodules-3.0.hook "$pkgdir/usr/share/libalpm/hooks/gtk-query-immodules-3.0.hook"

    # gtk-update-icon-cache will be provided in a separate package
    rm $pkgdir/usr/bin/gtk-update-icon-cache

    # remove files that are already provided by gtk3-print-backends
    cd "$pkgdir"
    for _f in usr/lib/*/*/printbackends/*; do
        case $_f in
            *-file.so|*-lpr.so) continue;;
        esac

        rm "$_f"
    done
}
