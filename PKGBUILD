# $Id: PKGBUILD 186139 2013-05-21 09:11:14Z tpowa $
# Contributor: Tobias Powalowski <tpowa@archlinux.org>
# Contributor: judd <jvinet@zeroflux.org>
# Contributor: Siosm <tim@siosm.fr>
# Maintainer: Nicky726 <nicky726@gmail.com>

pkgname=selinux-pam
_origname=pam
pkgver=1.1.6
pkgrel=4
pkgdesc="SELinux aware PAM (Pluggable Authentication Modules) library"
arch=('i686' 'x86_64')
license=('GPL2')
url="http://www.kernel.org/pub/linux/libs/pam/"
depends=('glibc' 'db' 'cracklib' 'libtirpc' 'selinux-pambase' 'selinux-usr-libselinux')
makedepends=('flex' 'w3m' 'docbook-xml>=4.4' 'docbook-xsl')
conflicts=("${_origname}")
provides=("${_origname}=${pkgver}-${pkgrel}")
backup=(etc/security/{access.conf,group.conf,limits.conf,namespace.conf,namespace.init,pam_env.conf,time.conf} etc/default/passwd etc/environment)
groups=('selinux' 'selinux-system-utilities')
source=(https://fedorahosted.org/releases/l/i/linux-pam/Linux-PAM-$pkgver.tar.bz2
        #http://www.kernel.org/pub/linux/libs/pam/library/Linux-PAM-$pkgver.tar.bz2
        ftp://ftp.archlinux.org/other/pam_unix2/pam_unix2-2.9.1.tar.bz2
        pam_include_sys_resource.patch
        pam_add_destdir.patch
        pamtmp.conf
        pam_unix2-glibc216.patch
        pam_unix2-rm_selinux_check_access.patch)
options=('!libtool' '!emptydirs')


md5sums=('7b73e58b7ce79ffa321d408de06db2c4'
         'da6a46e5f8cd3eaa7cbc4fc3a7e2b555'
         '58e378bb0bb2411be1bb6fc44d447ecf'
         '999d0f85d17e8bca8d8dfb106eb16be5'
         '8083703ccc050c8aee5cb921491caff7'
         'dac109f68e04a4df37575fda6001ea17'
         '6a0a6bb6f6f249ef14f6b21ab9880916')


build() {
  cd $srcdir/Linux-PAM-$pkgver
  ./configure --libdir=/usr/lib --sbindir=/usr/bin\
    --enable-selinux
  patch -Np2 -i ../pam_add_destdir.patch
  patch -Np1 -i ../pam_include_sys_resource.patch
  make

  cd $srcdir/pam_unix2-2.9.1
  patch -Np1 -i ../pam_unix2-glibc216.patch
  patch -Np1 -i ../pam_unix2-rm_selinux_check_access.patch
  ./configure --libdir=/usr/lib --sbindir=/usr/bin\
		--enable-selinux
  make
}

package() {
  cd $srcdir/Linux-PAM-$pkgver
  make DESTDIR=$pkgdir SCONFIGDIR=/etc/security install

  cd $srcdir
  install -m 644 -D pamtmp.conf $pkgdir/etc/tmpfiles.d/pamtmp.conf

  # build pam_unix2 module
  # source ftp://ftp.suse.com/pub/people/kukuk/pam/pam_unix2
  cd $srcdir/pam_unix2-2.9.1
  make DESTDIR=$pkgdir install

  # add the realtime permissions for audio users
  sed -i 's|# End of file||' $pkgdir/etc/security/limits.conf
  cat >>$pkgdir/etc/security/limits.conf <<_EOT
*               -       rtprio          0
*               -       nice            0
@audio          -       rtprio          65
@audio          -       nice           -10
@audio          -       memlock         40000
_EOT

  # fix some missing symlinks from old pam for compatibility
  cd $pkgdir/usr/lib/security
  ln -s pam_unix.so pam_unix_acct.so
  ln -s pam_unix.so pam_unix_auth.so
  ln -s pam_unix.so pam_unix_passwd.so
  ln -s pam_unix.so pam_unix_session.so

  # set unix_chkpwd uid
  chmod +s $pkgdir/usr/bin/unix_chkpwd
}
