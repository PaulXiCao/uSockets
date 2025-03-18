cmake_minimum_required(VERSION 3.10)

project(uSocketsProject)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)

add_library(uSockets STATIC)

target_sources(
  uSockets
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

target_include_directories(uSockets PUBLIC src/)
target_compile_definitions(uSockets PRIVATE LIBUS_NO_SSL)
