cmake_minimum_required(VERSION 2.8)
project(snappy)

include(CheckIncludeFile)
INCLUDE(CheckFunctionExists)

# version
set(SNAPPY_MAJOR 1)
set(SNAPPY_MINOR 1)
set(SNAPPY_PATCHLEVEL 2)


set(SRCS
    snappy.cc
    snappy-sinksource.cc
    snappy-stubs-internal.cc
    snappy-c.cc)

include_directories(${CMAKE_CURRENT_BINARY_DIR})


CHECK_INCLUDE_FILE(byteswap.h HAVE_BYTESWAP_H)
CHECK_INCLUDE_FILE(stdint.h HAVE_STDINT_H)
CHECK_INCLUDE_FILE(stddef.h HAVE_STDDEF_H)
CHECK_INCLUDE_FILE(sys/byteorder.h HAVE_SYS_BYTEORDER_H)
CHECK_INCLUDE_FILE(sys/endian.h HAVE_SYS_ENDIAN_H)
CHECK_INCLUDE_FILE(sys/uio.h HAVE_SYS_UIO_H)
CHECK_FUNCTION_EXISTS(mmap HAVE_FUNC_MMAP)

# more defines in the unit tests
CHECK_INCLUDE_FILE(sys/mman.h HAVE_SYS_MMAN_H)
CHECK_INCLUDE_FILE(sys/resource.h HAVE_SYS_RESOURCE_H)
CHECK_INCLUDE_FILE(sys/time.h HAVE_SYS_TIME_H)

if (HAVE_STDINT_H)
    add_definitions(-DHAVE_STDINT_H)
    set(ac_cv_have_stdint_h 1)
else()
    set(ac_cv_have_stdint_h 0)
endif()

if (HAVE_STDDEF_H)
    add_definitions(-DHAVE_STDDEF_H)
    set(ac_cv_have_stddef_h 1)
else()
    set(ac_cv_have_stddef_h 0)
endif()

if (HAVE_SYS_UIO_H)
    add_definitions(-DHAVE_SYS_UIO_H)
    set(ac_cv_have_sys_uio_h 1)
else()
    set(ac_cv_have_sys_uio_h 0)
endif()

if (HAVE_SYS_MMAN_H)
    add_definitions(-DHAVE_SYS_MMAN_H)
endif()
if (HAVE_SYS_RESOURCE_H)
    add_definitions(-DHAVE_SYS_RESOURCE_H)
endif()
if (HAVE_SYS_TIME_H)
    add_definitions(-DHAVE_SYS_TIME_H)
endif()
if (HAVE_FUNC_MMAP)
    add_definitions(-DHAVE_FUNC_MMAP)
endif()

if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
    add_definitions(-DHAVE_BUILTIN_EXPECT)
endif()

if (WIN32)
    add_definitions(-DHAVE_WINDOWS_H)
endif()

configure_file(snappy-stubs-public.h.in
    ${CMAKE_CURRENT_BINARY_DIR}/snappy-stubs-public.h
    @ONLY)


add_library(snappy STATIC ${SRCS})


find_package(GTest)

if (GTEST_FOUND)
    enable_testing()
    add_definitions(-DHAVE_GTEST)
    include_directories(${GTEST_INCLUDE_DIRS})

    add_executable(snappy_unittest
        snappy_unittest.cc
        snappy-test.cc)

    target_link_libraries(snappy_unittest
        snappy
        ${GTEST_BOTH_LIBRARIES})

    add_test(all_snappy_tests snappy_unittest)
else()
    message(STATUS "Tests will not be built!")
endif()
