cmake_minimum_required(VERSION 3.10)

project(uSocketsProject)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)

add_library(uSocketsC OBJECT)
add_library(uSockets STATIC)

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

target_sources(uSockets PRIVATE $<TARGET_OBJECTS:uSocketsC>)

option(WITH_EXAMPLES "with examples" OFF)
if(WITH_EXAMPLES)
  add_subdirectory(examples)
endif()
