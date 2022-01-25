lz_sources = files()

if '#lz' in desired_filters
  if '#lz' in desired_encoders
    lz_sources += files('lz_encoder.c', 'lz_encoder_mf.c')
  endif

  if '#lz' in desired_decoders
    lz_sources += files('lz_decoder.c')
  endif
endif
