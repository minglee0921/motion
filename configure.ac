# Process this file with autoconf to produce a configure script

AC_INIT(motion, esyscmd(['./version.sh']))
AC_GNU_SOURCE
AC_CONFIG_SRCDIR([motion.c])
AC_CONFIG_HEADERS(config.h)
AC_PROG_CC


###############################################################################
###  Host system
###  ToDo:  Evaluate for replacing with case statement on autotools $host_os
###############################################################################
AC_MSG_CHECKING(for Darwin/BSD)
DISTRO=""
DISTRO=`uname -a | grep -i "Darwin"`
if test "x${DISTRO}" = "x"; then
    DISTRO=`uname -s | grep -i "FreeBSD"`
fi
if test "x${DISTRO}" = "x"; then
    DISTRO=`uname -s | grep -i "NetBSD"`
fi
if test "x${DISTRO}" = "x"; then
    DISTRO=`uname -s | grep -i "OpenBSD"`
fi
if test "x${DISTRO}" = "x"; then
    DISTRO="Linux"
    AC_MSG_RESULT(no)
else
    AC_MSG_RESULT($DISTRO)
fi
AC_SUBST(DISTRO)

###############################################################################
###  Host specific paths
###############################################################################
TEMP_LIBS=""
TEMP_CFLAGS=""
TEMP_CPPFLAGS=""
TEMP_LDFLAGS=""
if test "${DISTRO}" = "Darwin"; then
    TEMP_CFLAGS="${CFLAGS} -I/sw/include"
    TEMP_CPPFLAGS="${CPPFLAGS} -I/sw/include"
    TEMP_LDFLAGS="${LDFLAGS} -L/sw/lib"
    TEMP_LIBS="-L/sw/lib"
else
    if test "x${DISTRO}" != "Linux"; then
        TEMP_CFLAGS="${CFLAGS} -I/usr/local/include"
        TEMP_CPPFLAGS="${CPPFLAGS} -I/usr/local/include"
        TEMP_LDFLAGS="${LDFLAGS} -L/usr/local/lib"
        TEMP_LIBS="-L/usr/local/lib"
    fi
fi
TEMP_LIBS="-lm ${TEMP_LIBS}"
TEMP_CFLAGS="${TEMP_CFLAGS} ${CFLAGS}"
TEMP_CPPFLAGS="${TEMP_CPPFLAGS} ${CPPFLAGS}"
TEMP_LDFLAGS="${TEMP_LDFLAGS} ${LDFLAGS}"

AC_SUBST(CFLAGS, "${TEMP_CFLAGS}")
AC_SUBST(CPPFLAGS, "${TEMP_CPPFLAGS}")
AC_SUBST(LDFLAGS, "${TEMP_LDFLAGS}")

###############################################################################
###  Video System
###############################################################################
BKTR="yes"
AC_ARG_WITH(bktr,
  AS_HELP_STRING([--without-bktr],[Exclude to use bktr subsystem for BSD]),
  BKTR="$withval")

V4L2="yes"
AC_ARG_WITH(v4l2,
  AS_HELP_STRING([--without-v4l2],[Disable V4L2 devices]),
  V4L2="$withval")

###
### Check for headers.  If needed headers are not found, flip switch to off.
###
if test "x${BKTR}" = "xyes"; then
    if test "${DISTRO}" = "FreeBSD"; then
        AC_CHECK_HEADERS(dev/bktr/ioctl_meteor.h dev/bktr/ioctl_bt848.h,[BKTR="yes"],[BKTR="no"])
    elif test "${DISTRO}" = "OpenBSD" || test "${DISTRO}" = "NetBSD"; then
        AC_CHECK_HEADERS(dev/ic/bt8xx.h,[BKTR="yes"],[BKTR="no"])
    else
        BKTR="no"
    fi
fi

if test "${V4L2}" = "yes"; then
    AC_CHECK_HEADERS(linux/videodev2.h,[V4L2="yes"],[V4L2="no"])
fi
if test "x${V4L2}" = "xyes"; then
    AC_DEFINE([HAVE_V4L2], 1, [Define to 1 if V4L2 is around])
fi
if test "x${BKTR}" = "xyes"; then
    AC_DEFINE([HAVE_BKTR], 1, [Define to 1 if BKTR is around])
fi

##############################################################################
###  Check for threading
##############################################################################
THREAD_CFLAGS=""
THREAD_CHECK="/usr/include/pthread.h"
LINUXTHREADS="no"
AC_ARG_WITH(linuxthreads,
[  --with-linuxthreads     Use linuxthreads in BSD instead of native pthreads
                ]
                ,
LINUXTHREADS="$withval"
)
if test "${DISTRO}" != "Linux" && test "${DISTRO}" != "Darwin" ; then
    if test "${LINUXTHREADS}" = "no"; then
        AC_MSG_CHECKING(Linuxthreads)
        AC_MSG_RESULT(skipping)
    else
        THREAD_CHECK="/usr/local/include/pthread/linuxthreads/pthread.h"
        THREAD_LIB_CHECK="/usr/local/lib/liblthread.so"
    fi
fi

if test "${DISTRO}" != "Linux" && test "${DISTRO}" != "Darwin" && test "${LINUXTHREADS}" != "no" ; then

AC_MSG_CHECKING(for linuxthreads)

#
# Check for thread header
#
    if test -f "${THREAD_CHECK}"; then
        HEADERS_THREAD_CFLAGS="-I/usr/local/include/pthread/linuxthreads"
        THREADS="yes"
    else
        THREADS="no"
    fi

