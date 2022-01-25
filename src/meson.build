subdir('common')
subdir('liblzma')

cli_option = get_option('cli')
if cli_option.auto()
  cli_enabled = not meson.is_subproject() and getopt_supported
else
  cli_enabled = cli_option.enabled()
  if cli_enabled and not getopt_supported
    error('CLI requires getopt() for now (when using the Meson build system)')
  endif
endif
if cli_enabled
  subdir('xz')
endif
