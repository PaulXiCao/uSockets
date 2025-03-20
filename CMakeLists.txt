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

if(NOT WITH_OPENSSL)
  option(WITH_WOLFSSL "WolfSSL support" ON)
else()
  set(WITH_WOLFSSL
      OFF
      CACHE
        BOOL
        "WolfSSL support (only available when BoringSSL and OpenSLL are OFF)"
        FORCE)
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

elseif(WITH_WOLFSSL)
  message(
    FATAL_ERROR
      "Build errors: 4.2.0 does not even support CMake; 5.7.6 fails to compile: unsupported types."
  )

  include(FetchContent)
  FetchContent_Declare(
    WolfSSL
    GIT_REPOSITORY https://github.com/wolfSSL/wolfssl.git
    # GIT_TAG 239b85c80438bf60d9a5b9e0ebe9ff097a760d0d # Release 5.7.6 (Dec 31,
    # 2024)
    GIT_TAG 48c4b2fedcee4d61b6a76c5ce9e33ab212d6ab4a # Release 4.2.0
                                                     # (2019-10-22)
    GIT_SHALLOW ON
    GIT_SUBMODULES "")
  FetchContent_MakeAvailable(WolfSSL)

  target_compile_definitions(uSocketsC PRIVATE LIBUS_USE_WOLFSSL)
  target_link_libraries(uSocketsC PUBLIC wolfssl)

else()
  target_compile_definitions(uSocketsC PRIVATE LIBUS_NO_SSL)
endif()

option(WITH_EXAMPLES "with examples" OFF)
if(WITH_EXAMPLES)
  add_subdirectory(examples)
endif()