#
# Check for thread lib
#
    if test -f "${THREAD_LIB_CHECK}" ; then
        THREADS="yes"
        LIB_THREAD="-llthread -llgcc_r"
    else
        THREADS="no"
    fi

# Checks for Library linuxthreads for BSD
#
# linuxthreads on BSD, ports collection
# /usr/local/include/pthreads/linuxthreads/pthread.h
# #include <linuxthreads/pthread.h>
# /usr/local/lib/libpthread.so
#

    if test "${THREADS}" = "yes"; then
        TEMP_CFLAGS="${HEADERS_THREAD_CFLAGS} $TEMP_CFLAGS -DWITH_LINUXTREADS"
        TEMP_LIBS="$TEMP_LIBS ${LIB_THREAD}"
        THREAD_CFLAGS="-D_THREAD_SAFE"
        PTHREAD_SUPPORT="yes"
        AC_MSG_RESULT($THREADS)
    else
        PTHREAD_SUPPORT="no"
        AC_MSG_RESULT($THREADS)
        echo
        echo "You do not have linuxthread installed"
        echo
    fi

elif test -f "${THREAD_CHECK}"; then


AC_MSG_CHECKING(threads)
AC_TRY_LINK([#include <pthread.h>],
[pthread_t th; pthread_join(th, 0);
 pthread_attr_init(0); pthread_cleanup_push(0, 0);
 pthread_create(0,0,0,0); pthread_cleanup_pop(0); ],
 [PTHREAD_LIB=yes])

if test x$PTHREAD_LIB != xyes; then
    if test "${DISTRO}" != "Linux" && test "${DISTRO}" != "Darwin" ; then
        TEMP_LIBS="$TEMP_LIBS -pthread"
        TEMP_CFLAGS="${TEMP_CFLAGS} -D_REENTRANT -D_THREAD_SAFE"
    else
        TEMP_LIBS="$TEMP_LIBS -lpthread"
        TEMP_CFLAGS="${TEMP_CFLAGS} -D_REENTRANT"
    fi
    PTHREAD_SUPPORT="yes"
fi
    AC_MSG_RESULT($PTHREAD_SUPPORT)

else
    echo
    echo "You do not have threads support"
    echo
fi

#
# Check for the libjpeg-turbo library
#
JPEG_TURBO="no"
JPEG_TURBO_OK="not_found"

AC_ARG_WITH(jpeg-turbo,
[  --with-jpeg-turbo[=DIR]   Specify the prefix for the install path for
                          jpeg-turbo for optimized jpeg handling (optional).
                          ],
JPEG_TURBO="$withval"
)

if test "${JPEG_TURBO}" = "no"; then
    AC_MSG_CHECKING(for libjpeg-turbo)
    AC_MSG_RESULT(skipping)
else
    AC_MSG_CHECKING(for libjpeg-turbo in -> [${JPEG_TURBO}] <-)
    if test -f ${JPEG_TURBO}/lib/libjpeg.a -o -f ${JPEG_TURBO}/lib/libjpeg.so; then
        AC_MSG_RESULT(found)
        JPEG_TURBO_OK="found"
    else
        AC_MSG_RESULT(not found)
    fi
fi


if test "${JPEG_TURBO_OK}" = "found"; then
    saved_CFLAGS="$CFLAGS"
    saved_LIBS="$LIBS"
    saved_LDFLAGS="$LDFLAGS"
    LDFLAGS="-L${JPEG_TURBO}/lib"
    CFLAGS="$CFLAGS -I${JPEG_TURBO}/include"
    LIBS="$LIBS -L${JPEG_TURBO}/lib -ljpeg"
    AC_CHECK_LIB(jpeg, jpeg_start_compress,
        [ TEMP_LIBS="$TEMP_LIBS -L${JPEG_TURBO}/lib -ljpeg"
          TEMP_CFLAGS="${TEMP_CFLAGS} -I${JPEG_TURBO}/include"
          TEMP_LDFLAGS="$TEMP_LDFLAGS $LDFLAGS"
          JPEG_SUPPORT="yes"],,)
    LIBS="$saved_LIBS"
    CFLAGS="$saved_CFLAGS"
    LDFLAGS="$saved_LDFLAGS"
    JPEG_SUPPORT_TURBO="yes"
fi


#
# Check for the special mmx accelerated jpeg library
#
JPEG_MMX="no"
JPEG_MMX_OK="not_found"
AC_ARG_WITH(jpeg-mmx,
[  --with-jpeg-mmx[=DIR]     Specify the prefix for the install path for
                          jpeg-mmx for optimized jpeg handling (optional).
                          If this is not specified motion will try to find
                          the library /usr/lib/libjpeg-mmx.a /usr/local/lib/libjpeg-mmx.a.
                          ],
JPEG_MMX="$withval"
)

#
# --without-jpeg-mmx or with-jpeg-mmx=no
#

if test "${JPEG_MMX}" = "no" || test x$JPEG_SUPPORT != xyes; then
        AC_MSG_CHECKING(for libjpeg-mmx)
        AC_MSG_RESULT(skipping)
elif test "${JPEG_MMX}" = "yes"; then
    # AUTODETECT STATIC LIB
    AC_MSG_CHECKING(for libjpeg-mmx autodetecting)

    if test -f /usr/lib/libjpeg-mmx.a ; then
        AC_MSG_RESULT(found)
        JPEG_MMX_OK="found"
        JPEG_MMX="/usr/lib"
    elif test -f /usr/local/lib/libjpeg-mmx.a ; then
        AC_MSG_RESULT(found)
        JPEG_MMX_OK="found"
        JPEG_MMX="/usr/local/lib"
    else
        AC_MSG_RESULT(not found)
    fi
else
    AC_MSG_CHECKING(for libjpeg-mmx in -> [${JPEG_MMX}] <-)
    if test -f ${JPEG_MMX}/libjpeg-mmx.a ; then
        AC_MSG_RESULT(found)
        JPEG_MMX_OK="found"
    else
        AC_MSG_RESULT(not found)
    fi
fi

if test "${JPEG_MMX_OK}" = "found"; then
    saved_CFLAGS="$CFLAGS"
    saved_LIBS="$LIBS"
    CFLAGS="$CFLAGS -I${JPEG_MMX}"
    LIBS="$LIBS -L${JPEG_MMX}"
    AC_CHECK_LIB(jpeg-mmx, jpeg_start_compress,
        [ TEMP_LIBS="$TEMP_LIBS -ljpeg-mmx"
        TEMP_CFLAGS="${TEMP_CFLAGS} -I${JPEG_MMX}"
        JPEG_SUPPORT="yes"],,)
    LIBS="$saved_LIBS"
    CFLAGS="$saved_CFLAGS"
    JPEG_SUPPORT_MMX="yes"
fi

#
# Look for _a_ jpeg lib that will work.
#
if test x$JPEG_SUPPORT != xyes ; then
  # Checks for libraries
  LDFLAGS=$TEMP_LDFLAGS

  AC_CHECK_LIB(jpeg, jpeg_set_defaults, [
        TEMP_LIBS="$TEMP_LIBS -ljpeg"
        JPEG_SUPPORT="yes"
    ], [
        echo
        echo "You do not have libjpeg installed"
        echo
    ]
  )
fi

#
# Check for the pkg-config.
#

AC_CHECK_PROG([PKGCONFIG],[pkg-config],[yes],[no])
AM_CONDITIONAL([FOUND_PKGCONFIG], [test "x$PKGCONFIG" = xyes])
AM_COND_IF([FOUND_PKGCONFIG],,[AC_MSG_ERROR([Required package 'pkg-config' not found, please check motion_guide.html and install necessary dependencies.])])


# Check for the libwebp library
#
AC_ARG_WITH([webp],
AS_HELP_STRING([--with-webp],
               [Compile with Webp image support]),
WEBP="yes",
WEBP="no")

HAVE_WEBP=""
if test "${WEBP}" = "yes"; then
  AC_MSG_CHECKING(for libwebp)
  WEBP_DEPS="libwebp"
  if pkg-config $WEBP_DEPS; then
    AC_MSG_RESULT(found)
    AC_DEFINE([HAVE_WEBP], 1, [Define to 1 if WEBP is around])
    HAVE_WEBP="yes"
    TEMP_LIBS="$TEMP_LIBS -lwebp -lwebpmux"
  else
    AC_MSG_RESULT(not found)
    AC_MSG_ERROR([Required package 'libwebp-dev' not found. Please check motion_guide.html and install necessary dependencies or use the '--without-webp' configuration option.])
  fi
fi


# Check for raspberry pi mmal interface
#
WITHOUT_MMAL="no"
AC_ARG_WITH([mmal],
AS_HELP_STRING([--without-mmal],
               [Compile without RaspberyPi mmal camera support]),
WITHOUT_MMAL="yes",
WITHOUT_MMAL="no")

if test "${WITHOUT_MMAL}" = "no"; then
  HAVE_MMAL=""
  if test "${DISTRO}" = "FreeBSD" ; then
    LIBRASPBERRYPIDEVPATH="/usr/local/include/interface/mmal"
  else
    LIBRASPBERRYPIDEVPATH="/opt/vc/include/interface/mmal"
  fi

  if test -d ${LIBRASPBERRYPIDEVPATH}; then
       HAVE_MMAL="yes"
  fi

  AS_IF([test "${HAVE_MMAL}" = "yes" ], [
           AC_SUBST(MMAL_CFLAGS)
           AC_SUBST(MMAL_OBJ)
           AC_SUBST(MMAL_LIBS)
           MMAL_OBJ="mmalcam.o raspicam/RaspiCamControl.o raspicam/RaspiCLI.o"
           MMAL_CFLAGS="-std=gnu99 -DHAVE_MMAL -Irasppicam -I/opt/vc/include"
           AS_IF([test "${DISTRO}" = "FreeBSD" ], [
                MMAL_CFLAGS="${MMAL_CFLAGS} -I/usr/local/include -I/usr/local/include/interface/vcos -I/usr/local/include/interface/vcos/pthreads/ -I/usr/local/include/interface/vmcs_host/linux"
               ])
           MMAL_LIBS="-L/opt/vc/lib -lmmal_core -lmmal_util -lmmal_vc_client -lvcos -lvchostif -lvchiq_arm"
           AC_DEFINE([HAVE_MMAL], 1, [Define to 1 if we want MMAL])
  ])
fi

#
# Check for libavcodec and libavformat from ffmpeg
#
FFMPEG_OBJ=""
AC_ARG_WITH([ffmpeg],
        AS_HELP_STRING([--with-ffmpeg[=DIR]], [Build with FFMPEG support]),
        [with_ffmpeg=$withval],
        [with_ffmpeg=yes])

AS_IF([test "x$with_ffmpeg" != "xno"], [
       AS_IF([test "x$with_ffmpeg" != "xyes"], [
          PKG_CONFIG_PATH=${with_ffmpeg}/lib/pkgconfig:$PKG_CONFIG_PATH
          export PKG_CONFIG_PATH
    ])

       AC_SUBST(FFMPEG_LIBS)
       AC_SUBST(FFMPEG_CFLAGS)
       FFMPEG_DEPS="libavutil libavformat libavcodec libswscale"
       if pkg-config $FFMPEG_DEPS; then
               FFMPEG_CFLAGS=`pkg-config --cflags $FFMPEG_DEPS`
               FFMPEG_LIBS=`pkg-config --libs $FFMPEG_DEPS`
               HAVE_FFMPEG="yes"
       else
              AC_MSG_ERROR([Required ffmpeg packages 'libavutil-dev libavformat-dev libavcodec-dev libswscale-dev' were not found.  Please check motion_guide.html and install necessary dependencies or use the '--without-ffmpeg' configuration option.])
       fi
])

AS_IF([test "${HAVE_FFMPEG}" = "yes" ], [
       FFMPEG_OBJ="ffmpeg.o"
       AC_DEFINE([HAVE_FFMPEG], 1, [Define to 1 if FFMPEG is around])
])
AC_SUBST(FFMPEG_OBJ)

#
# Check SQLITE3
#

SQLITE_OBJ=""
SQLITE3_SUPPORT="no"
AC_ARG_WITH(sqlite3,
    AS_HELP_STRING([--without-sqlite3],[Disable sqlite3 support in motion.]),
    [SQLITE3="$withval"])

if test "${SQLITE3}" = "no"; then
    AC_MSG_CHECKING(for sqlite3)
    AC_MSG_RESULT(skipping)
else
    saved_CFLAGS=$CFLAGS
    saved_LIBS=$LIBS

    # first we check to see if the sqlite3 amalgamation (sqlite3.c), is in with our source
    # this is the preferred way to use sqlite
    if test -f sqlite3.c; then
        SQLITE3_SUPPORT="yes"
        SQLITE_OBJ="sqlite3.o"
        TEMP_LIBS="$TEMP_LIBS -ldl"
        AC_DEFINE([HAVE_SQLITE3],1,[Define to 1 if you have SQLITE3])
        AC_DEFINE([HAVE_SQLITE3_EMBEDDED],1,[Define to 1 if you have SQLITE3 embedded support])
    else
        # if sqlite3.c is not found then we look for the shared library
        AC_CHECK_LIB(sqlite3, sqlite3_open,
            [
            TEMP_LIBS="$TEMP_LIBS -lsqlite3"
            SQLITE3_SUPPORT="yes"
            AC_DEFINE([HAVE_SQLITE3],1,[Define to 1 if you have SQLITE3 shared library support])
            ]
        )
    fi

    CFLAGS=$saved_CFLAGS
    LIBS=$saved_LIBS
fi
AC_SUBST(SQLITE_OBJ)

#
# Check Mysql
#

MYSQL="yes"
MYSQL_SUPPORT="no"
MYSQL_HEADERS="yes"
MYSQL_LIBS="yes"

AC_MSG_CHECKING(for mysql support)

AC_ARG_WITH(mysql,
[  --without-mysql         Disable mysql support in motion.
                          ],
MYSQL="$withval"
# if not given argument, assume standard
)

AC_ARG_WITH(mysql-lib,
[  --with-mysql-lib[=DIR]        Normally, configure will scan all possible default
                          installation paths for mysql libs. When it fails, use
                          this command to tell configure where mysql libs
                          installation root directory is.
                          ],
MYSQL_LIBS="$withval"
# if not given argument, assume standard
)


AC_ARG_WITH(mysql-include,
[  --with-mysql-include[=DIR]    Normally, configure will scan all possible default
                          installation paths for mysql include. When it fails, use
                          this command to tell configure where mysql include
                          installation root directory is.
                          ],
MYSQL_HEADERS="$withval"
# if not given argument, assume standard
)


if test "${MYSQL}" = "no"; then
    AC_MSG_RESULT(skipped)
else
    AC_MSG_RESULT(testing)
    # ******* Search mysql headers *******

    if test "${MYSQL_HEADERS}" = "yes"; then
        AC_MSG_CHECKING(autodect mysql headers)
        # Autodetect
        for w in /usr/include /usr/local/include /usr/mysql /usr/local/mysql /usr/local/mysql/include /opt /opt/mysql; do
            # check for plain setups
            if test -f $w/mysql.h; then
                MYSQL_INCDIR=$w
                break
            fi
            # check for "/usr/include/<packagename>" type setups
            if test -f $w/mysql/mysql.h; then
                MYSQL_INCDIR=$w/mysql
                break
            fi
            # check for "/usr/<packagename>/include" type setups
            if test -f $w/mysql/include/mysql.h; then
                MYSQL_INCDIR=$w/mysql/include
                break
            fi
        done
    elif test "${MYSQL_HEADERS}" = "no"; then
        AC_MSG_CHECKING(for mysql headers)
        AC_MSG_RESULT(skipped)
    else
        AC_MSG_CHECKING(for mysql headers in $MYSQL_HEADERS)
        # Manual detection for <withval>
        if test -f $MYSQL_HEADERS/mysql.h; then
            MYSQL_INCDIR=$MYSQL_HEADERS
            fi
    fi

    if test -z "$MYSQL_INCDIR" ; then
        MYSQL_HEADERS="no"
        AC_MSG_RESULT(not found)
        echo "Invalid MySQL directory - unable to find mysql.h."
    else
        AC_MSG_RESULT($MYSQL_INCDIR yes)
        MYSQL_HEADERS="yes"
    fi


    if test "${MYSQL_HEADERS}" = "yes"; then

    # ******* Search mysql libs *********
    if test "${MYSQL_LIBS}" = "yes"; then
        AC_MSG_CHECKING(autodect mysql libs)
        # Autodetect
        for w in /usr/lib64 /usr/lib /usr/local/lib /usr/mysql /usr/local/mysql /usr/local/mysql/lib /opt /opt/mysql /usr/lib/x86_64-linux-gnu; do
            # check for plain setups
            if test -f $w/libmysqlclient.a -o -f $w/libmysqlclient.so; then
                MYSQL_LIBDIR=$w
                break
            fi
            # check for "/usr/lib/<packagename>" type setups
            if test -f $w/mysql/libmysqlclient.a -o -f $w/mysql/libmysqlclient.so; then
                MYSQL_LIBDIR=$w/mysql
                break
            fi
            # check for "/usr/<packagename>/lib" type setups
            if test -f $w/mysql/lib/libmysqlclient.a -o -f $w/mysql/lib/libmysqlclient.so; then
                MYSQL_LIBDIR=$w/mysql/lib
                break
            fi
        done
    elif test "${MYSQL_LIBS}" = "no"; then
        AC_MSG_CHECKING(for mysql libs)
            AC_MSG_RESULT(skipped)
    else
        AC_MSG_CHECKING(for mysql libs in $MYSQL_LIBS)
        # Manual detection for <withval>
        if test -f $MYSQL_LIBS/libmysqlclient.a -o -f $MYSQL_LIBS/libmysqlclient.so; then
            MYSQL_LIBDIR=$MYSQL_LIBS
        fi
    fi


    if test -z "$MYSQL_LIBDIR" ; then
        AC_MSG_RESULT(not found)
        echo "Invalid MySQL directory - unable to find libmysqlclient.a or libmysqlclient.so."
    else
        AC_MSG_RESULT($MYSQL_LIBDIR)
        #LDFLAGS="-L$MYSQL_LIBDIR"
        saved_CFLAGS=$CFLAGS
        saved_LIBS=$LIBS
        CFLAGS="-I$MYSQL_INCDIR"
        LIBS="-L$MYSQL_LIBDIR"
        AC_CHECK_LIB(mysqlclient,mysql_init,[
                    TEMP_LIBS="$TEMP_LIBS -L$MYSQL_LIBDIR -lmysqlclient -lz"
                    TEMP_CFLAGS="$TEMP_CFLAGS -I$MYSQL_INCDIR"
                    MYSQL_SUPPORT="yes"
                    AC_DEFINE([HAVE_MYSQL],1,[Define to 1 if you have MySQL support])
                    ],
                    AC_MSG_ERROR(MySQL support can't build without MySQL libraries))
        CFLAGS=$saved_CFLAGS
        LIBS=$saved_LIBS
    fi

    # end mysql-include , mysql-libs
    fi

# end Mysql detection
fi


#
# Check PostgreSQL
#
PGSQL="yes"
PGSQL_SUPPORT="no"
PGSQL_HEADERS="yes"
PGSQL_LIBS="yes"

AC_DEFUN([PGSQL_INC_CHK],[if test -r $i$1/libpq-fe.h; then PGSQL_DIR=$i; PGSQL_INCDIR=$i$1])

AC_ARG_WITH(pgsql,
[  --without-pgsql         Disable PostgreSQL support in motion.
                          ],
PGSQL="$withval"
# if not given argument, assume standard
)

AC_ARG_WITH(pgsql-lib,
[  --with-pgsql-lib[=DIR]        Normally, configure will scan all possible default
                          installation paths for pgsql libs. When it fails, use
                          this command to tell configure where pgsql libs
                          installation root directory is.
                          ],
PGSQL_LIBS="$withval"
# if not given argument, assume standard
)

AC_ARG_WITH(pgsql-include,
[  --with-pgsql-include[=DIR]    Normally, configure will scan all possible default
                          installation paths for pgsql include. When it fails, use
                          this command to tell configure where pgsql include
                          installation root directory is.
                          ],
PGSQL_HEADERS="$withval"
# if not given argument, assume standard
)

AC_MSG_CHECKING(for PostgreSQL)

if test "${PGSQL}" = "no"; then
    AC_MSG_RESULT(skipped)
else
    AC_MSG_RESULT(testing)

    # ******* Search pgsql headers *******
    if test "${PGSQL_HEADERS}" = "yes"; then

        AC_MSG_CHECKING(autodect pgsql headers)
        # Autodetect
        for i in /usr /usr/local /usr/local/pgsql $PHP_PGSQL; do
            PGSQL_INC_CHK(/include)
            el[]PGSQL_INC_CHK(/include/pgsql)
            el[]PGSQL_INC_CHK(/include/postgresql)
            fi
        done

    elif test "${PGSQL_HEADERS}" = "no"; then
        AC_MSG_CHECKING(for pgsql headers)
        AC_MSG_RESULT(skipped)
    else
        AC_MSG_CHECKING(for pgsql headers in $PGSQL_HEADERS)
        # Manual detection for <withval>
        if test -f $PGSQL_HEADERS/libpq-fe.h; then
            PGSQL_INCDIR=$PGSQL_HEADERS
            fi
    fi

    if test -z "$PGSQL_INCDIR" ; then
        PGSQL_HEADERS="no"
        AC_MSG_RESULT(not found)
        echo "Invalid PostgreSQL directory - unable to find libpq-fe.h."
    else
        AC_MSG_RESULT(yes [$PGSQL_INCDIR])
        PGSQL_HEADERS="yes"
    fi


    if test "${PGSQL_HEADERS}" = "yes"; then

        # ******* Search pgsql libs *********
        if test "${PGSQL_LIBS}" = "yes"; then
            AC_MSG_CHECKING(autodect pgsql libs)
            # Autodetect
            PGSQL_INCLUDE="-I$PGSQL_INCDIR"
            PGSQL_LIBDIR=$PGSQL_DIR/lib

            if test -f /usr/lib64/libpq.so ; then
                PGSQL_LIBDIR=/usr/lib64
            elif test -f $PGSQL_DIR/lib/pgsql/libpq.so ; then
                PGSQL_LIBDIR=$PGSQL_DIR/lib/pgsql
            elif test -f $PGSQL_DIR/lib/postgresql/libpq.so ; then
                PGSQL_LIBDIR=$PGSQL_DIR/lib/postgresql
            elif test -f $PGSQL_DIR/lib/libpq.so ; then
                PGSQL_LIBDIR=$PGSQL_DIR/lib
            else
                PGSQL_LIBDIR=""
            fi

            AC_MSG_RESULT($PGSQL_LIBDIR)

        elif test "${PGSQL_LIBS}" = "no"; then
            AC_MSG_CHECKING(for pgsql libs)
                AC_MSG_RESULT(skipped)
        else
            AC_MSG_CHECKING(for pgsql libs in $PGSQL_LIBS)
            # Manual detection for <withval>
            if test -f $PGSQL_LIBS/libpq.a -o -f $PGSQL_LIBS/libpq.so; then
                PGSQL_LIBDIR=$PGSQL_LIBS
            fi
        fi


        if test -z "$PGSQL_LIBDIR" ; then
            AC_MSG_RESULT(not found)
            echo "Invalid PostgreSQL directory [$PGSQL_LIBDIR] - unable to find libpq.a or libpq.so."
        else
            #LDFLAGS="$TEMP_LDFLAGS -L$PGSQL_LIBDIR"
            saved_CFLAGS=$CFLAGS
            saved_LIBS=$LIBS
            CFLAGS="-I$PGSQL_INCDIR"
            LIBS="-L$PGSQL_LIBDIR"
            AC_CHECK_LIB(pq, PQconnectStart, [
                        PGSQL_SUPPORT="yes"
                        TEMP_LIBS="$TEMP_LIBS -L$PGSQL_LIBDIR -lpq"
                        TEMP_CFLAGS="$TEMP_CFLAGS -I$PGSQL_INCDIR"
                        AC_DEFINE([HAVE_PGSQL],1,[Define to 1 if you have PostgreSQL support])
                    ],
            AC_MSG_ERROR(PostgreSQL support can't build without PostgreSQL libraries))
            LDFLAGS=""
            CFLAGS=$saved_CFLAGS
            LIBS=$saved_LIBS
        fi

    fi # end pgsql-include , pgsql-libs

# end PostgreSQL detection
fi


#Checks for header files.
AC_HEADER_STDC
AC_CHECK_HEADERS(stdio.h unistd.h stdint.h fcntl.h time.h signal.h sys/ioctl.h sys/mman.h linux/videodev2.h sys/param.h sys/types.h)


OPTIMIZECPU="yes"

AC_ARG_WITH(optimizecpu,
[  --without-optimizecpu   Exclude autodetecting platform and cpu type.
                          This will disable the compilation of gcc
                          optimizing code by platform and cpu.
              ],
OPTIMIZECPU="$withval"
)

DEVELOPER_FLAGS="no"

AC_ARG_WITH(developer-flags,
[  --with-developer-flags  Causes practically all of the possible gcc
                          warning flags to be set.  This may produce
                          a large amount of warnings.],
DEVELOPER_FLAGS="$withval"
)

# Checks for typedefs, structures, and compiler characteristics.
AC_C_CONST

if test "${DISTRO}" != "Linux"; then
    OPTIMIZECPU=""
fi

if test "${OPTIMIZECPU}" = "yes"; then

# Try to autodetect cpu type
CPU_NAME="unknown"
CPU_TYPE="unknown"
if test -e "/proc/device-tree/model"; then
    # explicit test for RPI3 as /proc/cpuinfo reports armv7 even though
    # it is armv8
    RPI3=`grep "Raspberry Pi 3 Model" /proc/device-tree/model`
    if test "x${RPI3}" != "x"; then
        CPU_NAME="Raspberry Pi 3"
        CPU_TYPE="Raspberry Pi 3"
    CPU_OPTIONS="-mcpu=cortex-a53 -mfpu=neon-fp-armv8"
    fi
fi

if test "x${CPU_TYPE}" = "xunknown" && test -e "/proc/cpuinfo" ; then
    intel[[30]]="-march=i386"
    intel[[32]]="-march=i386"
    intel[[34]]="-march=i386"
    intel[[40]]="-march=i486"
    intel[[41]]="-march=i486"
    intel[[42]]="-march=i486"
    intel[[43]]="-march=i486"
    intel[[44]]="-march=i486"
    intel[[45]]="-march=i486"
    intel[[47]]="-march=i486"
    intel[[48]]="-march=i486"
    intel[[51]]="-march=pentium"
    intel[[52]]="-march=pentium"
    intel[[54]]="-march=pentium-mmx"
    intel[[56]]="-march=pentium-mmx"
    intel[[61]]="-march=pentiumpro"
    intel[[63]]="-march=pentium2"
    intel[[65]]="-march=pentium2"
    intel[[66]]="-march=pentium2"
    intel[[67]]="-march=pentium3"
    intel[[68]]="-march=pentium3"
    intel[[610]]="-march=pentium3"
    intel[[611]]="-march=pentium3"
    intel[[150]]="-march=pentium4"
    intel[[151]]="-march=pentium4"
    intel[[152]]="-march=pentium4"
    intel[[154]]="-march=pentium4"
    intel[[614]]="-march=prescott"
    intel[[628]]="-march=core2"
    amd[[50]]="-march=i586"
    amd[[51]]="-march=i586"
    amd[[52]]="-march=i586"
    amd[[53]]="-march=i586"
    amd[[56]]="-march=k6"
    amd[[57]]="-march=k6"
    amd[[58]]="-march=k6-2"
    amd[[510]]="-march=k6-2"
    amd[[59]]="-march=k6-3"
    amd[[513]]="-march=k6-3"
    amd[[61]]="-march=athlon"
    amd[[62]]="-march=athlon"
    amd[[63]]="-march=athlon"
    amd[[64]]="-march=athlon"
    amd[[66]]="-march=athlon"
    amd[[67]]="-march=athlon"
    amd[[68]]="-march=athlon"
    amd[[610]]="-march=athlon"
    amd[[158]]="-march=athlon-xp"
    amd[[154]]="-march=k8"
    amd[[155]]="-march=athlon64"
    amd[[1543]]="-march=athlon64"
    amd[[1544]]="-march=athlon64"
    amd[[1565]]="-march=opteron"
    amd[[1572]]="-march=k8"
    via[[67]]="-march=c3"
    via[[68]]="-march=c3"
    via[[69]]="-march=i686"
    via[[610]]="-march=i686"

    CPU_TYPE="known"
    CPU_FAMILY=`cat /proc/cpuinfo | grep "cpu family" | head -n1`
    CPU_MODEL=`cat /proc/cpuinfo | grep model[[^\ ]] | head -n1`
    CPU_NAME=`cat /proc/cpuinfo | grep "model name" | head -n1`
    CPU_FLAGS=`cat /proc/cpuinfo | grep "flags" | head -n1`
    CPU_VENDOR=`cat /proc/cpuinfo | grep "vendor_id" | head -n1`
    CPU_FAMILY=${CPU_FAMILY#*: }
    CPU_MODEL=${CPU_MODEL#*: }
    CPU_NAME=${CPU_NAME#*: }
    CPU_FLAGS=${CPU_FLAGS#*: }
    CPU_VENDOR=${CPU_VENDOR#*: }
    if test "x${CPU_VENDOR}" = "xGenuineIntel" ; then
        CPU_OPTIONS=${intel[[$CPU_FAMILY$CPU_MODEL]]}
    fi
    if test "x${CPU_VENDOR}" = "xAuthenticAMD" ; then
        CPU_OPTIONS=${amd[[$CPU_FAMILY$CPU_MODEL]]}
    fi
    if test "x${CPU_VENDOR}" = "xCentaurHauls"; then
        CPU_OPTIONS=${via[[$CPU_FAMILY$CPU_MODEL]]}
    fi
    if test "x${CPU_OPTIONS}" = "x" ; then
        CPU_TYPE="unknown"
    fi
    CPU_EXT=""
    for i in $CPU_FLAGS ; do
        case $i in
        fpu)
            CPU_FPU="-mfpmath=387"
            ;;
        mmx)
            CPU_EXT="$CPU_EXT -mmmx"
            ;;
        sse)
            CPU_FPU="-mfpmath=sse -msse"
            ;;
        sse2)
            CPU_FPU="-mfpmath=sse -msse2"
            ;;
        sse3)
            CPU_FPU="-msse3"
            ;;
        ssse3)
            CPU_FPU="-mfpmath=sse -msse2 -mssse3"
            ;;
        3dnow)
            CPU_EXT="$CPU_EXT -m3dnow"
            ;;
        esac
    done
    CPU_OPTIONS="$CPU_OPTIONS $CPU_FPU $CPU_EXT"
fi
if test "x${CPU_TYPE}" = "xunknown"; then
    CPU_TYPE=`( uname -p ) 2>&1`
    case $CPU_TYPE in
    i386)
        CPU_OPTIONS="-march=i386"
        ;;
    i486)
        CPU_OPTIONS="-march=i486"
        ;;
    Pentium2)
        CPU_OPTIONS="-march=pentium2"
        ;;
    Pentiumpro)
        CPU_OPTIONS="-march=pentiumpro"
        ;;
    Pentium*)
        CPU_OPTIONS="-march=pentium"
        ;;
    k6)
        CPU_OPTIONS="-march=k6"
        ;;
    k6-2)
        CPU_OPTIONS="-march=k6-2"
        ;;
    k6-3)
        CPU_OPTIONS="-march=k6-3"
        ;;
    "VIA C3 Ezra")
        CPU_OPTIONS="-march=c3"
        CPU_TYPE="known"
               ;;
    *)
        CPU_OPTIONS=""
        CPU_TYPE="unknown"
        ;;
    esac
    if test "x${CPU_TYPE}" = "xunknown"; then
        CPU_TYPE=`( uname -m ) 2>&1`
        case $CPU_TYPE in
        i386)
            CPU_OPTIONS="-march=i386"
            ;;
        i486)
            CPU_OPTIONS="-march=i486"
            ;;
        i586)
            CPU_OPTIONS="-march=i586"
            ;;
        i686)
            CPU_OPTIONS="-march=i686"
            ;;
        Pentium2)
            CPU_OPTIONS="-march=pentium2"
            ;;
        Pentiumpro)
            CPU_OPTIONS="-march=pentiumpro"
            ;;
        k6)
            CPU_OPTIONS="-march=k6"
            ;;
        k6-2)
            CPU_OPTIONS="-march=k6-2"
            ;;
        k6-3)
            CPU_OPTIONS="-march=k6-3"
            ;;
        *)
            CPU_OPTIONS="-march=native -mtune=native"
            ;;
        esac
    fi
