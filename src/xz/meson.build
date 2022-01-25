sources = [
  'args.c',
  'coder.c',
  'file_io.c',
  'hardware.c',
  'main.c',
  'message.c',
  'mytime.c',
  'options.c',
  'signals.c',
  'suffix.c',
  'util.c',
]

if desired_decoders.length() > 0
  sources += 'list.c'
endif

xz = executable('xz', sources,
  tuklib_open_stdxxx,
  tuklib_progname,
  tuklib_exit,
  tuklib_mbstr,
  include_directories : [confinc, tukinc],
  dependencies : [core_deps, xz_deps, lzma_dep],
  install : true
)
