# Maintainer: Jan de Groot <jgc@archlinux.org>
# Contributor: Andrea Scarpino <andrea@archlinux.org>
# Contributor: Pierre Schmitz <pierre@archlinux.de>

pkgname=apache
pkgver=2.4.26
pkgrel=2
pkgdesc='A high performance Unix-based HTTP server'
arch=('i686' 'x86_64')
url='http://www.apache.org/dist/httpd'
license=('APACHE')
backup=(
    etc/httpd/conf/httpd.conf
    etc/httpd/conf/extra/httpd-{autoindex,dav,default,info,languages}.conf
    etc/httpd/conf/extra/httpd-{manual,mpm,multilang-errordoc}.conf
    etc/httpd/conf/extra/httpd-{ssl,userdir,vhosts}.conf
    etc/httpd/conf/extra/proxy-html.conf
    etc/httpd/conf/{mime.types,magic}
    etc/logrotate.d/httpd
)
depends=('zlib' 'apr-util' 'pcre' 'libnghttp2' 'openssl')
makedepends=('libxml2' 'lua')
optdepends=(
    'lua: for mod_lua module'
    'libxml2: for mod_proxy_html, mod_xml2enc modules'
    'lynx: apachectl status'
)
source=(
    https://www.apache.org/dist/httpd/httpd-${pkgver}.tar.bz2{,.asc}
    openssl-malloc-init.patch
    apache.tmpfiles.conf
    httpd.logrotate
    httpd.service
    arch.layout
    nohttp2forprefork.patch # https://github.com/apache/httpd/commit/ab03196a879f4064f618a9a45d63c67a67f4b901
)
sha256sums=('a07eb52fafc879e0149d31882f7da63173e72df4478db4dc69f7a775b663d387'
            'SKIP'
            'd305f8b52ac2a9bbda7bb0776496471e69e9d30642740f594d00086a8c7e344c'
            '63da1a420f4714a3e7af2672d28384419cc7eedbe7bf35baebd02938fabc15bf'
            '0bbbfae23a917b2fce0bf8f900f60319b50769224a96314e9301a75ccd078e16'
            'f574bac6d5f398e7a649fc0e1ca66ff01ad4ef34dac71258e93d8a9d9a2b3960'
            'dda05c6e76f12624e418ca18a36f2e90ec1c5b1cc52fed7142fce6076ec413f3'
            '25d297199c536ccfdb2c4d92988a74113ac2e14261090b5c1d3ac01537c5c742')
validpgpkeys=('A93D62ECC3C8EA12DB220EC934EA76E6791485A8') # Jim Jagielski

prepare() {
  cd httpd-${pkgver}

  # set default user
  sed -e 's#User daemon#User http#' \
      -e 's#Group daemon#Group http#' \
      -i docs/conf/httpd.conf.in

  cat "${srcdir}/arch.layout" >> config.layout

  # https://github.com/openssl/openssl/issues/2865
  patch -Np1 -i ../openssl-malloc-init.patch

  # https://bugs.archlinux.org/task/54666
  patch -Np1 -i ../nohttp2forprefork.patch
}

build() {
  cd httpd-${pkgver}

  ./configure --sbindir=/usr/bin \
      --enable-layout=Arch \
      --enable-mpms-shared=all \
      --enable-modules=all \
      --enable-mods-shared=all \
      --enable-so \
      --enable-suexec \
      --with-suexec-caller=http \
      --with-suexec-docroot=/srv/http \
      --with-suexec-logfile=/var/log/httpd/suexec.log \
      --with-suexec-bin=/usr/bin/suexec \
      --with-suexec-uidmin=99 --with-suexec-gidmin=99 \
      --enable-ldap --enable-authnz-ldap --enable-authnz-fcgi \
      --enable-cache --enable-disk-cache --enable-mem-cache --enable-file-cache \
      --enable-ssl --with-ssl \
      --enable-deflate --enable-cgi --enable-cgid \
      --enable-proxy --enable-proxy-connect \
      --enable-proxy-http --enable-proxy-ftp \
      --enable-dbd --enable-imagemap --enable-ident --enable-cern-meta \
      --enable-lua --enable-xml2enc --enable-http2 \
      --with-apr=/usr/bin/apr-1-config \
      --with-apr-util=/usr/bin/apu-1-config \
      --with-pcre=/usr

  make
}

package() {
  cd httpd-${pkgver}

  make DESTDIR="${pkgdir}" install
   
  install -D -m644 "${srcdir}/httpd.logrotate" "${pkgdir}/etc/logrotate.d/httpd"
  install -D -m644 "${srcdir}/apache.tmpfiles.conf" "${pkgdir}/usr/lib/tmpfiles.d/apache.conf"
  install -D -m644 "${srcdir}/httpd.service" "${pkgdir}/usr/lib/systemd/system/httpd.service"	
  install -D -m644 LICENSE "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"

  # symlinks for /etc/httpd
  # do we really need this symlink?
  ln -fs /usr/lib/httpd/modules "${pkgdir}/etc/httpd/modules"

  # set sane defaults
  sed -e 's#/usr/lib/httpd/modules/#modules/#' \
      -e 's|#\(LoadModule negotiation_module \)|\1|' \
      -e 's|#\(LoadModule include_module \)|\1|' \
      -e 's|#\(LoadModule userdir_module \)|\1|' \
      -e 's|#\(LoadModule slotmem_shm_module \)|\1|' \
      -e 's|#\(Include conf/extra/httpd-multilang-errordoc.conf\)|\1|' \
      -e 's|#\(Include conf/extra/httpd-autoindex.conf\)|\1|' \
      -e 's|#\(Include conf/extra/httpd-languages.conf\)|\1|' \
      -e 's|#\(Include conf/extra/httpd-userdir.conf\)|\1|' \
      -e 's|#\(Include conf/extra/httpd-default.conf\)|\1|' \
      -e 's|#\(Include conf/extra/httpd-mpm.conf\)|\1|' \
      -i "${pkgdir}/etc/httpd/conf/httpd.conf"

  # cleanup
  rm -r "${pkgdir}/usr/share/httpd/manual"
  rm -r "${pkgdir}/etc/httpd/conf/original"
  rm -r "${pkgdir}/srv/"
  rm -r "${pkgdir}/run"
}
