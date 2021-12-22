delta_sources = files()

if 'delta' in desired_filters
  delta_sources += files('delta_common.c')

  if 'delta' in desired_encoders
    delta_sources += files('delta_encoder.c')
  endif

  if 'delta' in desired_decoders
    delta_sources += files('delta_decoder.c')
  endif
endif
