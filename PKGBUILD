# Maintainer: Pierre Schmitz <pierre@archlinux.de>

pkgname=apache
pkgver=2.2.9
pkgrel=3
pkgdesc="A high performance Unix-based HTTP server"
arch=('i686' 'x86_64')
options=('!libtool')
url='http://www.apache.org/dist/httpd'
license=('APACHE')
backup=(etc/httpd/conf/httpd.conf
        etc/httpd/conf/extra/httpd-{autoindex,dav,default,info,languages}.conf
        etc/httpd/conf/extra/httpd-{manual,mpm,multilang-errordoc}.conf
        etc/httpd/conf/extra/httpd-{ssl,userdir,vhosts}.conf)
depends=('openssl>=0.9.8b' 'zlib' 'apr-util>=1.3.2-2' 'db>=4.7' 'pcre')
install='httpd.install'
source=("http://www.apache.org/dist/httpd/httpd-${pkgver}.tar.bz2"
        'httpd.logrotate' 'httpd' 'arch.layout')
md5sums=('3afa8137dc1999be695a20b62fdf032b'
         'f4d627c64024c1b7b95efb5ffbaa625e'
         'fb6baeced65b7cf5b80083f278adebba'
         '3d659d41276ba3bfcb20c231eb254e0c')

build() {
	cd ${srcdir}/httpd-${pkgver}

	# set default user
	sed -e 's#User daemon#User http#' \
	    -e 's#Group daemon#Group http#' \
	    -i docs/conf/httpd.conf.in || return 1

	cat ${srcdir}/arch.layout >> config.layout
	./configure --enable-layout=Arch \
		--enable-modules=all \
		--enable-mods-shared=all \
		--enable-so \
		--enable-suexec \
		--with-suexec-caller=http \
		--with-suexec-docroot=/srv/http \
		--with-suexec-logfile=/var/log/httpd/suexec.log \
		--with-suexec-bin=/usr/sbin/suexec \
		--with-suexec-uidmin=99 --with-suexec-gidmin=99 \
		--enable-ldap --enable-authnz-ldap \
		--enable-cache --enable-disk-cache --enable-mem-cache --enable-file-cache \
		--enable-ssl --with-ssl \
		--enable-deflate --enable-cgid \
		--enable-proxy --enable-proxy-connect \
		--enable-proxy-http --enable-proxy-ftp \
		--enable-dbd \
		--with-apr=/usr/bin/apr-1-config \
		--with-apr-util=/usr/bin/apu-1-config \
		--with-pcre=/usr || return 1

	make || return 1

	make DESTDIR=${pkgdir} install || return 1
	install -D -m755 ${srcdir}/httpd ${pkgdir}/etc/rc.d/httpd
	install -D -m644 ${srcdir}/httpd.logrotate ${pkgdir}/etc/logrotate.d/httpd

	# symlinks for /etc/httpd
	ln -fs /var/log/httpd ${pkgdir}/etc/httpd/logs
	ln -fs /var/run/httpd ${pkgdir}/etc/httpd/run
	ln -fs /usr/lib/httpd/modules ${pkgdir}/etc/httpd/modules
	ln -fs /usr/lib/httpd/build ${pkgdir}/etc/httpd/build

	# set sane defaults
	sed -e 's#/usr/lib/httpd/modules/#modules/#' \
	    -e 's|#\(Include conf/extra/httpd-multilang-errordoc.conf\)|\1|' \
	    -e 's|#\(Include conf/extra/httpd-autoindex.conf\)|\1|' \
	    -e 's|#\(Include conf/extra/httpd-languages.conf\)|\1|' \
	    -e 's|#\(Include conf/extra/httpd-userdir.conf\)|\1|' \
	    -e 's|#\(Include conf/extra/httpd-default.conf\)|\1|' \
	    -i ${pkgdir}/etc/httpd/conf/httpd.conf || return 1

	# cleanup
	rm -rf ${pkgdir}/usr/share/httpd/manual
	rm -rf ${pkgdir}/etc/httpd/conf/original
	rm -rf ${pkgdir}/srv/http/*
	rmdir ${pkgdir}/usr/bin
}