fi
echo "Detected CPU: $CPU_NAME"
# Now we check if the compiler supports the detected cpu
COMPILER=$CC
for I in "$TMPDIR" "$TEMPDIR" "/tmp" ; do
    test "$I" && break
done
TMPC="$I/cpu_test-$RANDOM-$$.c"
TMPO="$I/cpu_test-$RANDOM-$$.o"
cat > $TMPC << EOF
int main(void) { return 0; }
EOF
( $COMPILER $CPU_OPTIONS -o $TMPO $TMPC ) 2>&1
TMP="$?"
rm -f $TMPO
rm -f $TMPC


if test "x${TMP}" = "x0" ; then
    echo "CPU optimization: $CPU_OPTIONS"
else
    CPU_OPTIONS=""
    echo "No CPU optimizations will be added"
fi

else
    CPU_OPTIONS=""
fi


if test "${DEVELOPER_FLAGS}" = "yes"; then
    TEMP_CFLAGS="${TEMP_CFLAGS} -W -Wshadow -Wpointer-arith -Wcast-align -Wwrite-strings -Waggregate-return -Wstrict-prototypes -Wmissing-prototypes -Wnested-externs -Winline -Wredundant-decls -Wno-long-long -ggdb -g3"
fi

CFLAGS="${TEMP_CFLAGS} $UNAME_DEFS  $CPU_OPTIONS"

