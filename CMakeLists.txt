cmake_minimum_required(VERSION 3.25)

project(uSocketsProject)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)

# By default we use LTO, but Windows does not support it
option(WITH_LTO "Link-time optimization" ON)
option(WITH_ASIO "ASIO event-loop" OFF)

add_library(uSocketsC)

target_sources(
  uSocketsC
  PRIVATE src/bsd.c
          src/context.c
          src/loop.c
          src/quic.c
          src/socket.c
          src/udp.c
          src/crypto/openssl.c
          src/eventing/epoll_kqueue.c
          src/eventing/gcd.c
          src/eventing/libuv.c
          src/io_uring/io_context.c
          src/io_uring/io_loop.c
          src/io_uring/io_socket.c)

target_include_directories(uSocketsC PUBLIC src/)
target_compile_definitions(uSocketsC PRIVATE LIBUS_NO_SSL)
if(WITH_LTO)
  set_target_properties(uSocketsC PROPERTIES INTERPROCEDURAL_OPTIMIZATION true)
endif()

add_library(uSockets)
target_sources(uSockets PRIVATE $<TARGET_OBJECTS:uSocketsC>)

if(WITH_ASIO)
  add_library(uSocketsASIO)

  target_compile_definitions(uSocketsASIO PUBLIC LIBUS_USE_ASIO)
  target_link_libraries(uSocketsASIO PUBLIC pthread stdc++)

  target_include_directories(uSocketsASIO PRIVATE src/)
  target_compile_features(uSocketsASIO PRIVATE cxx_std_14)
  target_sources(uSocketsASIO PRIVATE src/eventing/asio.cpp)
  if(WITH_LTO)
    set_target_properties(uSocketsASIO PROPERTIES INTERPROCEDURAL_OPTIMIZATION
                                                  true)
  endif()

  target_link_libraries(uSockets PRIVATE $<TARGET_OBJECTS:uSocketsASIO>)
endif()

option(WITH_EXAMPLES "with examples" OFF)
if(WITH_EXAMPLES)
  add_subdirectory(examples)
endif()
