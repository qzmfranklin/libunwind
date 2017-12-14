licenses(['notice'])

# This BUILD file consists of a single public scoped target, 'libunwind', which
# depends on a private target that is suitable for a particular platform.

arch = 'x86_64'

cc_library(
    name = 'libunwind',
    visibility = [
        '//visibility:public'
    ],
    deps = [
        ':libunwind-%s' % arch,
    ],
)

cc_library(
    name = 'libunwind-x86_64',
    srcs = glob([
        # Priviate headers, including the Linux x86_64 headers.
        'include/**/*.h',
        'src/x86_64/*.h',
        'src/elf*.h',
        'src/os-linux.h',

        # Generic source code.
        'src/dwarf/L*.c',
        'src/dwarf/global.c',
        'src/mi/L*.c',
        'src/mi/backtrace.c',
        'src/mi/dyn-cancel.c',
        'src/mi/dyn-info-list.c',
        'src/mi/dyn-register.c',
        'src/mi/flush_cache.c',
        'src/mi/init.c',
        'src/mi/mempool.c',
        'src/mi/strerror.c',

        # Linux x86_64 specific source code.
        'src/elf64.c',
        'src/os-linux.c',
        'src/x86_64/*.S',
        'src/x86_64/L*.c',
        'src/x86_64/is_fpreg.c',
        'src/x86_64/regname.c',
    ], exclude = [
        # TODO (zhongming): Understand this problem better.
        # Somehow this could cause the symbol WSIZE to be undefined.  Need more
        # investigation.
        'src/mi/Ldyn-remote.c',
        'src/x86_64/Los-freebsd.c',
    ]) + [
        ':generated_headers',
    ],
    hdrs = [
        # This file is generated as part of the 'generated_headers' target.
        'include/libunwind.h',
    ],
    textual_hdrs = glob([
        'include/*/*.h',

        # Files included by other .c files.
        #
        # Using the Bazel textual_hdrs is a slight overkill as Bazel
        # textual_hdrs are also made available to clients depending on this
        # target.  But that is as small a scope as we can get right at the
        # moment.
        'src/dwarf/G*.c',
        'src/elfxx.c',
        'src/mi/G*.c',
        'src/x86_64/G*.c',
    ], exclude = [
        'src/x86_64/Gos-freebsd.c',
    ]),
    includes = [
        'include',
    ],
    copts = [
        '-Wno-error',
        '-DHAVE_CONFIG_H',
        '-D_GNU_SOURCE',
        '-I%s' % '/'.join(
                ([PACKAGE_NAME] if PACKAGE_NAME else []) + ['src']),
        '-I%s' % '/'.join(
                ([PACKAGE_NAME] if PACKAGE_NAME else []) + ['include']),
        '-I%s' % '/'.join(
                ([PACKAGE_NAME] if PACKAGE_NAME else []) + ['include/tdep']),
        '-I$(GENDIR)/include',

        # The -Wheader-guard warning misfires for src/mi/backtrace.c.
        '-Wno-header-guard',
    ],
)

# CAVEAT: Requires the host system to have autoconf, autotool, automake, and
# libtool installed.
genrule(
    name = 'generated_headers',
    tools = [
        'autogen.sh',
    ],
    outs = [
        # An easy way to retrieve this list is by taking a look at the
        # .gitignore file shipped with the libunwind git repository (wink).
        'include/config.h',
        'include/libunwind-common.h',
        'include/libunwind.h',
        'include/tdep/libunwind_i.h',
    ],
    srcs = [
        # Prefer an exhaustive list than using glob pattern to make the build
        # fully hermetic and independent of irrelevant changes in the working
        # tree.
        #
        # TODO: The list of dependencies can be retrieved by parsing the
        # configure.ac file.  Maybe we should write a python script to do that.
        'AUTHORS',
        'ChangeLog',
        'Makefile.am',
        'NEWS',
        'README',
        'configure.ac',
        'doc/Makefile.am',
        'doc/common.tex.in',
        'include/libunwind-common.h.in',
        'include/libunwind.h.in',
        'include/tdep/libunwind_i.h.in',
        'src/Makefile.am',
        'src/coredump/libunwind-coredump.pc.in',
        'src/libunwind-generic.pc.in',
        'src/mi/backtrace.c',
        'src/ptrace/libunwind-ptrace.pc.in',
        'src/setjmp/libunwind-setjmp.pc.in',
        'src/unwind/libunwind.pc.in',
        'tests/Makefile.am',
        'tests/check-namespace.sh.in',
    ],
    cmd = ' && '.join([
        # The list of commands that autoreconf invokes can be retrived by
        #   autoreconf --force -v --install 2>&1| grep '\brunning'| cut -f3 -d:
        #
        # For easy reference, they are:
        #       aclocal --force
        #       libtoolize --copy --force
        #       /usr/bin/autoconf --force
        #       /usr/bin/autoheader --force
        #       automake --add-missing --copy --force-missing
        #
        # The last command, `automake`, generates the various Makefiles.  But we
        # do not need those.  So we only run the previous commands to reduce the
        # dependencies.
        #
        # NOTE:
        #
        #   It took many hours to figure out that the various Makefile.am must
        #   be added for `automake` to work.  If not, the only error messages
        #   seen were
        #       configure.ac:430: error: required file 'doc/Makefile.in' not found
        #       configure.ac:435: error: required file 'tests/Makefile.in' not found
        #       configure.ac:438: error: required file 'src/Makefile.in' not found
        #
        #   It might be beneficial to record
        #       In /usr/bin/automake, function locate_am() uses '-f $1.am' to
        #       test if the Makefile.am file exists.  If it exsits, the
        #       corresponding Makefile.in file will be added to, @make_list, the
        #       list of Makefiles to generate. If not not, the corresponding
        #       Makefile.in will be added to, @other_input_files, the list of
        #       input files.
        '$(location autogen.sh) &> /dev/null',

        # On Ubuntu 16.04, autogen.sh already runs ./configure.  If you port
        # this BUILD file to other platforms and see errors where it cannot find
        # config/* files, uncomment the line below to unblock.
        #'./configure &> /dev/null',

        # Copy the generated files to the destination location.  These header
        # files are accessible in cc_library targets via -I$(GENDIR).
        'cp include/config.h $(location include/config.h)',
        'cp include/libunwind-common.h $(location include/libunwind-common.h)',
        'cp include/libunwind.h $(location include/libunwind.h)',
        'cp include/tdep/libunwind_i.h $(location include/tdep/libunwind_i.h)',
    ]),
)
