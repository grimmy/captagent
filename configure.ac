AC_PREREQ(2.61)
AC_INIT(captagent,6.1.0,support@sipcapture.org,,[http://www.sipcapture.org])
AC_COPYRIGHT("SIP Capture Solution")
AC_CONFIG_MACRO_DIR([m4])
AM_INIT_AUTOMAKE(foreign tar-ustar)
AC_CONFIG_HEADERS([src/config.h])
AC_CONFIG_SRCDIR([src/captagent.c])
AC_MSG_CHECKING([whether to use compression])

AC_PREFIX_DEFAULT(/usr/local/captagent)

if test "$prefix" = "NONE"; then
   prefix=$ac_default_prefix
fi

AS_AC_EXPAND(agent_config_dir, "$sysconfdir/captagent/")
AS_AC_EXPAND(agent_plan_dir, "$sysconfdir/captagent/")
AS_AC_EXPAND(module_dir, "$libdir/captagent/modules")

AC_DEFINE_UNQUOTED(AGENT_PREFIX, ["$prefix"], [our system prefix])
AC_DEFINE_UNQUOTED(AGENT_CONFIG_DIR, ["$agent_config_dir"], [captagent config dir])
AC_DEFINE_UNQUOTED(AGENT_PLAN_DIR, ["$agent_plan_dir"], [capture plans dir])
AC_DEFINE_UNQUOTED(MODULE_DIR, ["$module_dir"], [directory that modules will be installed to])

enableCompression=no
AC_ARG_ENABLE(compression,
   [  --enable-compression	Enable compression support)],
   [ZLIB="$enableval"]
   enableCompression=yes,
   [ZLIB="no"]
)
AC_MSG_RESULT([$ZLIB])
AC_SUBST([ZLIB])

AC_MSG_CHECKING([whether to use ssl])
enableSSL=no
AC_ARG_ENABLE(ssl,
   [  --enable-ssl	Enable SSL support],
   [SSL="$enableval"]
   enableSSL=yes,
   [SSL="no"]
)
AC_MSG_RESULT([$SSL])
AC_SUBST([SSL])

useMysql=no
AC_MSG_CHECKING([whether to use mysql])
AC_ARG_ENABLE(mysql,
   [  --enable-mysql    Enable mysql support],
   [MYSQL="$enableval"]
   useMysql=yes,
   [MYSQL="no"]
)
AC_MSG_RESULT([$MYSQL])
AC_SUBST([MYSQL])


usePCRE=no
AC_MSG_CHECKING([whether to use pcre])
AC_ARG_ENABLE(pcre,
   [  --enable-pcre    Enable pcre support],
   [PCRE="$enableval"]
   usePCRE=yes,
   [PCRE="no"]
)
AC_MSG_RESULT([$PCRE])
AC_SUBST([PCRE])

useRedis=no
AC_MSG_CHECKING([whether to use redis])
AC_ARG_ENABLE(redis,
   [  --enable-redis    Enable redis support],
   [REDIS="$enableval"]
   useRedis=yes,
   [REDIS="no"]
)
AC_MSG_RESULT([$REDIS])
AC_SUBST([REDIS])

useLIBUV=no
AC_MSG_CHECKING([whether to use libuv])
AC_ARG_ENABLE(libuv,
   [  --enable-libuv    Enable libuv support],
   [LIBUV="$enableval"]
   useLIBUV=yes,
   [LIBUV="no"]
)
AC_MSG_RESULT([$LIBUV])
AC_SUBST([LIBUV])

enableExtraModules=no
AC_ARG_ENABLE(extramodules,
   [  --enable-extramodules	Enable extra modules],
   [EXTRAMODULES="$enableval"]
   enableExtraModules=yes,
   [EXTRAMODULES="no"]
)
AC_MSG_RESULT([$EXTRAMODULES])
AC_SUBST([EXTRAMODULES])

CONFIG_CFLAGS="${CFLAGS}"
CONFIG_LDFLAGS="${LDFLAGS}"

MODULES='$$(grep -v "\#" $(captagent_builddir)/modules.list | sed -e "s|^.*/||" | sort | uniq )'
AM_MAKEFLAGS='"OUR_MODULES=$(MODULESS)" `test -n "$(VERBOSE)" || echo -s`'
AC_SUBST(OUR_MODS)

#AC_ENABLE_SHARED(yes)
#AC_ENABLE_STATIC(no)

#AC_CANONICAL_SYSTEM
#AM_INIT_AUTOMAKE()

LT_INIT

AC_CANONICAL_HOST
case "${host}" in
  *-*-darwin*)
        AC_DEFINE([OS_DARWIN], [1], [Define to 1 if Operating System is Darwin])
        AC_SUBST(OS_DARWIN, 1)
        ;;
  *-*-freebsd*)
        AC_DEFINE([OS_FREEBSD], [1], [Define to 1 if Operating System is FreeBSD])
        AC_SUBST(OS_FREEBSD, 1)
        ;;
  *-*-linux*)
        AC_DEFINE([OS_LINUX], [1], [Define to 1 if Operating System is Linux])
        AC_SUBST(OS_LINUX, 1)
        ;;
  *-*-netbsd*)
        AC_DEFINE([OS_NETBSD], [1], [Define to 1 if Operating System is NETBSD])
        AC_SUBST(OS_NETBSD, 1)
        ;;
  *-*-solaris2*)
        AC_DEFINE([OS_SOLARIS], [1], [Define to 1 if Operating System is SOLARIS])
        AC_SUBST(OS_SOLARIS, 1)
        ;;
  *)
        AC_MSG_RESULT([Unsupported operating system: ${host}])
        ;;
