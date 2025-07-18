dnl
dnl autoconf for DRBD
dnl
dnl License: GNU General Public License Version 2 (GPLv2)

dnl Minimum autoconf version we require
AC_PREREQ([2.69])

dnl Include cpp feature testing macro
AC_CONFIG_MACRO_DIR([m4])

dnl What we are, our version, who to bug in case of problems
AC_INIT(DRBD, 9.31.0, [drbd-dev@lists.linbit.com])

dnl Sanitize $prefix. Autoconf does this by itself, but so late in the
dnl generated configure script that the expansion does not occur until
dnl after our eval magic below.
if test "$prefix" = "NONE"; then
  prefix=$ac_default_prefix
fi
exec_prefix=$prefix

dnl Expand autoconf variables so that we dont end up with '${prefix}'
dnl in #defines
dnl Autoconf deliberately leaves them unexpanded to allow make
dnl exec_prefix=/foo install. DRBD supports only DESTDIR, KDIR and
dnl KVER to be invoked with make.
prefix="`eval echo ${prefix}`"
exec_prefix="`eval echo ${exec_prefix}`"
bindir="`eval echo ${bindir}`"
sbindir="`eval echo ${sbindir}`"
libexecdir="`eval echo ${libexecdir}`"
datarootdir="`eval echo ${datarootdir}`"
datadir="`eval echo ${datadir}`"
sysconfdir="`eval echo ${sysconfdir}`"
sharedstatedir="`eval echo ${sharedstatedir}`"
localstatedir="`eval echo ${localstatedir}`"
libdir="`eval echo ${libdir}`"
includedir="`eval echo ${includedir}`"
oldincludedir="`eval echo ${oldincludedir}`"
infodir="`eval echo ${infodir}`"
mandir="`eval echo ${mandir}`"
docdir="`eval echo ${docdir}`"

dnl "--with-<foo>" options (all except rgm enabled by default, pass --without-<foo> to disable)
WITH_84_SUPPORT=yes
WITH_UDEV=yes
WITH_PACEMAKER=yes
WITH_HEARTBEAT=no
WITH_RGMANAGER=no
WITH_BASHCOMPLETION=yes
WITH_NOARCH_SUBPACKAGES=no
WITH_MANUAL=yes
WITH_DRBDMON=yes
WITH_WINDRBD=no
WITH_PREBUILTMAN=no
WITH_KEYUTILS=yes
AC_ARG_WITH([84support],
	[AS_HELP_STRING([--without-84support],
			[Do not include support for drbd driver/module 8.4])],
	[WITH_84_SUPPORT=$withval])
AC_ARG_WITH([udev],
	[AS_HELP_STRING([--with-udev],
			[Enable udev integration])],
	[WITH_UDEV=$withval])
AC_ARG_ENABLE([udevchecks],
	[AS_HELP_STRING([--disable-udevchecks],
			[Disable host udev version checks])],
	[],
	[enable_udevchecks=yes])
AC_ARG_WITH([pacemaker],
	[AS_HELP_STRING([--with-pacemaker],
			[Enable Pacemaker integration])],
	[WITH_PACEMAKER=$withval])
AC_ARG_WITH([heartbeat],
	[AS_HELP_STRING([--with-heartbeat],
			[Enable Heartbeat v1 haresources integration scripts (deprecated!)])],
	[WITH_HEARTBEAT=$withval])
AC_ARG_WITH([rgmanager],
	[AS_HELP_STRING([--with-rgmanager],
			[Enable Red Hat Cluster Suite integration (deprecated!)])],
	[WITH_RGMANAGER=$withval])
AC_ARG_WITH([bashcompletion],
	[AS_HELP_STRING([--with-bashcompletion],
			[Enable programmable bash completion])],
	[WITH_BASHCOMPLETION=$withval])
AC_ARG_WITH([distro],
	[AS_HELP_STRING([--with-distro],
			[Configure for a specific distribution (deprecated!)])],
	[DISTRO=$withval])
AC_ARG_WITH([initdir],
	[AS_HELP_STRING([--with-initdir],
			[Override directory for init scripts (default is distribution-specific)])],
	[INITDIR=$withval])