LIBS="${TEMP_LIBS}"
LDFLAGS="${TEMP_LDFLAGS}"


#
# Add the right exec path for rc scripts
#
if test $prefix = "NONE";then
    BIN_PATH="$ac_default_prefix"
    if test $exec_prefix = "NONE"; then
        BIN_PATH="$BIN_PATH/bin"
    else
        BIN_PATH="$BIN_PATH/$bindir"
    fi
else
    if test $exec_prefix = "NONE";then
        BIN_PATH="$prefix/bin"
    else
        BIN_PATH="$prefix/$bindir"
    fi
fi

AC_SUBST(BIN_PATH)

AC_CONFIG_FILES([
camera1-dist.conf
camera2-dist.conf
camera3-dist.conf
camera4-dist.conf
motion-dist.conf
motion.init-FreeBSD.sh
motion.init-Debian
motion.service
motion.spec
Makefile
])
AC_OUTPUT

echo ""
echo "   **************************"
echo "      Configure status       "
echo "      ${PACKAGE_NAME} ${PACKAGE_VERSION}"
echo "   **************************"
echo


if test "${DISTRO}" = "Darwin"; then
    echo "OS             :     Darwin"
elif test "${DISTRO}" != "Linux"; then
    echo "OS             :     *BSD"
else
    echo "OS             :     Linux"
