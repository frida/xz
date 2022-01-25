project('xz', 'c',
  version : '5.2.5',
  license : 'pd/lgpl2/gpl2/gpl3',
  default_options : [
    'optimization=2',
    'b_ndebug=true',
    'warning_level=2',
  ]
)

cc = meson.get_compiler('c')

# Suppress all compiler warnings when we're a subproject.
if meson.is_subproject()
  if cc.get_argument_syntax() == 'msvc'
    add_project_arguments('/w', language : 'c')
  else
    add_project_arguments('-w', language : 'c')
  endif
endif

cdata = configuration_data()

project_name = meson.project_name()
project_version = meson.project_version()
project_url = 'https://tukaani.org/xz/'
package_name = 'XZ Utils'
cdata.set_quoted('PACKAGE', project_name)
cdata.set_quoted('PACKAGE_NAME', package_name)
cdata.set_quoted('PACKAGE_TARNAME', project_name)
cdata.set_quoted('PACKAGE_VERSION', project_version)
cdata.set_quoted('PACKAGE_URL', project_url)
cdata.set_quoted('PACKAGE_BUGREPORT', 'lasse.collin@tukaani.org')
cdata.set_quoted('VERSION', project_version)
cdata.set_quoted('PACKAGE_STRING', package_name + ' ' + project_version)

headers = [
  'dlfcn.h',
  'fcntl.h',
  'getopt.h',
  'inttypes.h',
  'limits.h',
  'minix/config.h',
  'stdbool.h',
  'stdint.h',
  'stdio.h',
  'stdlib.h',
  'string.h',
  'strings.h',
  'sys/capsicum.h',
  'sys/param.h',
  'sys/stat.h',
  'sys/time.h',
  'sys/types.h',
  'unistd.h',
  'wchar.h',
]
if host_machine.cpu_family() in ['x86', 'x86_64']
  headers += 'immintrin.h'
endif
foreach h : headers
  if cc.has_header(h)
    cdata.set('HAVE_' + h.underscorify().to_upper(), 1)
  endif
endforeach

getopt_supported = cdata.has('HAVE_GETOPT_H')

core_deps = []
liblzma_deps = []
xz_deps = []


###########
# Filters #
###########

lz_filters = [
  'lzma1',
  'lzma2',
]
simple_filters = [
  'x86',
  'powerpc',
  'ia64',
  'arm',
  'armthumb',
  'sparc',
]
supported_filters = lz_filters + ['delta'] + simple_filters

desired_encoders = get_option('encoders')
if 'all' in desired_encoders
  desired_encoders = supported_filters
elif 'none' in desired_encoders
  desired_encoders = []
endif
if desired_encoders.length() > 0
  cdata.set('HAVE_ENCODERS', 1)
endif
foreach name : desired_encoders
  cdata.set('HAVE_ENCODER_' + name.to_upper(), 1)
endforeach
message('Encoders :', ', '.join(desired_encoders))

desired_decoders = get_option('decoders')
if 'all' in desired_decoders
  desired_decoders = supported_filters
elif 'none' in desired_decoders
  desired_decoders = []
endif
if desired_decoders.length() > 0
  cdata.set('HAVE_DECODERS', 1)
endif
foreach name : desired_decoders
  cdata.set('HAVE_DECODER_' + name.to_upper(), 1)
endforeach
message('Decoders :', ', '.join(desired_decoders))

if ('lzma2' in desired_encoders and 'lzma1' not in desired_encoders) \
    or ('lzma2' in desired_decoders and 'lzma1' not in desired_decoders)
  error('LZMA2 requires that LZMA1 is also enabled.')
endif

desired_filters = []
foreach name : desired_encoders + desired_decoders
  if name not in desired_filters
    desired_filters += name
  endif
endforeach

foreach name : simple_filters
  if name in desired_filters
    desired_filters += '#simple'
    break
  endif
endforeach
foreach name : simple_filters
  if name in desired_encoders
    desired_encoders += '#simple'
    break
  endif
endforeach
foreach name : simple_filters
  if name in desired_decoders
    desired_decoders += '#simple'
    break
  endif
endforeach

foreach name : lz_filters
  if name in desired_filters
    desired_filters += '#lz'
    break
  endif
endforeach
foreach name : lz_filters
  if name in desired_encoders
    desired_encoders += '#lz'
    break
  endif
