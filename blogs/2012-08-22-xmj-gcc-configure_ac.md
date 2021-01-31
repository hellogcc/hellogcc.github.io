转换时间： 2021-01-31

# 简析gcc configure.ac
Posted on 2012/08/22 by xmj

Copyright (c) 2012 邢明杰（xmj@hellogcc）

GCC使用configure.ac（通过autoconf）来生成configure文件。顶层目录下的configure.ac有三千多行，用来在配置阶段做各种测试检测，并将结果替换到Makefile.in中，生成最终的Makefile。本文简单介绍了configure.ac中所要做的事请，并没有覆盖所有的细节问题。

1、导出原始的configure参数
```
progname=$0
# if PWD already has a value, it is probably wrong.
if test -n "$PWD" ; then PWD=`${PWDCMD-pwd}`; fi

# Export original configure arguments for use by sub-configures.
# Quote arguments with shell meta charatcers.
TOPLEVEL_CONFIGURE_ARGUMENTS=
set -- "$progname" "$@"
for ac_arg
do
  case "$ac_arg" in
  *" "*|*"      "*|*[[\[\]\~\#\$\^\&\*\(\)\{\}\\\|\;\<\>\?\']]*)
    ac_arg=`echo "$ac_arg" | sed "s/'/'\\\\\\\\''/g"`
    # if the argument is of the form -foo=baz, quote the baz part only
    ac_arg=`echo "'$ac_arg'" | sed "s/^'\([[-a-zA-Z0-9]]*=\)/\\1'/"` ;;
  *) ;;
  esac
  # Add the quoted argument to the list.
  TOPLEVEL_CONFIGURE_ARGUMENTS="$TOPLEVEL_CONFIGURE_ARGUMENTS $ac_arg"
done
if test "$silent" = yes; then
  TOPLEVEL_CONFIGURE_ARGUMENTS="$TOPLEVEL_CONFIGURE_ARGUMENTS --silent"
fi
# Remove the initial space we just introduced and, as these will be
# expanded by make, quote '$'.
TOPLEVEL_CONFIGURE_ARGUMENTS=`echo "x$TOPLEVEL_CONFIGURE_ARGUMENTS" | sed -e 's/^x *//' -e 's,\\$,$$,g'`
AC_SUBST(TOPLEVEL_CONFIGURE_ARGUMENTS)
```
这部分代码位于38-64行，用来将configure的参数保存到变量TOPLEVEL_CONFIGURE_ARGUMENTS中，以供子目录的configure使用。

2、一些基本的测试
```
# Find the build, host, and target systems.
ACX_NONCANONICAL_BUILD
ACX_NONCANONICAL_HOST
ACX_NONCANONICAL_TARGET
... ...
AC_CANONICAL_SYSTEM
AC_ARG_PROGRAM

m4_pattern_allow([^AS_FOR_TARGET$])dnl
m4_pattern_allow([^AS_FOR_BUILD$])dnl

# Get 'install' or 'install-sh' and its variants.
AC_PROG_INSTALL
ACX_PROG_LN
AC_PROG_LN_S
AC_PROG_SED
AC_PROG_AWK
...
```
从66到108行，测试build，host，target系统（这些命令在acx.m4中定义），测试install，ln，sed，awk。