fi

if test "${PTHREAD_SUPPORT}" = "yes"; then
    echo "pthread support:     Yes"
else
    echo "pthread support:     No"
    echo "**********************************************"
    echo "** Fatal Error YOU MUST HAVE pthread Support *"
    echo "**********************************************"
fi

if test "${JPEG_SUPPORT_TURBO}" = "yes"; then
    echo "jpeg turbo support:  Yes"
elif test "${JPEG_SUPPORT_MMX}" = "yes"; then
    echo "jpeg-mmx support:    Yes"
elif test "${JPEG_SUPPORT}" = "yes"; then
    echo "jpeg support:        Yes"
else
    echo "jpeg support:        No"
    echo "**********************************************"
    echo "** Fatal Error YOU MUST HAVE jpeg Support  ***"
    echo "**********************************************"
fi

if test "${HAVE_WEBP}" = "yes"; then
   echo "webp support:        yes"
else
   echo "webp support:        no"
fi

if test "$V4L2" = "yes"; then
    echo "V4L2 support:        Yes"
else
    echo "V4L2 support:        No"
fi

if test "${BKTR}" = "yes"; then
    echo "BKTR support:        Yes"
else
    echo "BKTR support:        No"
fi

if test "${HAVE_MMAL}" = "yes"; then
    echo "MMAL support:        Yes"
    echo " ... MMAL_CFLAGS: $MMAL_CFLAGS"
    echo " ... MMAL_OBJ: $MMAL_OBJ"
    echo " ... MMAL_LIBS: $MMAL_LIBS"
elif test "${WITHOUT_MMAL}" = "yes"; then
    echo "MMAL support:        disabled"
else
    echo "MMAL support:        No"
    echo " ... libraspberrypi-dev package not installed"
fi

if test "${HAVE_FFMPEG}" = "yes"; then
    echo "FFmpeg support:      Yes"
    echo " ... FFMPEG_CFLAGS: $FFMPEG_CFLAGS"
    echo " ... FFMPEG_LIBS: $FFMPEG_LIBS"
else
    echo "FFmpeg support:      No"
fi

if test "${SQLITE3_SUPPORT}" = "yes"; then
    echo "SQLite3 support:     Yes"
else
    echo "SQLite3 support:     No"
fi

if test "${MYSQL_SUPPORT}" = "yes"; then
    echo "MYSQL support:       Yes"
else
    echo "MYSQL support:       No"
fi

if test "${PGSQL_SUPPORT}" = "yes"; then
    echo "PostgreSQL support:  Yes"
else
    echo "PostgreSQL support:  No"
fi
echo
echo "CFLAGS: $CFLAGS"
echo "LIBS: $LIBS"
echo "LDFLAGS: $LDFLAGS"
echo
echo  "Install prefix:       $prefix"
echo
