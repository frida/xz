common_sources = files(
  'common.c',
  'block_util.c',
  'easy_preset.c',
  'filter_common.c',
  'hardware_physmem.c',
  'index.c',
  'stream_flags_common.c',
  'vli_size.c',
)

if threads != 'no'
  common_sources += files('hardware_cputhreads.c')
endif

if desired_encoders.length() > 0
  common_sources += files(
    'alone_encoder.c',
    'block_buffer_encoder.c',
    'block_encoder.c',
    'block_header_encoder.c',
    'easy_buffer_encoder.c',
    'easy_encoder.c',
    'easy_encoder_memusage.c',
    'filter_buffer_encoder.c',
    'filter_encoder.c',
    'filter_flags_encoder.c',
    'index_encoder.c',
    'stream_buffer_encoder.c',
    'stream_encoder.c',
    'stream_flags_encoder.c',
    'vli_encoder.c',
  )
  if threads != 'no'
    common_sources += files('outqueue.c', 'stream_encoder_mt.c')
  endif
endif

if desired_decoders.length() > 0
  common_sources += files(
    'alone_decoder.c',
    'auto_decoder.c',
    'block_buffer_decoder.c',
    'block_decoder.c',
    'block_header_decoder.c',
    'easy_decoder_memusage.c',
    'filter_buffer_decoder.c',
    'filter_decoder.c',
    'filter_flags_decoder.c',
    'index_decoder.c',
    'index_hash.c',
    'stream_buffer_decoder.c',
    'stream_decoder.c',
    'stream_flags_decoder.c',
    'vli_decoder.c',
  )
endif