AC_ARG_WITH([noarchsubpkg],
	[AS_HELP_STRING([--with-noarchsubpkg],
			[Build subpackages that support it for the "noarch" architecture (makes sense only with --enable-spec, supported by RPM from 4.6.0 forward)])],
	[WITH_NOARCH_SUBPACKAGES=$withval])
AC_ARG_ENABLE([spec],
	[AS_HELP_STRING([--enable-spec],
			[Rather than creating Makefiles, create an RPM spec file only])],
	[SPECMODE=$enableval],
	[SPECMODE=""])
AC_ARG_WITH([manual],
	[AS_HELP_STRING([--without-manual],
			[Do not include manual pages])],
	[WITH_MANUAL=$withval])
AC_ARG_WITH([prebuiltman],
	[AS_HELP_STRING([--with-prebuiltman],
			[Do not try to (re)generate man pages])],
	[WITH_PREBUILTMAN=$withval])
AC_ARG_WITH([keyutils],
	[AS_HELP_STRING([--with-keyutils],
			[Enable keyring integration for DRBD TLS])],
	[WITH_KEYUTILS=$withval])

AC_ARG_WITH([windrbd],
	[AS_HELP_STRING([--with-windrbd],
			[Include windrbd support (requires Cygwin)])],
	[WITH_WINDRBD=$withval])
AC_SUBST(WITH_WINDRBD)

if test x"$WITH_WINDRBD" = x"yes" ; then
   if test x"$WITH_84_SUPPORT" = x"yes" ; then
      AC_MSG_WARN([DRBD 8.4 (and earlier) not supported with WinDRBD, disabling drbd 8.4 userland tools build.])
      WITH_84_SUPPORT=no
   fi
fi

AC_SUBST(WITH_84_SUPPORT)
AC_SUBST(WITH_UDEV)
AC_SUBST(WITH_PACEMAKER)
AC_SUBST(WITH_BASHCOMPLETION)
AC_SUBST(WITH_MANUAL)
AC_SUBST(WITH_PREBUILTMAN)

dnl Check for functions
AC_CHECK_FUNCS([getentropy])
AC_CHECK_FUNCS([gethostbyname_r])

dnl Check for types
AC_CHECK_TYPES([__free_fn_t],[],[],
	[[#define _GNU_SOURCE
	#include <search.h>]]
)

dnl Check for keyutils
if test x"$with_keyutils" != xno; then
	AC_SEARCH_LIBS(
		[request_key],
		[keyutils],
		[WITH_KEYUTILS=yes],
		[
			if test x"$with_keyutils" = xyes; then
				AC_MSG_ERROR([Could not find keyutils])
			else
				AC_MSG_WARN([Could not find keyutils])
				WITH_KEYUTILS=no
			fi
		],
		[]
	)
fi
AC_SUBST(WITH_KEYUTILS)

dnl Do we know where systemd unit files go?
PKG_PROG_PKG_CONFIG
AC_ARG_WITH([systemdunitdir],
	[AS_HELP_STRING([--with-systemdunitdir=DIR],[Directory for systemd service files [Auto]])],
	[WITH_SYSTEMD_UNIT_DIR=$withval])
if test x"$with_systemdunitdir" = x || \
   test x"$with_systemdunitdir" = xyes ; then
	if test x"$PKG_CONFIG" != x; then
		systemdunitdir=$($PKG_CONFIG --variable=systemdsystemunitdir systemd)
	fi
	if test x"$systemdunitdir" = x; then
		AC_MSG_NOTICE([Could not detect systemd unit directory])
	fi
else
	systemdunitdir=$with_systemdunitdir
fi
AC_SUBST(systemdunitdir)
AC_MSG_RESULT([Using systemd unit directory: $systemdunitdir])

AC_ARG_WITH(systemdpresetdir,
	AS_HELP_STRING([--with-systemdpresetdir=DIR],
	[Directory for systemd system preset files [Auto]]),
	systemdpresetdir=$withval,systemdpresetdir=$($PKG_CONFIG --variable=systemdsystempresetdir systemd))
AC_SUBST(systemdpresetdir)

dnl Does systemd support ExecCondition=?
AC_ARG_WITH([systemd_exec_condition],
	[AS_HELP_STRING([--with-systemd-exec-condition],[Enable systemd scripts to use ExecCondition=, introduced with systemd 243 [Auto]])])
case "$with_systemd_exec_condition" in
	"")
		if test x"$PKG_CONFIG" != x && $PKG_CONFIG --atleast-version=243 systemd ; then
			systemd_exec_condition=yes
		else
			systemd_exec_condition=no
		fi ;;
	yes|no)
		systemd_exec_condition=$with_systemd_exec_condition ;;
	*)
		AC_MSG_ERROR([Illegal value -$with_systemd_exec_condition- for option --with-systemd-exec-condition]) ;;
