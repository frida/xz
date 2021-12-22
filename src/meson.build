subdir('liblzma')

cli_option = get_option('cli')
if cli_option.auto()
  cli_enabled = not meson.is_subproject()
else
  cli_enabled = cli_option.enabled()
endif
if cli_enabled
  subdir('xz')
endif