endforeach
foreach name : lz_filters
  if name in desired_decoders
    desired_decoders += '#lz'
    break
  endif
endforeach


#################
# Match finders #
#################

supported_match_finders = [
  'hc3',
  'hc4',
  'bt2',
  'bt3',
  'bt4',
]
desired_match_finders = get_option('match_finders')
if 'all' in desired_match_finders
  desired_match_finders = supported_match_finders
elif 'none' in desired_match_finders
  desired_match_finders = []
endif
message('Match finders :', ', '.join(desired_match_finders))
foreach name : desired_match_finders
  cdata.set('HAVE_MF_' + name.to_upper(), 1)
endforeach


####################
# Integrity checks #
####################

supported_checks = [
  'crc32',
  'crc64',
  'sha256',
]
desired_checks = get_option('checks')
if 'all' in desired_checks
  desired_checks = supported_checks
endif
if 'crc32' not in desired_checks
  error('For now, the CRC32 check must always be enabled.')
endif
message('Integrity checks :', ', '.join(desired_checks))
foreach name : desired_checks
  cdata.set('HAVE_CHECK_' + name.to_upper(), 1)
endforeach

if get_option('sha256') == 'external'
  header_found = false
  type_found = false
  func_found = false

  headers = [
    'CommonCrypto/CommonDigest.h',
    'sha256.h',
    'sha2.h',
  ]
  foreach h : headers
    if cc.has_header(h)
      cdata.set('HAVE_' + h.underscorify().to_upper(), 1)
      header_found = true
      break
    endif
  endforeach

  if header_found
    types = [
      'CC_SHA256_CTX',
      'SHA256_CTX',
      'SHA2_CTX',
    ]
    sha256_includes = '''
      #if @0@
      # include <sys/types.h>
      #endif
      #if @1@
      # include <CommonCrypto/CommonDigest.h>
      #endif
      #if @2@
      # include <sha256.h>
      #endif
      #if @3@
      # include <sha2.h>
      #endif
      '''.format(
        cdata.has('HAVE_SYS_TYPES_H').to_int(),
        cdata.has('HAVE_COMMONCRYPTO_COMMONDIGEST_H').to_int(),
        cdata.has('HAVE_SHA256_H').to_int(),
        cdata.has('HAVE_SHA2_H').to_int(),
      )
    foreach t : types
      if cc.has_type(t, prefix : sha256_includes)
        cdata.set('HAVE_' + t.to_upper(), 1)
        type_found = true
        break
      endif
    endforeach

    if type_found
      sha256_args = []
      if cc.has_function('SHA256Init', args : ['-lmd']) \
          or cc.has_function('SHA256_Init', args : ['-lmd'])
        sha256_args += '-lmd'
        liblzma_deps += cc.find_library('md')
      endif

      funcs = [
        'CC_SHA256_Init',
        'SHA256Init',
        'SHA256_Init',
      ]
      foreach f : funcs
        if cc.has_function(f, args : sha256_args)
          cdata.set('HAVE_' + f.to_upper(), 1)
          func_found = true
          break
        endif
      endforeach
    endif
  endif

  if not (header_found and type_found and func_found)
    error('-Dsha256=external was specified but no supported external SHA-256 implementation was found.')
  endif
endif
message('SHA256 implementation :', get_option('sha256'))


###########################
# Assembler optimizations #
###########################

assembler = get_option('assembler')
if assembler == 'yes'
  assembler = 'no'
  if host_machine.system() in ['android', 'cygwin', 'darwin', 'dragonfly', 'freebsd',
                               'linux', 'netbsd', 'openbsd', 'windows']
    cpu_family = host_machine.cpu_family()
    if cpu_family in ['x86', 'x86_64'] and cc.get_argument_syntax() != 'msvc'
      assembler = cpu_family
    endif
  endif
endif
message('Assembler optimizations :', assembler)


#####################
# Size optimization #
#####################

tune = get_option('tune')
if tune == 'size'
  cdata.set('HAVE_SMALL', 1)
endif
message('Tuning for :', tune)


#############
# Threading #
#############

threads = get_option('threads')
if threads == 'yes'
  if host_machine.system() == 'windows'
    if host_machine.cpu_family() in ['x86', 'x86_64']
      threads = 'win95'
    else
      threads = 'vista'
    endif
  else
    threads = 'posix'
  endif
