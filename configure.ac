dnl Process this file with autoconf to produce a configure script.
AC_PREREQ([2.69])
AC_INIT([libcddb],[1.3.2])
AC_CONFIG_SRCDIR(configure.ac)
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_FILES([Makefile              \
                 libcddb.pc            \
                 include/Makefile      \
                 include/cddb/Makefile \
                 include/cddb/cddb_config.h \
                 include/cddb/version.h \
                 lib/Makefile          \
                 examples/Makefile     \
                 tests/Makefile        \
                 tests/settings.sh])
AC_CONFIG_MACRO_DIR([m4])
AM_INIT_AUTOMAKE([dist-bzip2])
AM_MAINTAINER_MODE([enable])

LIBCDDB_VERSION_NUM=131
AC_SUBST(LIBCDDB_VERSION_NUM)

dnl iconv uses gettext macros (dependency on config.rpath)
AM_GNU_GETTEXT
AM_GNU_GETTEXT_VERSION(0.14.4)

dnl Process custom command-line options
AC_ARG_ENABLE([loglevel],
              AS_HELP_STRING([--enable-loglevel=LVL],[set default log level to one of debug, info, warn,
                             error, critical or none (default=warn)]),
              default_loglevel=$enableval, default_loglevel=$enableval)
AS_IF([test x$default_loglevel = xdebug],
	[default_loglevel="CDDB_LOG_DEBUG"],
    [test x$default_loglevel = xinfo],
	[default_loglevel="CDDB_LOG_INFO"],
    [test x$default_loglevel = xwarn -o x$default_loglevel = xyes],
	[default_loglevel="CDDB_LOG_WARN"],
    [test x$default_loglevel = xerror],
	[default_loglevel="CDDB_LOG_ERROR"],
    [test x$default_loglevel = xcritical],
	[default_loglevel="CDDB_LOG_CRITICAL"],
    [test x$default_loglevel = xnone -o x$default_loglevel = xno],
	[default_loglevel="CDDB_LOG_NONE"],
    [test x$default_loglevel != x],
	[AC_MSG_WARN([invalid loglevel, using default])]
)

AS_IF([test x$default_loglevel != x], [
    AC_DEFINE_UNQUOTED([LOGLEVEL],$default_loglevel,[Set to default log level])
])

AC_ARG_WITH([cdio],
            AS_HELP_STRING([--without-cdio],[do not use libcdio functionality when building the
                           example program (default = use if found)]))
AC_ARG_WITH([iconv],
            AS_HELP_STRING([--without-iconv],[do not include character set conversion support using
                           the iconv library (default = enabled if found)]))

dnl Check for target
AC_CANONICAL_HOST
AS_CASE([$host],
    [*beos*], [
        AC_DEFINE([BEOS],1,[Define if compiling on BeOS system.])
    ],
    [*darwin*], [
        AC_DEFINE([CDDB_NEED_UNISTD_H],1,
        [Define if you have <unistd.h> and need it included.
         On MacOS, <regex.h> needs this but that header doesn't
         include it.])
        AC_DEFINE([CDDB_NEED_SYS_SOCKET_H],1,
        [Define if you have <sys/socket.h> and need it included.
         On MacOS, <cddb_net.h> needs this but that header doesn't
         include it.])
    ],
    [*haiku*], [
        AC_DEFINE([HAIKU],1,[Define if compiling on Haiku system.])
        AC_DEFINE([CDDB_NEED_UNISTD_H],1,
        [Define if you have <unistd.h> and need it included.])
        AC_DEFINE([CDDB_NEED_SYS_SOCKET_H],1,
        [Define if you have <sys/socket.h> and need it included.])
    ]
)

dnl Checks for programs.
AC_PROG_AWK
AC_PROG_CC
AC_PROG_CPP

LT_INIT([win32-dll])
AC_PROG_INSTALL
AC_PROG_LN_S
AC_PROG_MAKE_SET

dnl Checks for libraries.
AC_CHECK_LIB([socket], [connect])
AC_CHECK_LIB([network], [connect])

dnl Checks for header files.
AC_CHECK_HEADERS([arpa/inet.h netdb.h netinet/in.h regex.h stdlib.h string.h sys/socket.h])
AC_CHECK_HEADERS([unistd.h errno.h time.h sys/time.h fcntl.h windows.h winsock2.h])

dnl Checks for typedefs, structures, and compiler characteristics.
AC_C_CONST
AC_TYPE_SIZE_T

