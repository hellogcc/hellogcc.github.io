转换时间： 2021-01-31

# GCC中–with-abi和–with-arch的实现分析
Posted on 2012/05/15 by xmj

by xmj

1、gcc的configure支持–with-abi和–with-arch等选项：
```
--with-schedule=cpu
--with-arch=cpu
--with-arch-32=cpu
--with-arch-64=cpu
--with-tune=cpu
--with-tune-32=cpu
--with-tune-64=cpu
--with-abi=abi
--with-fpu=type
--with-float=type
   These configure options provide default values for the -mschedule=, -march=,
-mtune=, -mabi=, and -mfpu= options and for -mhard-float or -msoft-float. As with
--with-cpu, which switches will be accepted and acceptable values of the arguments
depend on the target.
```
2、在configure的时候，如果指定了这些选项，那么gcc/config.gcc则会将这些值保存在configure_default_options变量中：
```
#  configure_default_options
#                       Set to an initializer for configure_default_options
#                       in configargs.h, based on --with-cpu et cetera.
```
这个变量值会被写入build目录下的configargs.h文件中，形如：
```
static const struct {
 const char *name, *value;
} configure_default_options[] = { { "abi", "o32" }, { "arch", "mips1" } };
```
3、gcc提供了一个目标宏，供port来定义缺省选项的spec
```
— Macro: OPTION_DEFAULT_SPECS

   A list of specs used to support configure-time default options
(i.e. --with options) in the driver. It should be a suitable
initializer for an array of structures, each containing two strings,
without the outermost pair of surrounding braces.

   The first item in the pair is the name of the default. This must
match the code in config.gcc for the target. The second item is a spec
to apply if a default with this name was specified. The string
`%(VALUE)' in the spec will be replaced by the value of the default
everywhere it occurs.

   The driver will apply these specs to its own command line between
loading default specs files and processing DRIVER_SELF_SPECS, using
the same mechanism as DRIVER_SELF_SPECS.

   Do not define this macro if it does not need to do anything.
```
4、gcc启动时，会分析option_default_specs和configure_default_options，将configure_default_options中的值替换到相应的option_default_specs中：
```
 /* Process any configure-time defaults specified for the command line
    options, via OPTION_DEFAULT_SPECS.  */
 for (i = 0; i < ARRAY_SIZE (option_default_specs); i++)
   do_option_spec (option_default_specs[i].name,
                   option_default_specs[i].spec);
```
这样，gcc就会在传给cc1和as等程序的选项中，增加了这些在configure时指定的选项。
