m4_define([rsvg_major_version],[2])
m4_define([rsvg_minor_version],[40])
m4_define([rsvg_micro_version],[21])
m4_define([rsvg_extra_version],[])
m4_define([rsvg_version],[rsvg_major_version.rsvg_minor_version.rsvg_micro_version()rsvg_extra_version])
m4_define([rsvg_lt_version_info],m4_eval(rsvg_major_version + rsvg_minor_version):rsvg_micro_version:rsvg_minor_version)

AC_INIT([RSVG],[rsvg_version],[https://github.com/oaken-source/librsvg-og/issues],[librsvg])

AC_CONFIG_HEADERS([config.h])
AC_CONFIG_SRCDIR([rsvg.h])
AC_CONFIG_AUX_DIR([build-aux])
AC_CONFIG_MACRO_DIR([m4])

AM_INIT_AUTOMAKE([-Wall gnu])

dnl ===========================================================================
dnl API versioning
dnl ===========================================================================

RSVG_API_VERSION=2.0
RSVG_API_MAJOR_VERSION=2
RSVG_API_MINOR_VERSION=0

AC_SUBST([RSVG_API_VERSION])
AC_SUBST([RSVG_API_MAJOR_VERSION])
AC_SUBST([RSVG_API_MINOR_VERSION])
AC_SUBST([RSVG_API_VERSION_U],[AS_TR_SH([$RSVG_API_VERSION])])

dnl ===========================================================================

GLIB_REQUIRED=2.12.0
GIO_REQUIRED=2.24.0
LIBXML_REQUIRED=2.9.0
CAIRO_REQUIRED=1.2.0
PANGO_REQUIRED=1.38.0
GDK_PIXBUF_REQUIRED=2.20
GTK3_REQUIRED=3.10.0
CROCO_REQUIRED=0.6.1

# ===========================================================================

AC_SUBST([RSVG_LT_VERSION_INFO],[rsvg_lt_version_info])
AC_SUBST([LIBRSVG_MAJOR_VERSION],[rsvg_major_version])
AC_SUBST([LIBRSVG_MINOR_VERSION],[rsvg_minor_version])
AC_SUBST([LIBRSVG_MICRO_VERSION],[rsvg_micro_version])

dnl ===========================================================================

AC_PROG_LN_S
AC_ISC_POSIX
AC_PROG_CC
AC_PROG_CPP
AC_PROG_AWK
AM_PROG_AR
AC_STDC_HEADERS
AM_PROG_CC_C_O

AC_C_BIGENDIAN

PKG_PROG_PKG_CONFIG

LT_INIT([win32-dll])
LT_LIB_M

dnl ===========================================================================

#AC_CANONICAL_HOST

AC_MSG_CHECKING([for some Win32 platform])
case "$host" in
  *-*-mingw*|*-*-cygwin*)
    platform_win32=yes
    ;;
  *)
    platform_win32=no
    ;;
esac
AC_MSG_RESULT([$platform_win32])
AM_CONDITIONAL(PLATFORM_WIN32, test "$platform_win32" = "yes")

AC_MSG_CHECKING([for native Win32])
case "$host" in
  *-*-mingw*)
    native_win32=yes
    ;;
  *)
    native_win32=no
    ;;
esac
AC_MSG_RESULT([$native_win32])
AM_CONDITIONAL(OS_WIN32, test "$native_win32" = "yes")

dnl ===========================================================================

AC_MSG_CHECKING([for glib-mkenums script])
GLIB_MKENUMS=`$PKG_CONFIG --variable=glib_mkenums glib-2.0`
AS_IF([test "x$GLIB_MKENUMS" = "x"],
	[AC_MSG_ERROR([glib-mkenums not listed in glib-2.0 pkg-config file])],
	[AC_SUBST([GLIB_MKENUMS])
	AC_MSG_RESULT([$GLIB_MKENUMS])])

PKG_CHECK_MODULES(LIBRSVG, \
	gdk-pixbuf-2.0 >= $GDK_PIXBUF_REQUIRED \
	glib-2.0 >= $GLIB_REQUIRED \
	gio-2.0 >= $GIO_REQUIRED \
	libxml-2.0 >= $LIBXML_REQUIRED \
	pangocairo >= $PANGO_REQUIRED \
	pangoft2 >= $PANGO_REQUIRED \
	cairo >= $CAIRO_REQUIRED \
	cairo-png >= $CAIRO_REQUIRED
        libcroco-0.6 >= $CROCO_REQUIRED)

PKG_CHECK_MODULES([GTHREAD],[gthread-2.0 >= $GLIB_REQUIRED])

PKG_CHECK_MODULES([GMODULE],[gmodule-2.0])