esac

# Checks for programs
AC_PROG_CC
# AC_PROG_CC([gcc cc])

AC_PROG_INSTALL
AC_PROG_LN_S
AC_PROG_MAKE_SET
AC_LIBTOOL_DLOPEN
AC_PROG_LIBTOOL
AC_PROG_LEX
if test "$LEX" != "flex"; then
	AC_MSG_ERROR([flex not found. Please install flex])
fi

if test -z "`echo %%|$LEX -t|grep yypop_buffer_state`"; then
	AC_MSG_ERROR([flex missing yypop_buffer_state - upgrade to version 2.5.33 or later])
fi

AC_PROG_YACC
if test "$YACC" != "bison -y"; then
	AC_MSG_ERROR([bison not found. Please install bison])
fi


# Checks for libraries.
AC_CHECK_LIB(pthread, pthread_create, , [AC_MSG_ERROR([captagent requires but cannot find pthread])])
if test "$OS_LINUX" = 1 ; then
  AC_CHECK_LIB(dl, dlopen, , [AC_MSG_ERROR([captagent requires but cannot find libdl])])
fi

AC_CHECK_LIB(expat, XML_ParserCreate, , [AC_MSG_ERROR([captagent requires but cannot find libexpat])])
AC_CHECK_LIB(pcap, pcap_open_live, ,[AC_CHECK_LIB(wpcap, pcap_open_live, ,[AC_MSG_ERROR([captagent requires but cannot find libpcap])])])

AC_CHECK_LIB(json, json_object_get,[ JSON_LIBS="-ljson" ],[
AC_CHECK_LIB(json-c, json_object_get,[ JSON_LIBS="-ljson-c" ],[
echo "ERROR:  You need libjson to build CaptAgent API module.";
echo "        Verify that you have libjson.a or libjson.so installed";
echo "        If it is in a different directory, try using";
echo "        the LDFLAGS to set its proper path.";
AC_MSG_ERROR([Fatal:  libjson not found.])])])

AC_CHECK_LIB(fl, yywrap, [ FLEX_LIBS="-lfl" ] , [AC_MSG_ERROR([captagent requires but cannot find libfl])])

AC_SUBST(PTHREAD_LIBS)
AC_SUBST(DL_LIBS)
AC_SUBST(EXPAT_LIBS)
AC_SUBST(PCAP_LIBS)
AC_SUBST(JSON_LIBS)
AC_SUBST(PCRE_LIBS)
AC_SUBST(HIREDIS_LIBS)
AC_SUBST(FLEX_LIBS)


dnl
dnl check for pcre library
dnl

# Checks for libpcre
if test "$PCRE" = "yes"; then
	AC_CHECKING([for pcre Library and Header files])
	AC_CHECK_HEADER([pcre.h], ,AC_MSG_ERROR([Could not find pcre headers !]))
	AC_CHECK_LIB([pcre], [pcre_compile], [PCRE_LIBS="-lpcre"], [AC_MSG_ERROR([libpcre required])])
	AC_DEFINE(USE_PCRE, 1, [Use PCRE library])
	AC_SUBST(PCRE_LIBS)
fi

dnl
dnl check for compression library
dnl
   
if test "$ZLIB" = "yes"; then
   AC_CHECKING([for zip Library and Header files])
   AC_CHECK_HEADER(zlib.h,,[AC_MSG_ERROR([zlib.h headers not found.])])
   AC_CHECK_LIB(z, inflate, [ LIBS="${LIBS} -lz" ], [AC_MSG_ERROR([captagent requires but cannot find lz])])
   AC_DEFINE(USE_ZLIB, 1, [Use ZIP library])
