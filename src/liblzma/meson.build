tuklib_sources = tuklib_physmem
if threads != 'no'
  tuklib_sources += tuklib_cpucores
endif

subdir('common')
subdir('check')

subdir('lz')

subdir('lzma')
subdir('rangecoder')

subdir('delta')

subdir('simple')

lzmainc = include_directories(
  'api',
  'common',
  'check',
  'lz',
  'rangecoder',
  'lzma',
  'delta',
  'simple',
)

lt_cur = 7
lt_rev = 7
lt_age = 2

if host_machine.system() in ['hpux', 'netbsd', 'openbsd', 'sunos']
  liblzma_version = '@0@.@1@'.format(lt_cur, lt_rev)
elif host_machine.system() == 'qnx'
  liblzma_version = lt_cur.to_string()
else
  liblzma_version = '@0@.@1@.@2@'.format(lt_cur - lt_age, lt_age, lt_rev)
endif

liblzma = library('lzma',
  tuklib_sources,
  common_sources,
  check_sources,
  lz_sources,
  lzma1_sources,
  rangecoder_sources,
  delta_sources,
  simple_sources,
  version : liblzma_version,
  include_directories : [confinc, lzmainc, tukinc],
  c_args : ['-DTUKLIB_SYMBOL_PREFIX=lzma_'],
  dependencies : [core_deps, liblzma_deps],
  install : true,
)

lzma_cflags = []
if get_option('default_library') == 'static'
  lzma_cflags += '-DLZMA_API_STATIC'
endif

lzma_dep = declare_dependency(
  compile_args : lzma_cflags,
  link_with : liblzma,
  include_directories : include_directories('api')
)

pkg = import('pkgconfig')
pkg.generate(liblzma,
  name : 'liblzma',
  description : 'General purpose data compression library',
  url : project_url,
  extra_cflags : lzma_cflags,
)

meson.override_dependency('liblzma', lzma_dep)

subdir('api')
