cmake_minimum_required(VERSION 3.12)

project(result_tests CXX)

include(FetchContent)
FetchContent_Declare(gtest
        GIT_REPOSITORY https://github.com/google/googletest.git
        GIT_TAG 870a6b5563e065b4b9fc3b633c4015ee17d2cbaa
        UPDATE_DISCONNECTED YES)
FetchContent_MakeAvailable(gtest)

option(TEST_INSTALLED_VERSION "Import the library using find_package" NO)
if(TEST_INSTALLED_VERSION)
  find_package(result REQUIRED CONFIG)
else()
  set(result_INCLUDE_WITHOUT_SYSTEM YES CACHE INTERNAL "")
  FetchContent_Declare(result SOURCE_DIR "${CMAKE_CURRENT_LIST_DIR}/..")
  FetchContent_MakeAvailable(result)
endif()

set(options /await /permissive-)
if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  set(options -stdlib=libc++)
endif()

function(add_test)
  cmake_parse_arguments(TEST "" "NAME;NAMESPACE" "" ${ARGN})

  set(target "${TEST_NAMESPACE}_${TEST_NAME}")

  add_executable("${target}" "source/${TEST_NAME}.cpp")

  target_compile_features("${target}" PRIVATE cxx_std_20)
  target_compile_options("${target}" PRIVATE ${options})
  target_include_directories("${target}" PRIVATE "${PROJECT_SOURCE_DIR}/include")
  target_link_libraries("${target}"
          PRIVATE
          DimanNe::result
          gmock
          gtest_main)

  # Call original add_test
  _add_test(NAME "${TEST_NAMESPACE}.${TEST_NAME}" COMMAND "${target}")
endfunction()

add_test(NAMESPACE co NAME coresult)
add_test(NAMESPACE nonco NAME result)
