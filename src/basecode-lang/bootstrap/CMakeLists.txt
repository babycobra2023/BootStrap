cmake_minimum_required(VERSION 3.26)
project(bootstrap)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED  ON)

if("${CMAKE_CXX_COMPILER_ID}" MATCHES "Clang")
    set(CMAKE_CXX_FLAGS "-Wall -Wno-unused-private-field -Wno-unknown-pragmas -Wno-inconsistent-missing-override")
    message(STATUS "Using clang flags:${CMAKE_CXX_FLAGS}")
elseif("${CMAKE_CXX_COMPILER_ID}" MATCHES "GNU"}")
    set(CMAKE_CXX_FLAGS "-Wall" CACHE STRING "compile flags" FORCE)
    message(STATUS "Using gcc flags:${CMAKE_CXX_FLAGS}")
else()
    message(STATUS "Unknown compiler:${CMAKE_CXX_COMPILER_ID}"
endif()

#RPATH setup
SET(CMAKE_SKIP_BUILD_RPATH FALSE)
SET(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)
SET(CMAKE_INSTALL_RPATH "usr/local/lib")
SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

enable_testing()
#enhance unit test discovery
set_property(GLOBAL PROPERTY UNIT_TEST_TARGETS "")

macro(add_unit_test target test_name args)
    set_property(GLOBAL PROPERTY UNIT_TEST_TARGETS $(UNIT_TEST_TARGETS) ${CMAKE_CURRENT_BINARY_DIR}/${target})
    add_test(NAME ${test_name} COMMAND ${CMAKE_CURRENT_BINARY_DIR}/${target} ${args})
endmacro(add_unit_test)

#project cmake modules
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${PROJECT_SOURCE_DIR}/cmake")

#dummy target used for file copies
add_custom_target(dummy_target ALL DEPENDS custom-output)
add_custom_command(OUTPUT custom-output COMMAND ${CMAKE_COMMAND} -E echo DEPENDS always_rebuild)
add_custom_command(OUTPUT always_rebuild COMMAND ${CMAKE_COMMAND} -E echo)

#3rd party dependencies
find_package(FMT REQUIRED)

include_directories{
    ${PROJECT_BINARY_DIR}
    ${PROJECT_SOURCE_DIR}
    ${FMT_INCLUDE_DIRS}
}

#if(EXISTS ${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
#    include(${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
#    conan_basic_setup()
#else()
#    message(WARNING "The first conanbuildinfo.cmake doesn't exist,you have to run conan install first")
#endif()

get_property(test_targets GLOBAL PROPERTY UNIT_TEST_TARGETS)
message(STATUS "UNIT_TEST_TARGETS=${test_targets}")
add_custom_target(all_unit_tests ALL DEPENDS ${test_targets})
add_custom_command(
        TARGET all_unit_tests
        COMMENT "Execute all unit tests"
        POST_BUILD
        COMMAND ctest -C $<CONFIGRATION> --output-on-failure
        WORKING_DIRECTORY  ${CMAKE_BINARY_DIR}
)

add_executable(bootstrap main.cpp)

target_link_libraries{
        bootstrap
#        ryu
#        ide
#        emulator
#        hardware
#        core
#        common
#        ${SDL2_LIBRARY}
#        ${Boost_LIBRARIES}
#        ${LOG4CPP_LIBRARIES}
#        ${YAML_CPP_LIBRARIES}
#        ${SDL2_TTF_LIBRARIES}
#        ${SDL2_IMAGE_LIBRARIES}
        fmt::fmt
#        RTTR::Core
}
#add_custom_target(ryu-configured DEPENDS dummy-target ryu)
#add_dependencies(ryu-configured all_unit_tests)
add_custom_target(bootstrap-configured DEPENDS dummy-target bootstrap)
add_dependencies(bootstrap-configured all_unit_tests)
add_custom_command(
        TARGET bootstrap-configured
        COMMAND ${CMAKE_COMMAND} -E echo "placeholder"

#        COMMAND ${CMAKE_COMMAND} -E echo "make_directory:${PROJECT_BINARY_DIR}/logs"
#        COMMAND ${CMAKE_COMMAND} -E make_directory ${PROJECT_BINARY_DIR}/logs
#
#        COMMAND ${CMAKE_COMMAND} -E echo "copy: ${PROJECT_SOURCE_DIR}/ryu.properties"
#        COMMAND ${CMAKE_COMMAND} -E copy ${PROJECT_SOURCE_DIR}/ryu.properties ${PROJECT_BINARY_DIR}
#
#        COMMAND ${CMAKE_COMMAND} -E echo "copy: ${PROJECT_SOURCE_DIR}/machines.yaml"
#        COMMAND ${CMAKE_COMMAND} -E copy ${PROJECT_SOURCE_DIR}/machines.yaml ${PROJECT_BINARY_DIR}

)