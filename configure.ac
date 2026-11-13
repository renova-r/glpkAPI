AC_INIT([glpkAPI],[1.3.3],[mayo.roettger@hhu.de])


dnl # --------------------------------------------------------------------
dnl # global (environment) variables
dnl # --------------------------------------------------------------------
dnl
dnl # PKG_CPPFLAGS       C preprocessor flags -I...
dnl # PKG_CFLAGS         C compiler flags
dnl # PKG_LIBS           libraries -l...


dnl # --------------------------------------------------------------------
dnl # R compiler and flags
dnl # --------------------------------------------------------------------

: ${R_HOME=`R RHOME`}
if test -z "${R_HOME}"; then
    echo "could not determine R_HOME"
    exit 1
fi

dnl # GNU R default settings
CC=`"${R_HOME}/bin/R" CMD config CC`
CFLAGS=`"${R_HOME}/bin/R" CMD config CFLAGS`
CPPFLAGS=`"${R_HOME}/bin/R" CMD config CPPFLAGS`
dnl # an explicitly given LDFLAGS (even empty) is kept, as before; R's value is only the default
if test "${LDFLAGS+set}" != "set" ; then
    LDFLAGS=`"${R_HOME}/bin/R" CMD config LDFLAGS`
fi
AC_PROG_CC
AC_PROG_CPP


dnl # --------------------------------------------------------------------
dnl # template config.h.in
dnl # --------------------------------------------------------------------

AC_CONFIG_HEADERS([src/config.h])


dnl # --------------------------------------------------------------------
dnl # test for global variables
dnl # --------------------------------------------------------------------

ARGS_OK="FALSE"

if test -n "$PKG_CPPFLAGS" && test -n "$PKG_LIBS" ; then
    ARGS_OK="TRUE"
else
    if test -n "$PKG_CPPFLAGS" || test -n "$PKG_LIBS" ; then
        AC_MSG_ERROR([Please set both variables: PKG_CPPFLAGS AND PKG_LIBS])
    fi
fi


dnl # --------------------------------------------------------------------
dnl # get arguments
dnl # --------------------------------------------------------------------

dnl # AC_ARG_WITH below resets GLPK_INCLUDE/GLPK_LIB as before (the environment is
dnl # ignored there); remember the environment values for the fallback only
glpk_env_include="$GLPK_INCLUDE"
glpk_env_lib="$GLPK_LIB"

AC_ARG_WITH(glpk-include,
            AS_HELP_STRING([--with-glpk-include=PATH],[location of glpk include directory [[/usr/local/include]]
                          ]),
            [GLPK_INCLUDE=$withval],
            [GLPK_INCLUDE=""])

AC_ARG_WITH(glpk-lib,
            AS_HELP_STRING([--with-glpk-lib=PATH],[location of GLPK callable library [[/usr/local/lib]]
                          ]),
            [GLPK_LIB=$withval],
            [GLPK_LIB=""])

AC_ARG_ENABLE(gmp,
              AS_HELP_STRING([--enable-gmp[=yes|no]],[link to GNU bignum library libgmp [[yes]]
                            ]),
              [GMP=$enableval], [GMP="yes"])

AC_ARG_ENABLE(chkargs,
              AS_HELP_STRING([--enable-chkargs[=yes|no]],[check arguments to GLPK [[yes]]
                            ]),
              [CHKARGS=$enableval], [CHKARGS="yes"])


dnl # --------------------------------------------------------------------
dnl # test arguments
dnl # --------------------------------------------------------------------

