cmake_minimum_required(VERSION 3.2)
project(CVT C CXX)

set(CMAKE_CXX_FLAGS "-std=c++11 -fPIC")

include_directories("include")

file(GLOB OBJ_SRCS
        src/runtime/*.cc
        src/node/*.cc)

add_library(cvt_objs OBJECT ${OBJ_SRCS})

add_library(cvt SHARED $<TARGET_OBJECTS:cvt_objs>)
set_property(TARGET cvt APPEND PROPERTY LINK_OPTIONS "${CVT_VISIBILITY_FLAGS}")

set(CVT_TEST_LIBRARY_NAME cvt)

# Tests
set(TEST_EXECS "")
file(GLOB TEST_SRCS tests/cpp/*.cc)
find_path(GTEST_INCLUDE_DIR gtest/gtest.h)
find_library(GTEST_LIB gtest "$ENV{GTEST_LIB}")

add_executable(temp temp.cpp)
target_link_libraries(temp cvt ${GTEST_LIB} pthread)

# Create the `cpptest` target if we can find GTest.  If not, we create dummy
# targets that give the user an informative error message.
if (GTEST_INCLUDE_DIR AND GTEST_LIB)
    foreach (__srcpath ${TEST_SRCS})
        get_filename_component(__srcname ${__srcpath} NAME)
        string(REPLACE ".cc" "" __execname ${__srcname})
        add_executable(${__execname} ${__srcpath})
        list(APPEND TEST_EXECS ${__execname})
        target_include_directories(${__execname} SYSTEM PUBLIC ${GTEST_INCLUDE_DIR})
        target_link_libraries(${__execname} PRIVATE ${CVT_TEST_LIBRARY_NAME} ${GTEST_LIB} pthread dl)
        set_target_properties(${__execname} PROPERTIES EXCLUDE_FROM_ALL 1)
        set_target_properties(${__execname} PROPERTIES EXCLUDE_FROM_DEFAULT_BUILD 1)
    endforeach ()
    add_custom_target(cpptest DEPENDS ${TEST_EXECS})
elseif (NOT GTEST_INCLUDE_DIR)
    add_custom_target(cpptest
            COMMAND echo "Missing Google Test headers in include path"
            COMMAND exit 1)
elseif (NOT GTEST_LIB)
    add_custom_target(cpptest
            COMMAND echo "Missing Google Test library"
            COMMAND exit 1)
endif ()
