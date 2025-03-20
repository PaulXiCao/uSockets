cmake_minimum_required(VERSION 3.25)

project(uSocketsProject)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)

# By default we use LTO, but Windows does not support it
option(WITH_LTO "Link-time optimization" ON)
option(WITH_ASIO "ASIO event-loop" OFF)

option(WITH_BORINGSSL "BoringSSL support" ON)

if(NOT WITH_BORINGSSL)
  option(WITH_OPENSSL "OpenSSL support" ON)
else()
  set(WITH_OPENSSL
      OFF
      CACHE BOOL "OpenSSL support (only available when BoringSSL is OFF)" FORCE)
endif()

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

if(WITH_BORINGSSL)
  add_subdirectory(boringssl)

  set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
  set(THREADS_PREFER_PTHREAD_FLAG TRUE)
  find_package(Threads REQUIRED)

  target_include_directories(uSocketsC PRIVATE boringssl/include/)
  target_compile_definitions(uSocketsC PRIVATE LIBUS_USE_OPENSSL)
  target_link_libraries(uSockets PRIVATE ssl crypto stdc++ Threads::Threads)
elseif(WITH_OPENSSL)
  find_package(OpenSSL REQUIRED)
  target_compile_definitions(uSocketsC PRIVATE LIBUS_USE_OPENSSL)
  target_link_libraries(uSockets PRIVATE OpenSSL::SSL OpenSSL::Crypto stdc++)
else()
  target_compile_definitions(uSocketsC PRIVATE LIBUS_NO_SSL)
endif()

option(WITH_EXAMPLES "with examples" OFF)
if(WITH_EXAMPLES)
  add_subdirectory(examples)
endif()