fi


dnl
dnl check for redis library
dnl
dnl
dnl check for MYSQL library
dnl
 
if test "$MYSQL" = "yes"; then
    AC_CHECKING([for MYSQL Library and Header files])
    AC_CHECK_HEADER([mysql/mysql.h], ,AC_MSG_ERROR([Could not find mysql headers !]))
    AC_CHECK_LIB(mysqlclient, mysql_init, [ MYSQL_LIBS="-lmysqlclient" ], [AC_MSG_ERROR([captagent requires but cannot find mysqlclient])])
    AC_DEFINE(USE_MYSQL, 1, [Use MYSQL library])
    AC_SUBST(MYSQL_LIBS)
fi
                   

dnl
dnl check for redis library
dnl
   
if test "$REDIS" = "yes"; then
   AC_CHECKING([for redis Library and Header files])
   AC_CHECK_HEADER(hiredis/hiredis.h,,[AC_MSG_ERROR([hiredis/hiredis.h headers not found.])])
   AC_CHECK_LIB(hiredis, redisCommand, [ HIREDIS_LIBS="-lhiredis" ], [AC_MSG_ERROR([captagent requires but cannot find lhiredis])])
   AC_DEFINE(USE_REDIS, 1, [Use REDIS library])
   AC_SUBST(HIREDIS_LIBS)
fi


dnl
dnl check for extra modules
dnl
   
if test "$EXTRAMODULES" = "yes"; then
   AC_CHECKING([for extra modules files])
   AC_DEFINE(HAVE_EXTRA_MODULES, 1, [We have extra modules])
fi


dnl
dnl check for libuv library
dnl
   
if test "$LIBUV" = "yes"; then
   AC_CHECKING([for LiBUV and Header files])
   AC_CHECK_HEADER(uv.h,,[AC_MSG_ERROR([uv.h headers not found.])])
   AC_CHECK_LIB(uv, uv_now, [ UV_LIBS="-luv" ], [AC_MSG_ERROR([captagent requires but cannot find libuv])])
   AC_DEFINE(USE_LIBUV, 1, [Use lib UV])
   AC_SUBST(UV_LIBS)
fi



dnl
dnl check for OpenSSL-SSL library
dnl

if test "$SSL" = "yes"; then
   AC_CHECKING([for OpenSSL SSL Library and Header files])
   AC_CHECK_HEADER(openssl/ssl.h,, [AC_MSG_ERROR([OpenSSL SSL headers not found.])])	
   AC_CHECK_LIB(ssl, SSL_accept, [ LIBS="${LIBS} -lssl" ], [AC_MSG_ERROR([captagent requires but cannot find ssl])])
   AC_DEFINE(USE_SSL, 1, [Use OpenSSL SSL library])
fi

# Checks for header files.
AC_CHECK_HEADER(pcap.h,,[AC_MSG_ERROR([captagent cannot find pcap.h])])
AC_CHECK_HEADERS([json-c/json.h json/json.h json.h])
# check JSON
AM_CONDITIONAL([HAVE_JSON_JSON_H],[test "$ac_cv_header_json_json_h" = 'yes'])
AM_CONDITIONAL([HAVE_JSON_C_JSON_H],[test "$ac_cv_header_json_c_json_h" = 'yes'])
AM_CONDITIONAL([HAVE_JSON_H],[test "$ac_cv_header_json_h" = 'yes'])



# Checks for typedefs, structures, and compiler characteristics.
AC_TYPE_INT32_T
AC_TYPE_INT8_T
AC_TYPE_PID_T
AC_TYPE_UINT16_T
AC_TYPE_UINT32_T
AC_TYPE_UINT8_T

# Checks for library functions.
AC_FUNC_FORK
#AC_FUNC_MALLOC
AC_CHECK_FUNCS([gettimeofday memset select socket strdup strerror strndup])

AC_CONFIG_FILES([
	Makefile
	captagent.spec
	include/Makefile
	src/Makefile
	conf/captagent.xml
	pkg/debian/captagent.init
])

m4_include([m4/modules_makefiles.m4])

AC_OUTPUT


echo
echo $PACKAGE $VERSION
echo
echo Build directory............. : $captagent_builddir
echo Installation prefix......... : $prefix
echo HEP Compression............. : $enableCompression
echo HEP SSL/TLS................. : $enableSSL
echo Flex........................ : ${LEX:-NONE}
echo Bison....................... : ${YACC:-NONE}

echo
echo Build with REDIS............ : $useRedis
echo Build with MySQL............ : $useMysql
echo Build with PCRE............. : $usePCRE
echo Build with LibUV............ : $useLIBUV
echo