3、确定哪些库和工具将被配置
```
# these library is used by various programs built for the build
# environment
#
build_libs="build-libiberty"

# these tools are built for the build environment
build_tools="build-texinfo build-flex build-bison build-m4 build-fixincludes"

# these libraries are used by various programs built for the host environment
#
host_libs="intl libiberty opcodes bfd readline tcl tk itcl libgui zlib libcpp libdecnumber gmp mpfr mpc isl cloog libelf libiconv"

# these tools are built for the host environment
# Note, the powerpc-eabi build depends on sim occurring before gdb in order to
# know that we are building the simulator.
# binutils, gas and ld appear in that order because it makes sense to run
# "make check" in that particular order.
# If --enable-gold is used, "gold" may replace "ld".
host_tools="texinfo flex bison binutils gas ld fixincludes gcc cgen sid sim gdb gprof etc expect dejagnu m4 utils guile fastjar gnattools"

# libgcj represents the runtime libraries only used by gcj.
libgcj="target-libffi \
        target-zlib \
        target-libjava"

# these libraries are built for the target environment, and are built after
# the host libraries and the host tools (which may be a cross compiler)
# Note that libiberty is not a target library.
target_libraries="target-libgcc \
                target-libgloss \
                target-newlib \
                target-libgomp \
                target-libatomic \
                target-libitm \
                target-libstdc++-v3 \
                target-libmudflap \
                target-libssp \
                target-libquadmath \
                target-libgfortran \
                target-boehm-gc \
                ${libgcj} \
                target-libobjc \
                target-libada \
                target-libgo"

# these tools are built using the target libraries, and are intended to
# run only in the target environment
#
# note: any program that *uses* libraries that are in the "target_libraries"
# list belongs in this list.
#
target_tools="target-rda"
```
这部分代码位于112-2036行，这部分代码主要是为了确定哪些库和工具将被配置。首先，定义变量build_libs, build_tools, host_libs, host_tools, target_libraries（这里没有使用target_libs，是因为target_libs变量在config-lang.in中已经被定义使用了，参见http://gcc.gnu.org/ml/gcc-patches/2003-06/msg02977.html）, target_tools，用来列出所有可能需要的库和工具。
```
## All tools belong in one of the four categories, and are assigned above
## We assign ${configdirs} this way to remove all embedded newlines.  This
## is important because configure will choke if they ever get through.
## ${configdirs} is directories we build using the host tools.
## ${target_configdirs} is directories we build using the target tools.
configdirs=`echo ${host_libs} ${host_tools}`
target_configdirs=`echo ${target_libraries} ${target_tools}`
build_configdirs=`echo ${build_libs} ${build_tools}`
```
然后，定义变量configdirs, target_configdirs, build_configdirs，用来列出这些库和工具所对应的将要进行配置的源码目录。
```
# Skipdirs are removed silently.
skipdirs=
# Noconfigdirs are removed loudly.
noconfigdirs=""
```
然后，定义变量skipdirs和noconfigdirs，用来列出哪些目录将被跳过，或者不进行配置。其中skipdirs中的目录将被隐式的跳过，而noconfigdirs中的目录将会在终端显式的打印出消息，告知这些目录将不被配置。
```
use_gnu_ld=
# Make sure we don't let GNU ld be added if we didn't want it.
if test x$with_gnu_ld = xno ; then
  use_gnu_ld=no
  noconfigdirs="$noconfigdirs ld gold"
fi

use_gnu_as=
# Make sure we don't let GNU as be added if we didn't want it.
if test x$with_gnu_as = xno ; then
  use_gnu_as=no
  noconfigdirs="$noconfigdirs gas"
fi
... ...
```
接下来，就是根据configure的各种选项，来设置skipdirs和noconfigdirs的值。
```
# Remove the entries in $skipdirs and $noconfigdirs from $configdirs,
# $build_configdirs and $target_configdirs.
# If we have the source for $noconfigdirs entries, add them to $notsupp.

notsupp=""
for dir in . $skipdirs $noconfigdirs ; do
  dirname=`echo $dir | sed -e s/target-//g -e s/build-//g`
  if test $dir != .  && echo " ${configdirs} " | grep " ${dir} " >/dev/null 2>&1; then
    configdirs=`echo " ${configdirs} " | sed -e "s/ ${dir} / /"`
    if test -r $srcdir/$dirname/configure ; then
      if echo " ${skipdirs} " | grep " ${dir} " >/dev/null 2>&1; then
        true
      else
        notsupp="$notsupp $dir"
      fi
    fi
  fi
  if test $dir != .  && echo " ${build_configdirs} " | grep " ${dir} " >/dev/null 2>&1; then
    build_configdirs=`echo " ${build_configdirs} " | sed -e "s/ ${dir} / /"`
    if test -r $srcdir/$dirname/configure ; then
      if echo " ${skipdirs} " | grep " ${dir} " >/dev/null 2>&1; then
        true
      else
        notsupp="$notsupp $dir"
      fi
    fi
  fi
  if test $dir != . && echo " ${target_configdirs} " | grep " ${dir} " >/dev/null 2>&1; then
    target_configdirs=`echo " ${target_configdirs} " | sed -e "s/ ${dir} / /"`
    if test -r $srcdir/$dirname/configure ; then
      if echo " ${skipdirs} " | grep " ${dir} " >/dev/null 2>&1; then
        true
      else
        notsupp="$notsupp $dir"
      fi
    fi
  fi