dnl Checks for library functions.
AC_FUNC_MALLOC
AC_FUNC_REALLOC
AC_FUNC_STAT
AC_FUNC_VPRINTF
AC_FUNC_SELECT_ARGTYPES
AC_CHECK_FUNCS([mkdir regcomp socket strdup strtol strchr memset alarm select realloc])
AC_CHECK_FUNC([gethostbyname], , AC_CHECK_LIB([nsl], [gethostbyname]))

dnl Check for libcdio
AS_IF([test x$with_cdio != xno], [
    PKG_CHECK_MODULES(LIBCDIO, libcdio >= 0.76, [
        HAVE_LIBCDIO=yes
        AC_DEFINE(HAVE_LIBCDIO, [], [Define this if you have libcdio installed])
        ],
        [AC_MSG_WARN(new enough libcdio not found. CD reading in CDDB test program disabled. Get libcdio from http://www.gnu.org/software/libcdio/)
        HAVE_LIBCDIO=no])
    AC_SUBST(LIBCDIO_LIBS)
    AC_SUBST(LIBCDIO_CFLAGS)
    ], [
    HAVE_LIBCDIO=no
])

dnl Check for iconv
AS_IF([test x$with_iconv != xno], [
    AC_CHECK_HEADERS([iconv.h], [with_iconv=1], [with_iconv=0])
    AM_ICONV
    ], [
    with_iconv=0
])
AC_SUBST(with_iconv)

dnl Check and add some GCC specific warning flags
dnl - we do this as the last thing so that a possible -Werror flag
dnl - does not cause a failure in one of the other tests above
AS_IF([test "x$GCC" != "xyes"], [
    echo "*** non GNU CC compiler detected."
    echo "*** This package has not been tested very well with non GNU compilers"
    echo "*** you should try to use 'gcc' for compiling this package."
    ], [
    WARN_CFLAGS="-Wall -Wchar-subscripts -Wmissing-prototypes -Wmissing-declarations"
    WARN_CFLAGS="$WARN_CFLAGS -Wunused -Wpointer-arith -Wwrite-strings -Wnested-externs"
    WARN_CFLAGS="$WARN_CFLAGS -Wno-sign-compare"
dnl MAINTAINER_CFLAGS="-std=c99 -pedantic -Werror -O0"
dnl MAINTAINER_CFLAGS="-Werror -O0"
    MAINTAINER_CFLAGS="-Werror"

    AS_IF([test "x$enable_maintainer_mode" = "xyes"], [
        maintainer_flags=""
        for MOPT in $MAINTAINER_CFLAGS; do
            SAVE_CFLAGS="$CFLAGS"
            CFLAGS="$CFLAGS $MOPT"
            AC_MSG_CHECKING([whether $CC accepts $MOPT])
            AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[]], [[]])],[has_option=yes],[has_option=no])
            CFLAGS="$SAVE_CFLAGS"
            AC_MSG_RESULT($has_option)
            AS_IF([test "x$has_option" = "xyes"] ,[
                maintainer_flags="$maintainer_flags $MOPT"
            ])
            unset has_option
            unset SAVE_CFLAGS
        done
        unset MOPT
        MAINTAINER_CFLAGS="$maintainer_flags"
        unset maintainer_flags
	], [
        MAINTAINER_CFLAGS=""
    ])

    warning_flags=""
    for WOPT in $WARN_CFLAGS; do
        SAVE_CFLAGS="$CFLAGS"
        CFLAGS="$CFLAGS $WOPT"
        AC_MSG_CHECKING([whether $CC accepts $WOPT])
        AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[]], [[]])],[has_option=yes],[has_option=no])
        CFLAGS="$SAVE_CFLAGS"
        AC_MSG_RESULT($has_option)
        AS_IF([test "x$has_option" = "xyes"], [
            warning_flags="$warning_flags $WOPT"
        ])
        unset has_option
        unset SAVE_CFLAGS
    done
    unset WOPT
    WARN_CFLAGS="$warning_flags"
    unset warning_flags
])
CFLAGS="$CFLAGS $WARN_CFLAGS $MAINTAINER_CFLAGS"

dnl Additional configuration (platform dependant)
AS_CASE([$host],
    [*mingw*], [
        LIBS="$LIBS -lws2_32 -lregex"
    ]
)
dnl Create makefiles
AC_OUTPUT