dnl # include and library directories
if test [ "$ARGS_OK" = "FALSE" ] ; then

    if test -n "$GLPK_INCLUDE" && test -n "$GLPK_LIB" ; then
    
        if test [ -d "$GLPK_INCLUDE" ] ; then
            PKG_CPPFLAGS="-I${GLPK_INCLUDE}"
        else
            AC_MSG_ERROR([directory $GLPK_INCLUDE does not exist])
        fi
    
        if test [ -d "$GLPK_LIB" ] ; then
            PKG_LIBS="-L${GLPK_LIB}"
            # PKG_LIBS="-L${GLPK_LIB} -Wl,-rpath ${GLPK_LIB}"
        else
            AC_MSG_ERROR([directory $GLPK_LIB does not exist])
        fi
    
    else
    
    dnl # if no arguments are given, try to find the glpsol executable. If is not
    dnl # found, use /usr/local and /usr
    dnl #
    dnl # That original selection is always tried first, with the original flags
    dnl # and order, and is validated by compiling and linking against GLPK. Only if
    dnl # it cannot be used (e.g. no /usr/include on macOS, host headers shadowing a
    dnl # cross compiler's) the fallbacks below are tried, in this order:
    dnl #   1. GLPK_INCLUDE/GLPK_LIB from the environment, or a single
    dnl #      --with-glpk-include/--with-glpk-lib
    dnl #   2. pkg-config glpk, using $PKG_CONFIG if set (cross wrapper), else
    dnl #      <host>-pkg-config for a cross build if it exists, else pkg-config
    dnl #   3. the prefixes of the directories in PKG_CONFIG_PATH and
    dnl #      EM_PKG_CONFIG_PATH (<prefix>/lib/pkgconfig, ...) holding include/glpk.h
    dnl #   4. the compiler's default search path (R's CPPFLAGS and LDFLAGS)
    
        glpk_prefix_given="$prefix"
        AC_PREFIX_PROGRAM(glpsol)

        if test [ "$glpk_prefix_given" != "NONE" ] ; then

            dnl # explicit --prefix=PATH: used as before

            AC_MSG_NOTICE([found executable glpsol in $prefix])
            
            if test [ -d "$prefix/include" ] ; then
                PKG_CPPFLAGS="-I$prefix/include"
            else
                AC_MSG_ERROR([directory $prefix/include does not exist])
            fi
    
            if test -d "$prefix/lib" || test -d "$prefix/lib64" ; then
                PKG_LIBS=""
                if test [ -d "$prefix/lib" ] ; then
                    PKG_LIBS="$PKG_LIBS -L$prefix/lib"
                fi
                if test [ -d "$prefix/lib64" ] ; then
                    PKG_LIBS="$PKG_LIBS -L$prefix/lib64"
                fi
            else
                AC_MSG_ERROR([neither $prefix/lib nor $prefix/lib64 exist])
            fi

        else

            dnl # the original selection: glpsol prefix, otherwise /usr and /usr/local

            glpk_cands=""
            glpk_nl='