case "$host" in
  *-*-mingw*)
    PKG_CHECK_MODULES([RSVG_CONVERT],[gio-2.0 gio-windows-2.0 gdk-pixbuf-2.0 cairo pangocairo])
    ;;
  *)
    PKG_CHECK_MODULES([RSVG_CONVERT],[gio-2.0 gio-unix-2.0 gdk-pixbuf-2.0 cairo pangocairo])
    ;;
esac

GLIB_TESTS

dnl ===========================================================================

AC_CHECK_FUNCS(strtok_r)

# ===========================================================================
# GTK
# ===========================================================================

PKG_CHECK_MODULES([GTK3],[gtk+-3.0 >= $GTK3_REQUIRED],[have_gtk_3=yes],[have_gtk_3=no])

GTK3_BINARY_VERSION=

if test "$have_gtk_3" = "yes"; then
    GTK3_BINARY_VERSION="`$PKG_CONFIG --variable=gtk_binary_version gtk+-3.0`"
fi

AC_SUBST([GTK3_BINARY_VERSION])

AM_CONDITIONAL([HAVE_GTK_3],[test "$have_gtk_3" = "yes"])

dnl ===========================================================================
dnl GDK-Pixbuf SVG loader
dnl ===========================================================================

AC_MSG_CHECKING([whether to build the GDK-Pixbuf SVG loader])
AC_ARG_ENABLE([pixbuf-loader],
  [AS_HELP_STRING([--disable-pixbuf-loader],[Disable a RSVG based GdkPixbuf loader (default=yes)])],
  [],[enable_pixbuf_loader=yes])
AC_MSG_RESULT([$enable_pixbuf_loader])

gdk_pixbuf_moduledir=
if test "$enable_pixbuf_loader" = "yes"; then
  PKG_CHECK_MODULES([GDK_PIXBUF],[gdk-pixbuf-2.0 >= $GDK_PIXBUF_REQUIRED])

  AC_PATH_PROGS([GDK_PIXBUF_QUERYLOADERS],[gdk-pixbuf-query-loaders gdk-pixbuf-query-loaders-64 gdk-pixbuf-query-loaders-32],[none])

  if test "x$GDK_PIXBUF_QUERYLOADERS" = "xnone"; then
    AC_MSG_ERROR([gdk-pixbuf-query-loaders not found in path])
  fi

  # First try the standalone gdk-pixbuf
  gdk_pixbuf_binary_version="`$PKG_CONFIG --variable=gdk_pixbuf_binary_version gdk-pixbuf-2.0`"
  gdk_pixbuf_binarydir="`$PKG_CONFIG --variable=gdk_pixbuf_binarydir gdk-pixbuf-2.0`"
  gdk_pixbuf_moduledir=`$PKG_CONFIG --variable gdk_pixbuf_moduledir gdk-pixbuf-2.0`
  gdk_pixbuf_cache_file=`$PKG_CONFIG --variable gdk_pixbuf_cache_file gdk-pixbuf-2.0`
fi

AC_SUBST([gdk_pixbuf_binary_version])
AC_SUBST([gdk_pixbuf_binarydir])
AC_SUBST([gdk_pixbuf_moduledir])
AC_SUBST([gdk_pixbuf_cache_file])

AM_CONDITIONAL([ENABLE_PIXBUF_LOADER],[test "$enable_pixbuf_loader" = "yes"])

AM_CONDITIONAL(CROSS_COMPILING, test $cross_compiling = yes)

##########################################################
# Check for -Bsymbolic-functions linker flag used to avoid
# intra-library PLT jumps, if available.
##########################################################

AC_ARG_ENABLE([Bsymbolic],
  [AS_HELP_STRING([--disable-Bsymbolic],
                  [disable linking with -Bsymbolic])],
  [],[enable_Bsymbolic=yes])

BSYMBOLIC_LDFLAG=
if test "$enable_Bsymbolic" != "no"; then
  CC_CHECK_LDFLAGS(["-Wl,-Bsymbolic-functions"],
    [BSYMBOLIC_LDFLAG="-Wl,-Bsymbolic-functions"],
    [if test "$enable_Bsymbolic" = "auto"; then
       AC_MSG_WARN([-Bsymbolic not supported by ld; disabling])
       enable_Bsymbolic=no
     else
       AC_MSG_ERROR([-Bsymbolic requested but not supported by ld. Use --disable-Bsymbolic to disable])
     fi])
fi

AC_SUBST([BSYMBOLIC_LDFLAG])

##################################################
# Checks for gtk-doc and docbook-tools
##################################################

GTK_DOC_CHECK([1.13],[--flavour no-tmpl])

AC_SUBST([GLIB_PREFIX],[$($PKG_CONFIG --variable=prefix glib-2.0)])
AC_SUBST([GDK_PIXBUF_PREFIX],[$($PKG_CONFIG --variable=prefix gdk-pixbuf-2.0)])
AC_SUBST([GTK_PREFIX],[$($PKG_CONFIG --variable=prefix gdk-pixbuf-2.0)])
AC_SUBST([CAIRO_PREFIX],[$($PKG_CONFIG --variable=prefix cairo)])

