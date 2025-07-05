cmake_minimum_required(VERSION 3.12)
project(asio VERSION 1.34.2 LANGUAGES CXX)

# Options to match Autotools configuration
option(ASIO_STANDALONE "Build as standalone (without Boost)" ON)
option(ASIO_SEPARATE_COMPILATION "Enable separate compilation" OFF)
option(ASIO_ENABLE_BOOST_COROUTINE "Enable Boost.Coroutine support" OFF)
option(WITH_OPENSSL "Enable OpenSSL support" OFF)

# Set C++ standard based on available features
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

message(STATUS "cur source dir: ${CMAKE_SOURCE_DIR}")
#add_definitions(-DASIO_CUSTOM_HANDLER_TRACKING="${CMAKE_SOURCE_DIR}/src/examples/cpp11/handler_tracking/custom_tracking.hpp")
add_definitions(-DASIO_ENABLE_HANDLER_TRACKING)

# Platform-specific settings
if(WIN32)
    add_definitions(-D_WIN32_WINNT=0x0601)
    list(APPEND ASIO_LIBS ws2_32 mswsock)
    if(MSVC)
        add_compile_options(/MT)  # 静态链接运行时库（或 /MD 动态链接）
    elseif(MINGW)
        add_compile_options(-mthreads)
        list(APPEND LINK_FLAGS -mthreads)
    endif()
elseif(APPLE)
    # macOS specific settings
elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    message(STATUS "compile at linux")
    add_compile_options(-pthread)
    list(APPEND LINK_FLAGS -pthread)
    list(APPEND ASIO_LIBS rt pthread)
elseif(CMAKE_SYSTEM_NAME STREQUAL "Solaris")
    if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
        add_definitions(-D_PTHREADS)
    else()
        # Assume Sun CC
        add_compile_options(-mt)
    endif()
    list(APPEND ASIO_LIBS socket nsl pthread)
elseif(CMAKE_SYSTEM_NAME MATCHES "BSD")
    add_compile_options(-pthread)
    list(APPEND LINK_FLAGS -pthread)
elseif(CMAKE_SYSTEM_NAME STREQUAL "Haiku")
    list(APPEND ASIO_LIBS network)
endif()

# Boost configuration
if(NOT ASIO_STANDALONE)
    find_package(Boost REQUIRED)
    add_definitions(-DASIO_ENABLE_BOOST -DBOOST_CHRONO_HEADER_ONLY -DBOOST_CHRONO_DONT_PROVIDE_HYBRID_ERROR_HANDLING)

    if(ASIO_ENABLE_BOOST_COROUTINE)
        find_package(Boost COMPONENTS coroutine REQUIRED)
        add_definitions(-DASIO_HAS_BOOST_COROUTINE)
    endif()
endif()

# OpenSSL configuration
if(WITH_OPENSSL)
    find_package(OpenSSL REQUIRED)
    add_definitions(-DASIO_HAS_OPENSSL)
    list(APPEND ASIO_LIBS OpenSSL::SSL OpenSSL::Crypto)
endif()

# Compiler-specific settings
if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    add_compile_options(-ftemplate-depth-256)
endif()

# Standalone mode
if(ASIO_STANDALONE)
    add_definitions(-DASIO_STANDALONE)
endif()

# Separate compilation
if(ASIO_SEPARATE_COMPILATION)
    add_definitions(-DASIO_SEPARATE_COMPILATION)
endif()

# Coroutine support
if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang" AND CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 14)
    if(CMAKE_CXX_STANDARD GREATER_EQUAL 20)
        check_include_file_cxx("coroutine" HAVE_COROUTINE_HEADER)
    else()
        check_include_file_cxx("experimental/coroutine" HAVE_EXPERIMENTAL_COROUTINE_HEADER)
    endif()
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    if(CMAKE_CXX_STANDARD GREATER_EQUAL 20)
        check_include_file_cxx("coroutine" HAVE_COROUTINE_HEADER)
    endif()
endif()

# Include directories
include_directories(include)

# Add subdirectories
add_subdirectory(include)
add_subdirectory(src)

# Install pkg-config file
configure_file(asio.pc.in asio.pc @ONLY)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/asio.pc
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)

# Install license
install(FILES LICENSE_1_0.txt
        DESTINATION ${CMAKE_INSTALL_DATADIR}/asio)