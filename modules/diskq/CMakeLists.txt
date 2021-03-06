set(SYSLOG_NG_DISK_BUFFER_SOURCES
    diskq-options.h
    diskq-options.c
    logqueue-disk.c
    logqueue-disk.h
    logqueue-disk-non-reliable.c
    logqueue-disk-non-reliable.h
    logqueue-disk-reliable.c
    logqueue-disk-reliable.h
    qdisk.h
    qdisk.c
)

add_library(syslog-ng-disk-buffer ${SYSLOG_NG_DISK_BUFFER_SOURCES})
target_link_libraries(syslog-ng-disk-buffer PUBLIC syslog-ng)

set(DISK_BUFFER_SOURCES
    diskq.c
    diskq.h
    diskq-parser.c
    diskq-parser.h
    diskq-plugin.c
    ${CMAKE_CURRENT_BINARY_DIR}/diskq-grammar.c
    ${CMAKE_CURRENT_BINARY_DIR}/diskq-grammar.h
)

generate_y_from_ym(modules/diskq/diskq-grammar)

bison_target(DiskBufferGrammar
    ${CMAKE_CURRENT_BINARY_DIR}/diskq-grammar.y
    ${CMAKE_CURRENT_BINARY_DIR}/diskq-grammar.c
    COMPILE_FLAGS ${BISON_FLAGS})

add_library(disk-buffer SHARED ${DISK_BUFFER_SOURCES})
target_link_libraries(disk-buffer PRIVATE syslog-ng-disk-buffer)
target_include_directories(disk-buffer
  PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}
  PRIVATE ${CMAKE_CURRENT_BINARY_DIR}
)
target_link_libraries(disk-buffer PRIVATE syslog-ng)

add_executable(dqtool dqtool.c)
target_link_libraries(dqtool PRIVATE eventlog syslog-ng-disk-buffer)
target_include_directories(dqtool PRIVATE ${CMAKE_CURRENT_BINARY_DIR})

install(TARGETS dqtool RUNTIME DESTINATION bin COMPONENT diskbuffer)
install(TARGETS disk-buffer LIBRARY DESTINATION lib/syslog-ng COMPONENT diskbuffer)

add_test_subdirectory(tests)
