# Librspamdserver
SET(LIBSTATSRC		${CMAKE_CURRENT_SOURCE_DIR}/stat_config.c
					${CMAKE_CURRENT_SOURCE_DIR}/stat_process.c)

SET(TOKENIZERSSRC	${CMAKE_CURRENT_SOURCE_DIR}/tokenizers/tokenizers.c
					${CMAKE_CURRENT_SOURCE_DIR}/tokenizers/osb.c)

SET(CLASSIFIERSSRC	${CMAKE_CURRENT_SOURCE_DIR}/classifiers/bayes.c)

SET(BACKENDSSRC 	${CMAKE_CURRENT_SOURCE_DIR}/backends/mmaped_file.c
					${CMAKE_CURRENT_SOURCE_DIR}/backends/sqlite3_backend.c)
SET(CACHESSRC 	${CMAKE_CURRENT_SOURCE_DIR}/learn_cache/sqlite3_cache.c)

IF(ENABLE_HIREDIS MATCHES "ON")
	SET(BACKENDSSRC 	${BACKENDSSRC}
					${CMAKE_CURRENT_SOURCE_DIR}/backends/redis_backend.c)
	SET(CACHESSRC 	${CACHESSRC}
					${CMAKE_CURRENT_SOURCE_DIR}/learn_cache/redis_cache.c)
ENDIF(ENABLE_HIREDIS MATCHES "ON")


SET(RSPAMD_STAT ${LIBSTATSRC}
			${TOKENIZERSSRC}
			${CLASSIFIERSSRC}
			${BACKENDSSRC}
			${CACHESSRC} PARENT_SCOPE)

