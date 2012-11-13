cmake_minimum_required(VERSION 2.8)
project(ZeroMQ)

include(${PROJECT_SOURCE_DIR}/cmake/Modules/TestZMQVersion.cmake)

# Set some default features required by 0MQ code.
add_definitions(-D_REENTRANT -D_THREAD_SAFE)

if(APPLE)
    set(ZMQ_HAVE_OSX 1)
    add_definitions(-D_DARWIN_C_SOURCE)
elseif(WIN32)
    set(ZMQ_HAVE_WINDOWS 1)
    add_definitions(-DDLL_EXPORT -DFD_SETSIZE=1024 -DWIN32)
elseif(CMAKE_SYSTEM_NAME STREQUAL Linux)
    set(ZMQ_HAVE_LINUX 1)
    if(CMAKE_CXX_COMPILER_ID STREQUAL GNU)
        add_definitions(-D_GNU_SOURCE)
    endif()
else()
    message(FATAL_ERROR
        "I don't know how to configure libzmq for ${CMAKE_SYSTEM_NAME}.")
endif()

# Use the same definitions inside the check_* functions.
set(CMAKE_REQUIRED_DEFINITIONS ${COMPILE_DEFINITIONS})

include(CheckIncludeFile)
check_include_file(ifaddrs.h ZMQ_HAVE_IFADDRS)
check_include_file(sys/eventfd.h ZMQ_HAVE_EVENTFD)
check_include_file(sys/uio.h ZMQ_HAVE_UIO)

include(CheckSymbolExists)
check_symbol_exists(SOCK_CLOEXEC sys/socket.h ZMQ_HAVE_SOCK_CLOEXEC)
check_symbol_exists(SO_KEEPALIVE sys/socket.h ZMQ_HAVE_SO_KEEPALIVE)
check_symbol_exists(TCP_KEEPALIVE netinet/tcp.h ZMQ_HAVE_TCP_KEEPALIVE)
check_symbol_exists(TCP_KEEPCNT netinet/tcp.h ZMQ_HAVE_TCP_KEEPCNT)
check_symbol_exists(TCP_KEEPIDLE netinet/tcp.h ZMQ_HAVE_TCP_KEEPIDLE)
check_symbol_exists(TCP_KEEPINTVL netinet/tcp.h ZMQ_HAVE_TCP_KEEPINTVL)

#-----------------------------------------------------------------------------
# source files

set(cxx-sources
    clock.cpp
    command.cpp
    connect_session.cpp
    ctx.cpp
    decoder.cpp
    device.cpp
    devpoll.cpp
    dist.cpp
    encoder.cpp
    epoll.cpp
    err.cpp
    fq.cpp
    io_object.cpp
    io_thread.cpp
    ip.cpp
    kqueue.cpp
    lb.cpp
    mailbox.cpp
    named_session.cpp
    object.cpp
    options.cpp
    own.cpp
    pair.cpp
    pgm_receiver.cpp
    pgm_sender.cpp
    pgm_socket.cpp
    pipe.cpp
    poll.cpp
    poller_base.cpp
    pub.cpp
    pull.cpp
    push.cpp
    reaper.cpp
    rep.cpp
    req.cpp
    select.cpp
    session.cpp
    signaler.cpp
    socket_base.cpp
    sub.cpp
    swap.cpp
    tcp_connecter.cpp
    tcp_listener.cpp
    tcp_socket.cpp
    thread.cpp
    transient_session.cpp
    trie.cpp
    uuid.cpp
    xpub.cpp
    xrep.cpp
    xreq.cpp
    xsub.cpp
    zmq.cpp
    zmq_connecter.cpp
    zmq_engine.cpp
    zmq_init.cpp
    zmq_listener.cpp
    )

include_directories(include ${PROJECT_BINARY_DIR})

foreach(source ${cxx-sources})
    list(APPEND sources ${PROJECT_SOURCE_DIR}/src/${source})
endforeach()

configure_file(cmake/platform.hpp.in platform.hpp)

#-----------------------------------------------------------------------------
# output

add_library(libzmq SHARED ${sources})
if(WIN32)
    target_link_libraries(libzmq ws2_32.lib rpcrt4.lib)
endif()

set(ZEROMQ_INCLUDE_DIRS ${PROJECT_BINARY_DIR} ${PROJECT_SOURCE_DIR}/include)
set(ZEROMQ_LIBRARIES libzmq)
configure_file(
    cmake/ZeroMQConfig.cmake.in ${CMAKE_BINARY_DIR}/ZeroMQConfig.cmake @ONLY)
