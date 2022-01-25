simple_sources = []

if '#simple' in desired_filters
  simple_sources += files('simple_coder.c')

  if '#simple' in desired_encoders
    simple_sources += files('simple_encoder.c')
  endif

  if '#simple' in desired_decoders
    simple_sources += files('simple_decoder.c')
  endif

  foreach name : simple_filters
    if name in desired_filters
      simple_sources += files(name + '.c')
    endif
  endforeach
endif