done
... ...
```
最后，从configdirs, target_configdirs, build_configdirs中，移除在skipdirs和noconfigdirs中存在的目录项。并且，移除那些无法进行配置（没有configure文件）的目录。

4、测试flags，复制目录
```
AC_ARG_WITH([build-sysroot],
  [AS_HELP_STRING([--with-build-sysroot=SYSROOT],
                  [use sysroot as the system root during the build])],
  [if test x"$withval" != x ; then
     SYSROOT_CFLAGS_FOR_TARGET="--sysroot=$withval"
   fi],
  [SYSROOT_CFLAGS_FOR_TARGET=])
AC_SUBST(SYSROOT_CFLAGS_FOR_TARGET)
... ...
```
从2046到2206行，这部分代码设定了CFLAGS_FOR_TARGET，CXXFLAGS_FOR_TARGET，LDFLAGS_FOR_TARGET；并且对选项–with-headers=XXX，–with-libs=XXX做了处理，分别将目录下的内容复制到$(tooldir)/sys-include和$(tooldir)/lib下。

5、设定Makefile片段（frag）
```
# Work in distributions that contain no compiler tools, like Autoconf.
host_makefile_frag=/dev/null
if test -d ${srcdir}/config ; then
case "${host}" in
  i[[3456789]]86-*-msdosdjgpp*)
    host_makefile_frag="config/mh-djgpp"
    ;;
  *-cygwin*)
    ACX_CHECK_CYGWIN_CAT_WORKS
    host_makefile_frag="config/mh-cygwin"
    ;;
... ...
# Makefile fragments.
for frag in host_makefile_frag target_makefile_frag alphaieee_frag ospace_frag;
do
  eval fragval=\$$frag
  if test $fragval != /dev/null; then
    eval $frag=${srcdir}/$fragval
  fi
done
AC_SUBST_FILE(host_makefile_frag)
AC_SUBST_FILE(target_makefile_frag)
AC_SUBST_FILE(alphaieee_frag)
AC_SUBST_FILE(ospace_frag)
```
这部分代码位于1074-1107，2226-2280，2871-2882行，用来设定host_makefile_frag，target_makefile_frag，alphaieee_frag，ospace_frag这些makefile片段的值。

6、gdb相关
```
# Create a .gdbinit file which runs the one in srcdir
# and tells GDB to look there for source files.

if test -r ${srcdir}/.gdbinit ; then
  case ${srcdir} in
    .) ;;
    *) cat > ./.gdbinit <
```
代码2287-2301行，用来创建.gdbinit文件，方便调试gcc。
```
# Determine whether gdb needs tk/tcl or not.
# Use 'maybe' since enable_gdbtk might be true even if tk isn't available
# and in that case we want gdb to be built without tk.  Ugh!
# In fact I believe gdb is the *only* package directly dependent on tk,
# so we should be able to put the 'maybe's in unconditionally and
# leave out the maybe dependencies when enable_gdbtk is false.  I'm not
# 100% sure that that's safe though.

