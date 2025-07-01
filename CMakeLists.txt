cmake_minimum_required(VERSION 2.8.3)
project(serial)

if(APPLE)
    find_library(IOKIT_LIBRARY IOKit)
    find_library(FOUNDATION_LIBRARY Foundation)
endif()

if(UNIX AND NOT APPLE)
    set(rt_LIBRARIES rt)
    set(pthread_LIBRARIES pthread)
endif()

## Sources
set(serial_SRCS
    src/serial.cc
    include/serial/serial.h
    include/serial/v8stdint.h
)

if(APPLE)
    list(APPEND serial_SRCS src/impl/unix.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_osx.cc)
elseif(UNIX)
    list(APPEND serial_SRCS src/impl/unix.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_linux.cc)
else()
    list(APPEND serial_SRCS src/impl/win.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_win.cc)
endif()

add_library(${PROJECT_NAME} ${serial_SRCS})

if(APPLE)
    target_link_libraries(${PROJECT_NAME} ${FOUNDATION_LIBRARY} ${IOKIT_LIBRARY})
elseif(UNIX)
    target_link_libraries(${PROJECT_NAME} rt pthread)
else()
    target_link_libraries(${PROJECT_NAME} setupapi)
endif()

add_executable(serial_example examples/serial_example.cc)
target_link_libraries(serial_example ${PROJECT_NAME})

include_directories(include)

install(TARGETS ${PROJECT_NAME}
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
    RUNTIME DESTINATION bin
)

install(FILES include/serial/serial.h include/serial/v8stdint.h
    DESTINATION include/serial)

if(EXISTS "${CMAKE_SOURCE_DIR}/tests/CMakeLists.txt")
    add_subdirectory(tests)
endif()