endif
if threads != 'no'
  message('Threading support : YES (' + threads + ')')
  if threads == 'posix'
    cdata.set('MYTHREAD_POSIX', 1)

    pthread_dep = dependency('threads')
    core_deps += pthread_dep

    if cc.has_function('clock_gettime')
      cdata.set('HAVE_CLOCK_GETTIME', 1)
    elif cc.has_function('clock_gettime', args : ['-lrt'])
      cdata.set('HAVE_CLOCK_GETTIME', 1)
      core_deps += cc.find_library('rt')
    endif

    if cc.has_function('pthread_condattr_setclock', dependencies : [pthread_dep])
      cdata.set('HAVE_PTHREAD_CONDATTR_SETCLOCK', 1)
    endif

    cdata.set10('HAVE_DECL_CLOCK_MONOTONIC', cc.has_header_symbol('time.h', 'CLOCK_MONOTONIC'))

    if cc.has_header_symbol('pthread.h', 'PTHREAD_PRIO_INHERIT',
                            args : ['-D_GNU_SOURCE'],
                            dependencies : [pthread_dep])
      cdata.set('HAVE_PTHREAD_PRIO_INHERIT', 1)
    endif
  elif threads == 'win95'
    cdata.set('MYTHREAD_WIN95', 1)
  elif threads == 'vista'
    cdata.set('MYTHREAD_VISTA', 1)
  endif
else
  message('Threading support : NO')
endif


#########################
# Assumed amount of RAM #
#########################

cdata.set('ASSUME_RAM', get_option('assume_ram'))


##############
# Sandboxing #
##############

# TODO: HAVE_CAPSICUM


#################################################################
# Checks for typedefs, structures, and compiler characteristics #
#################################################################

cdata.set('SIZEOF_SIZE_T', cc.sizeof('size_t'))

if cc.has_header_symbol('inttypes.h', 'uintptr_t')
  cdata.set('HAVE_UINTPTR_T', 1)
endif

if cc.has_type('_Bool')
  cdata.set('HAVE__BOOL', 1)
endif

if host_machine.endian() == 'big'
  cdata.set('WORDS_BIGENDIAN', 1)
endif

if cc.has_function('__builtin_bswap16') \
    and cc.has_function('__builtin_bswap32') \
    and cc.has_function('__builtin_bswap64')
  cdata.set('HAVE___BUILTIN_BSWAPXX', 1)
else
  foreach h : ['byteswap.h', 'sys/endian.h', 'sys/byteorder.h']
    if cc.has_header(h)
      cdata.set('HAVE_' + h.underscorify().to_upper(), 1)
      break
    endif
  endforeach
  if cdata.has('HAVE_BYTESWAP_H')
    foreach f : ['bswap_16', 'bswap_32', 'bswap_64']
      if cc.has_function(f)
        cdata.set('HAVE_' + f.to_upper(), 1)
      endif
    endforeach
  endif
endif

val = get_option('unaligned_access')
if val.auto()
  unaligned_access = host_machine.cpu_family() in ['ppc', 'ppc64', 'x86', 'x86_64']
else
  unaligned_access = val.enabled()
endif
if unaligned_access
  cdata.set('TUKLIB_FAST_UNALIGNED_ACCESS', 1)
endif
message('Fast unaligned access :', unaligned_access.to_string('YES', 'NO'))

unsafe_type_punning = get_option('unsafe_type_punning')
if unsafe_type_punning
  cdata.set('TUKLIB_USE_UNSAFE_TYPE_PUNNING', 1)
endif
message('Unsafe type punning :', unsafe_type_punning.to_string('YES', 'NO'))

if cc.has_function('__builtin_assume_aligned')
  cdata.set('HAVE___BUILTIN_ASSUME_ALIGNED', 1)
endif

# Based on GLib's implementation
have_visibility = cc.compiles('''
  void __attribute__((visibility("hidden")))   f_hidden(void)   {}
  void __attribute__((visibility("internal"))) f_internal(void) {}
  void __attribute__((visibility("default")))  f_default(void)  {}
  int
  main(void)
  {
    f_hidden();
    f_internal();
    f_default();
    return 0;
  }
  ''' + '\n',
  args : ['-Werror'],
  name : 'GNU C visibility attributes test')
if have_visibility
  cdata.set('HAVE_VISIBILITY', '1')
  add_project_arguments('-fvisibility=hidden', language : 'c')
