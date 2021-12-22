lzma1_sources = files()

if '#lz' in desired_filters
  if 'lzma1' in desired_filters
    lzma1_sources += files('lzma_encoder_presets.c')
  endif
  if 'lzma1' in desired_encoders
    lzma1_sources += files(
      'lzma_encoder.c',
      'lzma_encoder_optimum_fast.c',
      'lzma_encoder_optimum_normal.c',
    )
    if tune == 'speed'
      lzma1_sources += files('fastpos_table.c')
    endif
  endif
  if 'lzma1' in desired_decoders
    lzma1_sources += files('lzma_decoder.c')
  endif

  if 'lzma2' in desired_encoders
    lzma1_sources += files('lzma2_encoder.c')
  endif
  if 'lzma2' in desired_decoders
    lzma1_sources += files('lzma2_decoder.c')
  endif
endif