'
            glpk_legacy_ok="yes"
            glpk_legacy_cpp=""
            glpk_legacy_libs=""

            if test [ "$prefix" = "NONE" ] ; then

                AC_MSG_NOTICE([No prefix given, trying /usr and /usr/local])

                if test -d "/usr/include" || test -d "/usr/local/include" ; then
                    glpk_legacy_cpp="-I/usr/include -I/usr/local/include"
                else
                    AC_MSG_NOTICE([directories /usr/include and /usr/local/include do not exist])
                    glpk_legacy_ok="no"
                fi

                if test -d "/usr/local/lib" || test -d "/usr/local/lib64" || test -d "/usr/lib" || test -d "/usr/lib64" ; then
                    if test [ -d "/usr/local/lib" ] ; then
                        glpk_legacy_libs="$glpk_legacy_libs -L/usr/local/lib"
                    fi
                    if test [ -d "/usr/local/lib64" ] ; then
                        glpk_legacy_libs="$glpk_legacy_libs -L/usr/local/lib64"
                    fi
                    if test [ -d "/usr/lib" ] ; then
                        glpk_legacy_libs="$glpk_legacy_libs -L/usr/lib"
                    fi
                    if test [ -d "/usr/lib64" ] ; then
                        glpk_legacy_libs="$glpk_legacy_libs -L/usr/lib64"
                    fi
                else
                    AC_MSG_NOTICE([none of the directories /usr/local/lib, /usr/local/lib64, /usr/lib of /usr/lib64 exists])
                    glpk_legacy_ok="no"
                fi

            else

                AC_MSG_NOTICE([found executable glpsol in $prefix])

                if test [ -d "$prefix/include" ] ; then
                    glpk_legacy_cpp="-I$prefix/include"
                else
                    AC_MSG_NOTICE([directory $prefix/include does not exist])
                    glpk_legacy_ok="no"
                fi

                if test -d "$prefix/lib" || test -d "$prefix/lib64" ; then
                    if test [ -d "$prefix/lib" ] ; then
                        glpk_legacy_libs="$glpk_legacy_libs -L$prefix/lib"
                    fi
                    if test [ -d "$prefix/lib64" ] ; then
                        glpk_legacy_libs="$glpk_legacy_libs -L$prefix/lib64"
                    fi
                else
                    AC_MSG_NOTICE([neither $prefix/lib nor $prefix/lib64 exist])
                    glpk_legacy_ok="no"
                fi

            fi

            if test [ "$glpk_legacy_ok" = "yes" ] ; then
                glpk_cands="${glpk_cands}${glpk_legacy_cpp}|${glpk_legacy_libs}|original selection${glpk_nl}"
            fi

            dnl # fallbacks, only used if the original selection cannot be used

            glpk_fb_inc="$GLPK_INCLUDE"
            if test -z "$glpk_fb_inc" ; then glpk_fb_inc="$glpk_env_include" ; fi
            glpk_fb_lib="$GLPK_LIB"
            if test -z "$glpk_fb_lib" ; then glpk_fb_lib="$glpk_env_lib" ; fi
            glpk_fb_cpp=""
            glpk_fb_libs=""
            if test -n "$glpk_fb_inc" && test -d "$glpk_fb_inc" ; then glpk_fb_cpp="-I${glpk_fb_inc}" ; fi
            if test -n "$glpk_fb_lib" && test -d "$glpk_fb_lib" ; then glpk_fb_libs="-L${glpk_fb_lib}" ; fi
            if test -n "$glpk_fb_cpp" || test -n "$glpk_fb_libs" ; then
                glpk_cands="${glpk_cands}${glpk_fb_cpp}|${glpk_fb_libs}|GLPK_INCLUDE/GLPK_LIB${glpk_nl}"
            fi

            glpk_pkgconfig="$PKG_CONFIG"
            if test -z "$glpk_pkgconfig" && test -n "$host_alias" ; then
                if command -v "${host_alias}-pkg-config" >/dev/null 2>&1 ; then
                    glpk_pkgconfig="${host_alias}-pkg-config"
                fi
            fi
            if test -z "$glpk_pkgconfig" ; then
                glpk_pkgconfig="pkg-config"
            fi
            glpk_pcprog=`echo "$glpk_pkgconfig" | sed 's/[[ 	]].*$//'`
            if command -v "$glpk_pcprog" >/dev/null 2>&1 ; then
                if $glpk_pkgconfig --exists glpk >/dev/null 2>&1 ; then
                    glpk_pc_cflags=`$glpk_pkgconfig --cflags glpk 2>/dev/null`
                    glpk_pc_libs=`$glpk_pkgconfig --libs-only-L glpk 2>/dev/null`
                    glpk_cands="${glpk_cands}${glpk_pc_cflags}|${glpk_pc_libs}|$glpk_pkgconfig glpk${glpk_nl}"
                fi
            fi

            glpk_pfx_done=" "
            glpk_save_IFS=$IFS
            IFS=:
            for glpk_pcdir in $PKG_CONFIG_PATH $EM_PKG_CONFIG_PATH ; do
                case "$glpk_pcdir" in
                    */lib/pkgconfig)   glpk_pfx=`echo "$glpk_pcdir" | sed 's|/lib/pkgconfig$||'` ;;
                    */lib64/pkgconfig) glpk_pfx=`echo "$glpk_pcdir" | sed 's|/lib64/pkgconfig$||'` ;;
                    */share/pkgconfig) glpk_pfx=`echo "$glpk_pcdir" | sed 's|/share/pkgconfig$||'` ;;
                    *) glpk_pfx="" ;;
                esac
                case "$glpk_pfx_done" in
                    *" $glpk_pfx "*) glpk_pfx="" ;;
                esac
                if test -n "$glpk_pfx" && test -f "$glpk_pfx/include/glpk.h" ; then
                    glpk_pfx_done="$glpk_pfx_done$glpk_pfx "
                    glpk_pfx_libs=""
                    if test -d "$glpk_pfx/lib" ; then glpk_pfx_libs="$glpk_pfx_libs -L$glpk_pfx/lib" ; fi
                    if test -d "$glpk_pfx/lib64" ; then glpk_pfx_libs="$glpk_pfx_libs -L$glpk_pfx/lib64" ; fi
                    glpk_cands="${glpk_cands}-I$glpk_pfx/include|${glpk_pfx_libs}|prefix $glpk_pfx (PKG_CONFIG_PATH)${glpk_nl}"
                fi
            done
            IFS=$glpk_save_IFS

            glpk_cands="${glpk_cands}||compiler default search path${glpk_nl}"

            if test [ "${GMP}" != "no" ] ; then
                glpk_probe_link="-lglpk -lgmp"
            else
                glpk_probe_link="-lglpk"
            fi

            glpk_save_CPPFLAGS="$CPPFLAGS"
            glpk_save_LIBS="$LIBS"
            glpk_found="no"
            glpk_save_IFS=$IFS
            IFS="$glpk_nl"
            for glpk_cand in $glpk_cands ; do
                IFS=$glpk_save_IFS
                glpk_cpp=${glpk_cand%%|*}
                glpk_rest=${glpk_cand#*|}
                glpk_lib=${glpk_rest%%|*}
                glpk_desc=${glpk_rest#*|}
                CPPFLAGS="$glpk_save_CPPFLAGS $glpk_cpp"
                LIBS="$glpk_lib $glpk_probe_link $glpk_save_LIBS"
                AC_MSG_CHECKING([for glpk.h and libglpk in $glpk_desc])
                dnl # the same system headers the header checks below use, so that a selection
                dnl # that would only fail there (e.g. host headers in a cross build) is not accepted
                AC_LINK_IFELSE([AC_LANG_PROGRAM([[#include <stdio.h>
#include <stdlib.h>
#include <stdarg.h>
#include <string.h>
#include <float.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>
#include <inttypes.h>
#include <stdint.h>
#include <glpk.h>]],
                                                [[glp_prob *lp = glp_create_prob(); glp_delete_prob(lp);]])],
                               [glpk_found="yes"], [glpk_found="no"])
                AC_MSG_RESULT([$glpk_found])
                if test [ "$glpk_found" = "yes" ] ; then
                    PKG_CPPFLAGS="$glpk_cpp"
                    PKG_LIBS="$glpk_lib"
                    break
                fi
                IFS="$glpk_nl"
            done
            IFS=$glpk_save_IFS
            CPPFLAGS="$glpk_save_CPPFLAGS"
            LIBS="$glpk_save_LIBS"

            dnl # nothing worked: keep the original selection (if there was one), so the
            dnl # checks below fail with the original messages
            if test [ "$glpk_found" = "no" ] ; then
                if test [ "$glpk_legacy_ok" = "yes" ] ; then
                    PKG_CPPFLAGS="$glpk_legacy_cpp"
                    PKG_LIBS="$glpk_legacy_libs"
                fi
                AC_MSG_NOTICE([GLPK was not found; set GLPK_INCLUDE and GLPK_LIB (or use --with-glpk-include and --with-glpk-lib), or PKG_CPPFLAGS and PKG_LIBS.])
            fi

        fi

    fi

    if test [ "${GMP}" != "no" ] ; then
        dnl # AC_CHECK_HEADER([gmp.h], INCL="$INCL -lgmp", AC_MSG_NOTICE([Could not link to gmp.]))
        LINK="-lglpk -lgmp"
        AC_DEFINE([USE_GMP], [1], [defined, if gnu bignum is used])
    else
        LINK="-lglpk"
    fi
    
    PKG_LIBS="${PKG_LIBS} ${LINK}"
    ARGS_OK="TRUE"
fi


dnl # check arguments to CPLEX
if test [ "${CHKARGS}" != "no" ] ; then
    AC_DEFINE([CHECK_GLPK_ARGS], [1], [defined, if check arguments to GLPK])
    PKG_CPPFLAGS="-DHAVE_CONFIG_H ${PKG_CPPFLAGS}"
fi


dnl # --------------------------------------------------------------------
dnl # CFLAGS
dnl # --------------------------------------------------------------------

if test [ -z "$PKG_CFLAGS" ] ; then
    if test [ "${R_ARCH}" = "/x86_64" ] ; then
        PKG_CFLAGS="-m64"
    elif test [ "${R_ARCH}" = "/i386" ] ; then
        PKG_CFLAGS="-m32"
    else
        PKG_CFLAGS=""
    fi
fi


dnl # --------------------------------------------------------------------
dnl # check header and library
dnl # --------------------------------------------------------------------

LIBS="${PKG_LIBS}"
CFLAGS="${CFLAGS} ${PKG_CFLAGS}"
CPPFLAGS="${CPPFLAGS} ${PKG_CPPFLAGS}"

AC_CHECK_HEADER([glpk.h],,
    AC_MSG_ERROR([Could not find glpk.h:
      glpkAPI requires GLPK from http://www.gnu.org/software/glpk/
      use --with-glpk-include or GLPK_INCLUDE to specify the include path.]))


AC_SEARCH_LIBS([glp_create_prob], [glpk], ,
    AC_MSG_ERROR([Could not link to GLPK:
      use --with-glpk-lib or GLPK_LIB to specify the lib path.
      Maybe you want to set --enable-gmp=no]))

dnl # GLPK version
AC_CHECK_FUNC([glp_print_ranges], ,
    AC_MSG_ERROR([GLPK version must be >= 4.42.]))


dnl # --------------------------------------------------------------------
dnl # substitute src/Makevars
dnl # --------------------------------------------------------------------

AC_SUBST(PKG_CFLAGS)
AC_SUBST(PKG_LIBS)
AC_SUBST(PKG_CPPFLAGS)
AC_CONFIG_FILES([src/Makevars])
AC_OUTPUT

exit 0