endif

members = [
  ['struct stat', 'st_atim.tv_nsec'],
  ['struct stat', 'st_atimespec.tv_nsec'],
  ['struct stat', 'st_atimensec'],
  ['struct stat', 'st_uatime'],
  ['struct stat', 'st_atim.st__tim.tv_nsec'],
]
foreach m : members
  if cc.has_member(m[0], m[1], prefix : '#include <sys/stat.h>')
    cdata.set('HAVE_' + (m[0] + '_' + m[1]).underscorify().to_upper(), 1)
  endif
endforeach


################################
# Checks for library functions #
################################

libc_funcs = [
  'dcgettext',
  'getopt_long',
  'iconv',
  'posix_fadvise',
  'wcwidth',
]
foreach f : libc_funcs
  if cc.has_function(f)
    cdata.set('HAVE_' + f.to_upper(), 1)
  endif
endforeach
if not cdata.has('HAVE_ICONV')
  iconv = dependency('iconv', required : false)
  if iconv.found() and cc.has_function('iconv', dependencies : [iconv])
    cdata.set('HAVE_ICONV', 1)
    xz_deps += iconv
  endif
endif

if cc.has_header_symbol('wchar.h', 'mbrtowc')
  cdata.set('HAVE_MBRTOWC', 1)
endif

if cc.has_header_symbol('getopt.h', 'optreset')
  cdata.set('HAVE_OPTRESET', 1)
endif

utime_funcs = [
  'futimens',
  'futimes',
  'futimesat',
  'utimes',
  '_futime',
  'utime',
]
foreach f : utime_funcs
  if f == 'futimens' and host_machine.system() == 'darwin'
    # Avoid using APIs above the minimum version specified.
    found = cc.compiles('''
        #include <sys/stat.h>

        int
        main(void)
        {
          static struct timespec times[2];
          futimens(1, times);
          return 0;
        }
        ''',
        args : ['-Werror=unguarded-availability-new'],
        name : 'futimens is available')
  else
    found = cc.has_function(f)
  endif
  if found
    cdata.set('HAVE_' + f.to_upper(), 1)
    break
  endif
endforeach

#
# Check how to get the amount of physical memory.
# Meson port of implementation in m4/tuklib_physmem.m4.
# This information is used in tuklib_physmem.c.
#
# Supported methods:
#
#   - Windows (including Cygwin), OS/2, DJGPP (DOS), OpenVMS, AROS,
#     and QNX have operating-system specific functions.
#
#   - AIX has _system_configuration.physmem.
#
#   - sysconf() works on GNU/Linux and Solaris, and possibly on
#     some BSDs.
#
#   - BSDs use sysctl().
#
#   - Tru64 uses getsysinfo().
#
#   - HP-UX uses pstat_getstatic().
#
#   - IRIX has setinvent_r(), getinvent_r(), and endinvent_r().
#
#   - sysinfo() works on Linux/dietlibc and probably on other Linux
#     systems whose libc may lack sysconf().
#
if host_machine.system() in ['amigaos', 'aros', 'cygwin', 'dos', 'os2',
                             'qnx', 'vms', 'windows']
  physmem_method = 'special'
elif cc.compiles('''
    #include <sys/systemcfg.h>

    int
    main(void)
    {
      (void)_system_configuration.physmem;
      return 0;
    }
    ''',
    name : 'physical memory detection should be AIX-specific')
  # We look for an AIX-specific solution before sysconf(), because the test
  # for sysconf() will pass on AIX but won't actually work
  # (sysconf(_SC_PHYS_PAGES) compiles but always returns -1 on AIX).
  physmem_method = 'aix'
elif cc.compiles('''
    #include <unistd.h>

    int
    main(void)
    {
      long i;
      i = sysconf(_SC_PAGESIZE);
      i = sysconf(_SC_PHYS_PAGES);
      return 0;
    }
    ''',
    name : 'physical memory detection should use sysconf()')
  physmem_method = 'sysconf'
elif cc.compiles('''
    #include <sys/types.h>
    #if @0@
    # include <sys/param.h>
    #endif
    #include <sys/sysctl.h>

    int
    main(void)
    {
      int name[2] = { CTL_HW, HW_PHYSMEM };
      unsigned long mem;
      size_t mem_ptr_size = sizeof(mem);
      sysctl(name, 2, &mem, &mem_ptr_size, NULL, 0);
      return 0;
    }
    '''.format(cdata.has('HAVE_SYS_PARAM_H').to_int()),
    name : 'physical memory detection should use sysctl()')
  physmem_method = 'sysctl'
