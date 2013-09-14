dnl Process this file with autoconf to produce a configure script.

AC_PREREQ([2.59])

AC_INIT([mikmod],[3.2.2])
AM_INIT_AUTOMAKE([1.7 foreign])
AC_CONFIG_SRCDIR([src/mikmod.c])
AM_CONFIG_HEADER([config.h])
AM_MAINTAINER_MODE

AC_CANONICAL_HOST

dnl ==============================================================
dnl mikmod specific control variables and their default values.
dnl ==============================================================

mikmod_threads=yes

dnl =========================
dnl Configure script options.
dnl =========================

AC_ARG_ENABLE(
        threads,
        [  --enable-threads        use an own thread for the player [default=guessed]],
        if test "$enableval" = "yes"
        then
                mikmod_threads=yes
        else
                mikmod_threads=no
        fi
)

dnl ====================
dnl Checks for programs.
dnl ====================

AC_PROG_CC
AC_PROG_CPP
AC_PROG_INSTALL
AC_PROG_LN_S
AC_PROG_MAKE_SET

dnl =====================
dnl Checks for libraries.
dnl =====================

dnl libmikmod
AM_PATH_LIBMIKMOD(3.1.5, , AC_MSG_ERROR([
--- ERROR: No suitable libmikmod library found.
           You need at least libmikmod 3.1.5 for this program to work.
]))

dnl ncurses
AC_CHECK_LIB(ncurses, initscr, libcurses=ncurses, libcurses=curses)
dnl resizeterm is an optional part of ncurses
AC_CHECK_LIB($libcurses, resizeterm, AC_DEFINE(HAVE_NCURSES_RESIZETERM, 1,
		[Define if your libncurses defines resizeterm (not found in <4.2).]))
AC_HEADER_TIOCGWINSZ

dnl POSIX.4 threads
dnl ---------------
if test "$mikmod_threads" = "yes"; then
        mikmod_threads=no
        dnl AC_CHECK_HEADERS(pthread.h) unreliable
        AC_CHECK_LIB(
                pthread,
                pthread_create,
                mikmod_threads=-lpthread,
                AC_CHECK_LIB(
                        c_r,
                        pthread_attr_init,
                        mikmod_threads=-lc_r
                )
        )
fi

dnl ========================
dnl Checks for header files.
dnl ========================

AC_HEADER_STDC
AC_HEADER_SYS_WAIT
AC_CHECK_HEADERS(fcntl.h limits.h sys/ioctl.h sys/time.h unistd.h)
AC_CHECK_HEADERS(fnmatch.h)
AC_CHECK_HEADERS(ncurses.h)
AC_CHECK_HEADERS(sched.h)

dnl Checks for bogus Linux old libc5 sched.h
case "$host_os" in
    linux*)
	AC_MSG_CHECKING([whether sched.h is correct])
	AC_TRY_COMPILE([#include <sched.h>], [sched_yield();],
		broken_sched=no, broken_sched=yes)
	if test "$broken_sched" = "yes"
	then
		AC_DEFINE(BROKEN_SCHED, 1,
			  [Define if your copy of <sched.h> has a _P instead of __P (old Linux libc5)].)
		AC_MSG_RESULT(no)
	else
		AC_MSG_RESULT(yes)
	fi
	;;
esac

dnl ==============================================================
dnl Checks for typedefs, structures, and compiler characteristics.
dnl ==============================================================

AC_C_CONST
AC_TYPE_PID_T
AC_TYPE_SIZE_T
AC_HEADER_TIME

dnl =============================
dnl Checks for library functions.
dnl =============================

AC_FUNC_FNMATCH
AC_PROG_GCC_TRADITIONAL
AC_FUNC_MEMCMP
AC_TYPE_SIGNAL
AC_FUNC_VPRINTF

AC_CHECK_FUNCS(getopt_long_only, have_getopt_long_only=yes)
AC_CHECK_FUNCS(mkstemp srandom snprintf vsnprintf strerror)

AC_EGREP_HEADER(usleep, unistd.h, AC_DEFINE(HAVE_USLEEP_PROTO, 1,
		[Define if your system has the prototype for usleep(3).]))
AC_EGREP_HEADER(usleep, sys/unistd.h, AC_DEFINE(HAVE_USLEEP_PROTO))
AC_EGREP_HEADER(srandom, math.h, AC_DEFINE(SRANDOM_IN_MATH_H, 1,
		[Define if your system defines random(3) and srandom(3)
		 in math.h instead of stdlib.h.]))

dnl =================================
dnl Set PACKAGE_DATA_DIR in config.h.
dnl =================================

if test "x${datadir}" = 'x${prefix}/share'; then
  if test "x${prefix}" = "xNONE"; then
    AC_DEFINE_UNQUOTED(PACKAGE_DATA_DIR, "${ac_default_prefix}/share/${PACKAGE}",
		       [Define the directory for shared data.])
  else
    AC_DEFINE_UNQUOTED(PACKAGE_DATA_DIR, "${prefix}/share/${PACKAGE}")
  fi
else
  AC_DEFINE_UNQUOTED(PACKAGE_DATA_DIR, "${datadir}/${PACKAGE}")
fi

dnl ================
dnl Choose settings.
dnl ================

case $host in
    *-aix*)
	AC_DEFINE(AIX, 1, [Define if your system is AIX 3.* - might be needed for 4.* too.])
    ;;
esac

if test "$mikmod_threads" != "no"
then
        AC_DEFINE(HAVE_PTHREAD, 1, [Define if your system provides POSIX.4 threads.])
        CFLAGS="$CFLAGS -D_REENTRANT"
        PLAYER_LIB="$mikmod_threads $PLAYER_LIB"
        REENTRANT="-D_REENTRANT"
fi

dnl ===================
dnl Choose extra stuff.
dnl ===================

dnl solaris usleep is not thread safe, use an alternative
dnl implementation on this system

case $host in
    *-*-solaris*)
	if test "$mikmod_threads" != "no"
	then
	    have_usleep=no
	else
	    AC_CHECK_FUNCS(usleep, have_usleep=yes)
	fi
	AC_DEFINE(SOLARIS, 1, [Define if your system is SOLARIS.])
	;;
    *)
	AC_CHECK_FUNCS(usleep, have_usleep=yes)
	;;
esac

if test "$have_getopt_long_only" != "yes"
then
	EXTRA_OBJ="mgetopt.o mgetopt1.o $EXTRA_OBJ"
fi

dnl Yet another kluge to get the result of AC_FUNC_FNMATCH.
if test "$ac_cv_func_fnmatch_works" != "yes"
then
	EXTRA_OBJ="mfnmatch.o $EXTRA_OBJ"
fi
if test "$have_usleep" != "yes"
then
	EXTRA_OBJ="musleep.o $EXTRA_OBJ"
fi

PLAYER_LIB="$PLAYER_LIB -l$libcurses"

dnl =================
dnl Create Makefiles.
dnl =================

AC_SUBST(EXTRA_OBJ)
AC_SUBST(PLAYER_LIB)

AC_OUTPUT(Makefile
src/Makefile
dos/Makefile
os2/Makefile
win32/Makefile)