esac
AC_SUBST(systemd_exec_condition)
AC_MSG_RESULT([checking wether systemd supports ExecCondition... $systemd_exec_condition])

AC_ARG_WITH(tmpfilesdir,
	AS_HELP_STRING([--with-tmpfilesdir=DIR],[install configuration files for management of volatile files and directories in DIR [[PREFIX/lib/tmpfiles.d]]]),
		tmpfilesdir=$withval, tmpfilesdir='${prefix}/lib/tmpfiles.d')
AC_SUBST(tmpfilesdir)

# set default early
default_udevdir=/lib/udev
if test x"$with_udev" = x || \
   test x"$with_udev" = xyes ; then
	if test x"$PKG_CONFIG" != x; then
		udevdir=$($PKG_CONFIG --variable=udevdir udev)
	fi
	if test x"$udevdir" = x; then
		AC_MSG_NOTICE([Could not detect udev rules directory, using default])
		udevdir=$default_udevdir
	fi
	AC_MSG_RESULT([Using udev rules directory: $udevdir])
else
	udevdir=$default_udevdir
fi
dnl always replace, even if not used
AC_SUBST(udevdir)
udevrulesdir=$udevdir/rules.d
AC_SUBST(udevrulesdir)

AC_ARG_WITH([initscripttype],
	    [AS_HELP_STRING([--with-initscripttype=INIT_SCRIPT_TYPE],
			    [Type of init script to install (sysv|systemd|both|none). [auto]])
	    ])
case "$with_initscripttype" in
	"")
		if grep -ql systemd /sbin/init ; then
			initscripttype=systemd
		else
			initscripttype=sysv
		fi ;;
	sysv|systemd|both|none)
		initscripttype=$with_initscripttype ;;
	*)
		AC_MSG_ERROR([Illegal value -$with_initscripttype- for option --with-initscripttype]) ;;
esac
AC_SUBST(initscripttype)

AC_ARG_WITH([drbdmon],
	[AS_HELP_STRING([--without-drbdmon],
			[Do not include the DRBD resource reporting utility])],
	[WITH_DRBDMON=$withval])
AC_SUBST(WITH_DRBDMON)

dnl Checks for programs
AC_PROG_CC
AC_PROG_LN_S
AC_PATH_PROG(SED, sed)
AC_PATH_PROG(GREP, grep)
AC_PATH_PROG(FLEX, flex)
AC_PATH_PROG(RPMBUILD, rpmbuild)
AC_PATH_PROG(XSLTPROC, xsltproc)
AC_PATH_PROG(CLITEST, clitest)
AC_PATH_PROG(TAR, tar)
AC_PATH_PROG(GIT, git)
AC_PATH_PROG(PO4A_TRANSLATE, po4a-translate)
AC_PATH_PROG(PO4A_GETTEXTIZE, po4a-gettextize)
AC_PATH_PROG(DPKG_BUILDPACKAGE, dpkg-buildpackage)
AC_PATH_PROG(UDEVADM, udevadm, [false], [/sbin$PATH_SEPARATOR$PATH])
AC_PATH_PROG(UDEVINFO, udevinfo, [false], [/sbin$PATH_SEPARATOR$PATH])

if test -z "$CC"; then
   AC_MSG_ERROR([Cannot build utils without a C compiler.])
fi

if test -z $FLEX; then
   AC_MSG_ERROR([Cannot build utils without flex.])
fi

