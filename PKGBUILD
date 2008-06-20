# Maintainer: Pierre Schmitz <pierre@archlinux.de>

pkgname=apache
pkgver=2.2.9
pkgrel=1
pkgdesc="A high performance Unix-based HTTP server"
arch=('i686' 'x86_64')
options=('!libtool')
url='http://www.apache.org/dist/httpd'
license=('APACHE')
backup=(etc/httpd/conf/httpd.conf
        etc/httpd/conf/extra/httpd-{autoindex,dav,default,info,languages}.conf
        etc/httpd/conf/extra/httpd-{manual,mpm,multilang-errordoc}.conf
        etc/httpd/conf/extra/httpd-{ssl,userdir,vhosts}.conf)
depends=('openssl>=0.9.8b' 'zlib' 'apr-util>=1.3.0' 'db>=4.6' 'pcre')
source=("http://www.apache.org/dist/httpd/httpd-${pkgver}.tar.bz2"
        'httpd.logrotate' 'httpd' 'arch.layout')
md5sums=('3afa8137dc1999be695a20b62fdf032b'
         'a13925eef67108cf22f6bd1b2b8decb1'
         'fb6baeced65b7cf5b80083f278adebba'
         '0816f63f1dc68f39e08d57fedd53b95e')

build() {
	cd ${srcdir}/httpd-${pkgver}
	# fix the suexec user
	sed -i 's|^#define AP_HTTPD_USER.*$|#define AP_HTTPD_USER "nobody"|' \
		support/suexec.h || return 1i
	sed -e 's#User daemon#User nobody#' \
	    -e 's#Group daemon#Group nobody#' \
	    -i docs/conf/httpd.conf.in || return 1

	cat ${srcdir}/arch.layout >> config.layout
	./configure --enable-layout=Arch \
		--enable-modules=all \
		--enable-mods-shared=all \
		--enable-ssl \
		--enable-so \
		--enable-proxy \
		--enable-proxy-connect \
		--enable-proxy-ftp \
		--enable-proxy-http \
		--enable-suexec \
		--enable-dbd \
		--enable-cache \
		--enable-disk-cache \
		--enable-mem-cache \
		--with-apr=/usr \
		--with-apr-util=/usr \
		--with-pcre=/usr || return 1

	make || return 1

	make DESTDIR=${pkgdir} install || return 1
	install -D -m755 ${srcdir}/httpd ${pkgdir}/etc/rc.d/httpd
	install -D -m644 ${srcdir}/httpd.logrotate ${pkgdir}/etc/logrotate.d/httpd
	# remove the manual
	rm -rf ${pkgdir}/usr/share/httpd/manual
}