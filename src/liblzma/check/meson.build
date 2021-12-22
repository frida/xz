check_sources = files('check.c')

if 'crc32' in desired_checks
  if tune == 'size'
    check_sources += files('crc32_small.c')
  else
    check_sources += files('crc32_table.c')
    if assembler == 'x86'
      check_sources += files('crc32_x86.S')
    else
      check_sources += files('crc32_fast.c')
    endif
  endif
endif

if 'crc64' in desired_checks
  if tune == 'size'
    check_sources += files('crc64_small.c')
  else
    check_sources += files('crc64_table.c')
    if assembler == 'x86'
      check_sources += files('crc64_x86.S')
    else
      check_sources += files('crc64_fast.c')
    endif
  endif
endif

if 'sha256' in desired_checks and get_option('sha256') == 'internal'
  check_sources += files('sha256.c')
endif