elif cc.compiles('''
    #include <sys/sysinfo.h>
    #include <machine/hal_sysinfo.h>

    int
    main(void)
    {
      int memkb;
      int start = 0;
      getsysinfo(GSI_PHYSMEM, (caddr_t)&memkb, sizeof(memkb), &start);
      return 0;
    }
    ''',
    name : 'physical memory detection should use getsysinfo()')
  physmem_method = 'getsysinfo'
elif cc.compiles('''
    #include <sys/param.h>
    #include <sys/pstat.h>

    int
    main(void)
    {
      struct pst_static pst;
      pstat_getstatic(&pst, sizeof(pst), 1, 0);
      (void)pst.physical_memory;
      (void)pst.page_size;
      return 0;
    }
    ''',
    name : 'physical memory detection should use pstat_getstatic()')
  physmem_method = 'pstat_getstatic'
elif cc.compiles('''
    #include <invent.h>

    int
    main(void)
    {
      inv_state_t *st = NULL;
      setinvent_r(&st);
      getinvent_r(st);
      endinvent_r(st);
      return 0;
    }
    ''',
    name : 'physical memory detection should use getinvent_r()')
  physmem_method = 'getinvent_r'
elif host_machine.system() in ['android', 'linux'] and cc.compiles('''
    #include <sys/sysinfo.h>
    int
    main(void)
    {
      struct sysinfo si;
      sysinfo(&si);
      return 0;
    }
    ''',
    name : 'physical memory detection should use sysinfo()')
  # This version of sysinfo() is Linux-specific. Some non-Linux systems have
  # different sysinfo() so we must check the host system.
  physmem_method = 'sysinfo'
else
  physmem_method = 'unknown'
endif
if physmem_method != 'unknown'
  message('Physical memory detection method :', physmem_method)
  if physmem_method != 'special'
    cdata.set('TUKLIB_PHYSMEM_' + physmem_method.to_upper(), 1)
  endif
else
  warning('Physical memory detection not supported')
endif

#
# Check how to find out the number of available CPU cores in the system.
# Meson port of implementation in m4/tuklib_cpucores.m4.
# This information is used by tuklib_cpucores.c.
#
# Supported methods:
#   - GetSystemInfo(): Windows (including Cygwin)
#   - sched_getaffinity(): glibc (GNU/Linux, GNU/kFreeBSD)
#   - cpuset_getaffinity(): FreeBSD
#   - sysctl(): BSDs, OS/2
#   - sysconf(): GNU/Linux, Solaris, Tru64, IRIX, AIX, QNX, Cygwin (but
#     GetSystemInfo() is used on Cygwin)
#   - pstat_getdynamic(): HP-UX
#
if host_machine.system() in ['cygwin', 'windows']
  cpucores_method = 'special'
elif cc.links('''
    #ifndef _GNU_SOURCE
    # define _GNU_SOURCE
    #endif
    #include <sched.h>
    int
    main(void)
    {
      cpu_set_t cpu_mask;
      sched_getaffinity(0, sizeof(cpu_mask), &cpu_mask);
      return CPU_COUNT(&cpu_mask);
    }
    ''',
    name : 'CPU cores detection should use sched_getaffinity()')
  # glibc-based systems (GNU/Linux and GNU/kFreeBSD) have sched_getaffinity().
  # The CPU_COUNT() macro was added in glibc 2.9 so we try to link the
  # test program instead of merely compiling it. glibc 2.9 is old enough that
  # if someone uses the code on older glibc, the fallback to sysconf() should
  # be good enough.
  cpucores_method = 'sched_getaffinity'
elif cc.compiles('''
    #include <sys/param.h>
    #include <sys/cpuset.h>

    int
    main(void)
    {
      cpuset_t set;
      cpuset_getaffinity(CPU_LEVEL_WHICH, CPU_WHICH_PID, -1, sizeof(set), &set);
      return 0;
    }
    ''',
    name : 'CPU cores detection should use cpuset_getaffinity()')
  # FreeBSD has both cpuset() and sysctl(). We look for cpuset() first because
  # it's a better approach.
  #
  # This test would match on GNU/kFreeBSD too but it would require
  # -lfreebsd-glue when linking and thus in the current form this would
  # fail on GNU/kFreeBSD. The above test for sched_getaffinity() matches
  # on GNU/kFreeBSD so the test below should never run on that OS.
  cpucores_method = 'cpuset_getaffinity'