if test x"$WITH_DRBDMON" = x"yes" ; then
   AC_PROG_CXX
   AX_CXX_COMPILE_STDCXX_11(, optional)
   if test "$HAVE_CXX11" = "0"; then
      AC_MSG_WARN([No C++11 compiler found, disabling drbdmon build.])
      WITH_DRBDMON=no
   fi
   AC_CHECK_LIB([rt], [clock_gettime, timer_create, timer_settime, timer_delete], [],
                [AC_MSG_WARN([Could not find required librt functions, disabling drbdmon build.])
                 WITH_DRBDMON=no])
   AC_CHECK_LIB([pthread], [pthread_create], [],
                [AC_MSG_WARN([Could not find required libpthread functions, disabling drbdmon build.])
                 WITH_DRBDMON=no])
fi

if test x"$WITH_WINDRBD" = x"yes" ; then
   AC_COMPILE_IFELSE([AC_LANG_SOURCE([[
#ifndef __CYGWIN__
# error Not a cygwin compiler
#endif
]])], [AC_MSG_RESULT([This is a cygwin compiler, which is good])], [AC_MSG_ERROR([--with-windrbd requires a Cygwin gcc for being built])])
fi

if test -z $RPMBUILD; then
   AC_MSG_WARN([No rpmbuild found, building RPM packages is disabled.])
fi

if test -z $DPKG_BUILDPACKAGE; then
   AC_MSG_WARN([No dpkg-buildpackage found, building Debian packages is disabled.])
fi

if test -z $XSLTPROC; then
   AC_MSG_WARN([Cannot build man pages without xsltproc. You may safely ignore this warning when building from a tarball.])
   dnl default to some sane value at least,
   dnl so the error message about command not found makes sense
   dnl otherwise you get "--xinclude ... command not found" :-/
   XSLTPROC=xsltproc
fi

if test -z $CLITEST; then
   AC_MSG_WARN([Cannot run tests without clitest, disabling test target.])
   WITH_CLITEST=no
else
   WITH_CLITEST=yes
fi

if test -z $PO4A_TRANSLATE; then
   AC_MSG_WARN([Cannot build man pages without po4a-translate. You may safely ignore this warning when building from a tarball.])
   dnl default to some sane value at least,
   dnl so the error message about command not found makes sense
   PO4A_TRANSLATE=po4a-translate
fi

if test -z $PO4A_GETTEXTIZE; then
   AC_MSG_WARN([Cannot build man pages without po4a-gettextize. You may safely ignore this warning when building from a tarball.])
   dnl default to some sane value at least,
   dnl so the error message about command not found makes sense
   PO4A_GETTEXTIZE=po4a-gettextize
fi

if test -z $GIT; then
   AC_MSG_WARN(Cannot update buildtag without git. You may safely ignore this warning when building from a tarball.)
fi

if test "x$enable_udevchecks" != "xno" && test $UDEVADM = false && test $UDEVINFO = false; then
   if test "$WITH_UDEV" = "yes"; then
     AC_MSG_WARN([udev support enabled, but neither udevadm nor udevinfo found on this system.])
   fi
fi

dnl special header checks

if test "$WITH_WINDRBD" = "no"; then
AC_LANG(C)
AC_CHECK_HEADERS([linux/genetlink.h],
   [
      AC_COMPILE_IFELSE([AC_LANG_SOURCE[
         #include <linux/genetlink.h>
         static unsigned int i = CTRL_ATTR_VERSION;
         ]], [AC_DEFINE([HAVE_CTRL_ATTR_VERSION],[],[CTRL_ATTR_VERSION defined in linux/genetlink.h])]),
      AC_COMPILE_IFELSE([AC_LANG_SOURCE[
         #include <linux/genetlink.h>
         static unsigned int i = CTRL_ATTR_HDRSIZE;
         ]], [AC_DEFINE([HAVE_CTRL_ATTR_HDRSIZE],[],[CTRL_ATTR_HDRSIZE defined in linux/genetlink.h])]),
      AC_COMPILE_IFELSE([AC_LANG_SOURCE[
         #include <linux/genetlink.h>
         static unsigned int i = CTRL_ATTR_MCAST_GROUPS;
         ]], [AC_DEFINE([HAVE_CTRL_ATTR_MCAST_GROUPS],[],[CTRL_ATTR_MCAST_GROUPS defined in linux/genetlink.h])]),
      AC_COMPILE_IFELSE([AC_LANG_SOURCE[
         #include <linux/genetlink.h>
         static unsigned int i = CTRL_CMD_DELMCAST_GRP;
         ]], [AC_DEFINE([HAVE_CTRL_CMD_DELMCAST_GRP],[1],[Does genetlink provide CTRL_CMD_DELMCAST_GRP already])]),
   ],
   [AC_MSG_ERROR([Could not find linux/genetlink.h])])