gdb_tk="maybe-all-tcl maybe-all-tk maybe-all-itcl maybe-all-libgui"
case "$enable_gdbtk" in
  no)
    GDB_TK="" ;;
  yes)
    GDB_TK="${gdb_tk}" ;;
  *)
    # Only add the dependency on gdbtk when GDBtk is part of the gdb
    # distro.  Eventually someone will fix this and move Insight, nee
    # gdbtk to a separate directory.
    if test -d ${srcdir}/gdb/gdbtk ; then
      GDB_TK="${gdb_tk}"
    else
      GDB_TK=""
    fi
    ;;
esac
CONFIGURE_GDB_TK=`echo ${GDB_TK} | sed s/-all-/-configure-/g`
INSTALL_GDB_TK=`echo ${GDB_TK} | sed s/-all-/-install-/g`
```

代码2352-2378行，用来确定gdb是否需要tk/tcl。这里作者提到一个他认为可以进行改进，但又不太确定的地方。

7、去掉Makefile中不需要的目标（target）
```
extrasub_build=
for module in ${build_configdirs} ; do
  if test -z "${no_recursion}" \
     && test -f ${build_subdir}/${module}/Makefile; then
    echo 1>&2 "*** removing ${build_subdir}/${module}/Makefile to force reconfigure"
    rm -f ${build_subdir}/${module}/Makefile
  fi
  extrasub_build="$extrasub_build
/^@if build-$module\$/d
/^@endif build-$module\$/d
/^@if build-$module-$bootstrap_suffix\$/d
/^@endif build-$module-$bootstrap_suffix\$/d"
done
... ...
AC_CONFIG_FILES([Makefile],
  [sed "$extrasub_build" Makefile |
   sed "$extrasub_host" |
   sed "$extrasub_target" > mf$$
   mv -f mf$$ Makefile],
  [extrasub_build="$extrasub_build"
   extrasub_host="$extrasub_host"
   extrasub_target="$extrasub_target"])
AC_OUTPUT

```
这部分代码位于2382-2392，2445-2497，3199-3207行，用来删除掉Makefile（或者Makefile.in）中不需要的目标。

Makefile.in文件中有许多@if和@endif包裹的模块。configure.ac中定义了变量extrasub_build，extrasub_host，

extrasub_target，它们是根据build_configdirs，configdirs，target_configdirs的内容而定义的sed语句。

在最后的AC_CONFIG_FILES命令中，将会调用这些sed语句，修改Makefile，对@if语句块进行选择性的删除。

8、设定BUILD_CONFIG变量，以调整顶层makefile
```
# Adjust the toplevel makefile according to whether bootstrap was selected.
case $enable_bootstrap in
  yes)
    bootstrap_suffix=bootstrap
    BUILD_CONFIG=bootstrap-debug
    ;;
  no)
    bootstrap_suffix=no-bootstrap
    BUILD_CONFIG=
    ;;
esac
... ...
AC_MSG_RESULT($BUILD_CONFIG)

```
这部分代码位于2401-2443行，根据是否选择了bootstrap，来设定BUILD_CONFIG变量的值。

Makefile中会根据该变量的值将config子目录下相应的mk文件包含进来。

9、串行化configure
```
# Create the serialization dependencies.  This uses a temporary file.

AC_ARG_ENABLE([serial-configure],
[AS_HELP_STRING([[--enable-serial-[{host,target,build}-]configure]],
                [force sequential configuration of
                 sub-packages for the host, target or build
                 machine, or all sub-packages])])
... ...
serialization_dependencies=serdep.tmp
AC_SUBST_FILE(serialization_dependencies)
```

这部分代码位于2499-2547行，用来实现对串行化configure的支持。

10、对自举（bootstrap）的支持
```
# ---------------------
# GCC bootstrap support
# ---------------------

# Stage specific cflags for build.
stage1_cflags="-g"
case $build in
  vax-*-*)
    case ${GCC} in
      yes) stage1_cflags="-g -Wa,-J" ;;
      *) stage1_cflags="-g -J" ;;
    esac ;;
esac
... ...

```
这部分代码位于3118-3197行，用来设定不同阶段的命令行选项（比如stage1_cflags），

指定哪些文件在自举过程中不需要进行比较操作。