dnl ===========================================================================
dnl Do we build the miscellaneous tools provided with librsvg ?

AC_ARG_ENABLE(tools,
              AC_HELP_STRING([--disable-tools],
                             [do not build miscellaneous tools [default=no]]),
              [build_misc_tools=$enableval],
              [build_misc_tools=yes])

AC_MSG_CHECKING(whether to build miscellaneous tools)
if test "x$build_misc_tools" = "xyes" ; then
	AC_MSG_RESULT(yes)
else
	AC_MSG_RESULT(no)
fi
AM_CONDITIONAL([BUILD_MISC_TOOLS], [test "x$build_misc_tools" = "xyes"])

dnl ===========================================================================
dnl Build introspectable bindings

# No automagic please!
if test -z "$enable_introspection"; then
  enable_introspection=yes
fi

GOBJECT_INTROSPECTION_CHECK([0.10.8])

# Vala bindings
VAPIGEN_CHECK([0.17.1.26],,,[no])

dnl ===========================================================================

m4_copy([AC_DEFUN],[glib_DEFUN])
glib_DEFUN([GLIB_LC_MESSAGES],
  [AC_CHECK_HEADERS([locale.h])
    if test $ac_cv_header_locale_h = yes; then
    AC_CACHE_CHECK([for LC_MESSAGES], am_cv_val_LC_MESSAGES,
      [AC_TRY_LINK([#include <locale.h>], [return LC_MESSAGES],
       am_cv_val_LC_MESSAGES=yes, am_cv_val_LC_MESSAGES=no)])
    if test $am_cv_val_LC_MESSAGES = yes; then
      AC_DEFINE(HAVE_LC_MESSAGES, 1,
        [Define if your <locale.h> file defines LC_MESSAGES.])
    fi
  fi])

GLIB_LC_MESSAGES

dnl ===========================================================================

# Check for pangoft2 and fontconfig, which is optional on Windows

PKG_CHECK_MODULES([PANGOFT2],[pangoft2 >= $PANGO_REQUIRED],[have_pangoft2=yes],[have_pangoft2=no])
PKG_CHECK_MODULES([FONTCONFIG],[fontconfig],[have_fontconfig=yes],[have_fontconfig=no])

if test "x$native_win32" != "xyes"; then
  if test "x$have_pangoft2" != "xyes" -o "x$have_fontconfig" != "xyes"; then
    AC_MSG_ERROR([pangoft2 and fontconfig are required for non-Windows platforms.])
  fi
fi

if test "x$have_pangoft2" = "xyes" -a "x$have_fontconfig" = "xyes"; then
  AC_DEFINE(HAVE_PANGOFT2, 1, [Have the pangoft2 library])
fi

dnl ===========================================================================

# Check whether MSVC toolset is explicitly set
AM_CONDITIONAL(MSVC_BASE_NO_TOOLSET_SET, [test x$MSVC_BASE_TOOLSET = x])
AM_CONDITIONAL(MSVC_NO_TOOLSET_SET, [test x$MSVC_TOOLSET = x])

dnl ===========================================================================

AC_SUBST([AM_CPPFLAGS])
AC_SUBST([AM_CFLAGS])
AC_SUBST([AM_LDFLAGS])

AC_CONFIG_FILES([
librsvg-features.h
Makefile
librsvg-zip
gdk-pixbuf-loader/Makefile
tests/Makefile
tools/Makefile
doc/Makefile
doc/version.xml
])

AC_CONFIG_FILES([librsvg-${RSVG_API_VERSION}.pc:librsvg.pc.in],[],[RSVG_API_VERSION=$RSVG_API_VERSION])
AC_CONFIG_FILES([librsvg-${RSVG_API_VERSION}-uninstalled.pc:librsvg-uninstalled.pc.in],[],[RSVG_API_VERSION=$RSVG_API_VERSION])

AC_OUTPUT

dnl =============================================================================================
printf '\nlibrsvg-og-%s\n\n' "$VERSION"
printf '    %-40s : %s\n'   'prefix' "$prefix"
printf '    %-40s : %s\n\n' 'compiler' "$CC"
printf '    %-40s : %s\n'   'Build introspectable bindings' "$found_introspection"
printf '    %-40s : %s\n'   'Build Vala bindings' "$enable_vala"
printf '    %-40s : %s\n'   'Build GdkPixbuf loader' "$enable_pixbuf_loader"
printf '    %-40s : %s\n'   "GTK+ $GTK3_REQUIRED or later" "$have_gtk_3"
printf '    %-40s : %s\n\n' 'Build miscellaneous tools' "$build_misc_tools"