fi

dnl deprecated options
if test -n "$DISTRO"; then
   AC_MSG_WARN([--with-distro is deprecated and ignored and will be removed in the future.])
fi

if test "$WITH_RGMANAGER" = "yes"; then
   AC_MSG_WARN([--with-rgmanager is deprecated and ignored and will be removed in the future.])
fi

if test "$WITH_HEARTBEAT" = "yes"; then
   AC_MSG_WARN([--with-heartbeat is deprecated and ignored and will be removed in the future.])
fi

test -z $INITDIR && INITDIR="$sysconfdir/init.d"

AC_SUBST(INITDIR)
AC_SUBST(WITH_CLITEST)

AH_TEMPLATE(DRBD_LIB_DIR, [Local state directory. Commonly
			  /var/lib/drbd or /usr/local/var/lib/drbd])
AH_TEMPLATE(DRBD_RUN_DIR, [Runtime state directory. Commonly
			  /var/run/drbd or /usr/local/var/run/drbd])
AH_TEMPLATE(DRBD_LOCK_DIR, [Local lock directory. Typically DRBD_RUN_DIR/lock])
AH_TEMPLATE(DRBD_CONFIG_DIR, [Local configuration directory. Commonly
			   /etc or
			   /usr/local/etc])
AH_TEMPLATE(DRBD_BIN_DIR, [Installation directory. Commonly
			   /usr/bin])
AH_TEMPLATE(DRBD_LEGACY_84, [Include support for drbd-8.4 kernel code])
AH_TEMPLATE(WINDRBD, [Include support for windrbd (do not use this under Linux)])

AC_SUBST(DRBD_LIB_DIR, [$localstatedir/lib/$PACKAGE_TARNAME])
AC_SUBST(DRBD_RUN_DIR, [$localstatedir/run/$PACKAGE_TARNAME])
AC_SUBST(DRBD_LOCK_DIR, [$DRBD_RUN_DIR/lock])
AC_SUBST(DRBD_CONFIG_DIR, [$sysconfdir])
AC_SUBST(DRBD_BIN_DIR, [$exec_prefix/bin])

AC_DEFINE_UNQUOTED(DRBD_LIB_DIR, ["$DRBD_LIB_DIR"])
AC_DEFINE_UNQUOTED(DRBD_RUN_DIR, ["$DRBD_RUN_DIR"])
AC_DEFINE_UNQUOTED(DRBD_LOCK_DIR, ["$DRBD_LOCK_DIR"])
AC_DEFINE_UNQUOTED(DRBD_CONFIG_DIR, ["$DRBD_CONFIG_DIR"])
AC_DEFINE_UNQUOTED(DRBD_BIN_DIR, ["$DRBD_BIN_DIR"])

if test "$WITH_84_SUPPORT" = "yes"; then
   AC_DEFINE(DRBD_LEGACY_84, [1])
fi

if test "$WITH_WINDRBD" = "yes"; then
   AC_DEFINE(WINDRBD, [1])
fi

dnl The configuration files we create (from their .in template)
if test -z $SPECMODE; then
   AC_CONFIG_FILES([
	Makefile
	scripts/Makefile
	user/drbdmon/Makefile
	user/shared/Makefile
	user/v84/Makefile
	user/v9/Makefile
	user/windrbd/Makefile
	documentation/common/Makefile_v84_com
	documentation/common/Makefile_v9_com
   ])
   AC_CONFIG_HEADERS(user/shared/config.h)
else
   AC_CONFIG_FILES(drbd.spec)
fi

dnl output
AC_OUTPUT