elif cc.compiles('''
    #ifdef __QNX__
    # error Nope.
    #endif
    #include <sys/types.h>
    #if @0@
    # include <sys/param.h>
    #endif
    #include <sys/sysctl.h>

    int
    main(void)
    {
      int name[2] = { CTL_HW, HW_NCPU };
      int cpus;
      size_t cpus_size = sizeof(cpus);
      sysctl(name, 2, &cpus, &cpus_size, NULL, 0);
      return 0;
    }
    '''.format(cdata.has('HAVE_SYS_PARAM_H').to_int()),
    name : 'CPU cores detection should use sysctl()')
  # On OS/2, both sysconf() and sysctl() pass the tests in this file,
  # but only sysctl() works. On QNX it's the opposite: only sysconf() works
  # (although it assumes that _POSIX_SOURCE, _XOPEN_SOURCE, and _POSIX_C_SOURCE
  # are undefined or alternatively _QNX_SOURCE is defined).
  #
  # We test sysctl() first and intentionally break the sysctl() test on QNX
  # so that sysctl() is never used on QNX.
  cpucores_method = 'sysctl'
elif cc.compiles('''
    #include <unistd.h>

    int
    main(void)
    {
      long i;
    #ifdef _SC_NPROCESSORS_ONLN
      /* Many systems using sysconf() */
      i = sysconf(_SC_NPROCESSORS_ONLN);
    #else
      /* IRIX */
      i = sysconf(_SC_NPROC_ONLN);
    #endif
      return 0;
    }
    ''',
    name : 'CPU cores detection should use sysconf()')
  cpucores_method = 'sysconf'
elif cc.compiles('''
    #include <sys/param.h>
    #include <sys/pstat.h>

    int
    main(void)
    {
      struct pst_dynamic pst;
      pstat_getdynamic(&pst, sizeof(pst), 1, 0);
      (void)pst.psd_proc_cnt;
      return 0;
    }
    ''',
    name : 'CPU cores detection should use pstat_getdynamic()')
  cpucores_method = 'pstat_getdynamic'
else
  cpucores_method = 'unknown'
endif
if cpucores_method != 'unknown'
  message('CPU cores detection method :', cpucores_method)
  if cpucores_method != 'special'
    cdata.set('TUKLIB_CPUCORES_' + cpucores_method.to_upper(), 1)
  endif
else
  warning('CPU cores detection not supported')
endif

cdata.set10('HAVE_DECL_PROGRAM_INVOCATION_NAME',
            cc.has_header_symbol('errno.h', 'program_invocation_name',
                                 args : ['-D_GNU_SOURCE']))

if cc.compiles('''
    #if @0@
    # include <immintrin.h>
    #endif

    int
    main(void)
    {
      static __m128i v;
      _mm_movemask_epi8(v);
      return 0;
    }
    '''.format(cdata.has('HAVE_IMMINTRIN_H').to_int()),
    name : '_mm_movemask_epi8 is available')
  cdata.set('HAVE__MM_MOVEMASK_EPI8', 1)
endif

if cc.get_argument_syntax() != 'msvc'
  add_project_arguments(cc.get_supported_arguments(
      '-Wvla',
      '-Wformat=2',
      '-Winit-self',
      '-Wmissing-include-dirs',
      '-Wstrict-aliasing',
      '-Wfloat-equal',
      '-Wundef',
      '-Wshadow',
      '-Wpointer-arith',
      '-Wbad-function-cast',
      '-Wwrite-strings',
      '-Wlogical-op',
      '-Waggregate-return',
      '-Wstrict-prototypes',
      '-Wold-style-definition',
      '-Wmissing-prototypes',
      '-Wmissing-declarations',
      '-Wmissing-noreturn',
      '-Wredundant-decls',
    ), language : 'c')
endif

configure_file(input : 'config.h.meson',
  output : 'config.h',
  configuration : cdata
)
add_project_arguments('-DHAVE_CONFIG_H', language : 'c')

confinc = include_directories('.')
subdir('src')
