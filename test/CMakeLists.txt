cmake_minimum_required(VERSION 3.5 FATAL_ERROR)

if(FBGEMM_BUILD_TESTS AND NOT TARGET gtest)
  #Download Googletest framework from github if
  #GOOGLETEST_SOURCE_DIR is not specified.
  if(NOT DEFINED GOOGLETEST_SOURCE_DIR)
    message(STATUS "Downloading googletest to
      ${FBGEMM_THIRDPARTY_DIR}/googletest
      (define GOOGLETEST_SOURCE_DIR to avoid it)")
    configure_file("${FBGEMM_SOURCE_DIR}/cmake/modules/DownloadGTEST.cmake"
      "${FBGEMM_BINARY_DIR}/googletest-download/CMakeLists.txt")
    execute_process(COMMAND "${CMAKE_COMMAND}" -G "${CMAKE_GENERATOR}" .
      WORKING_DIRECTORY "${FBGEMM_BINARY_DIR}/googletest-download")
    execute_process(COMMAND "${CMAKE_COMMAND}" --build .
      WORKING_DIRECTORY "${FBGEMM_BINARY_DIR}/googletest-download")
    set(GOOGLETEST_SOURCE_DIR "${FBGEMM_THIRDPARTY_DIR}/googletest" CACHE STRING
      "googletest source directory")
  endif()

  #build Googletest framework
  add_subdirectory("${GOOGLETEST_SOURCE_DIR}" "${FBGEMM_BINARY_DIR}/googletest")
endif()

macro(add_gtest TESTNAME)
  add_executable(${TESTNAME} ${ARGN}
    ../bench/BenchUtils.cc QuantizationHelpers.cc TestUtils.cc)
  set_target_properties(${TESTNAME} PROPERTIES
          CXX_STANDARD 11
          CXX_EXTENSIONS NO)
 target_compile_options(${TESTNAME} PRIVATE
   "-m64" "-mavx2" "-mfma" "-masm=intel")
  target_link_libraries(${TESTNAME} gtest gmock gtest_main fbgemm)
  add_dependencies(${TESTNAME} gtest fbgemm)
  add_test(${TESTNAME} ${TESTNAME})
  set_target_properties(${TESTNAME} PROPERTIES FOLDER test)
endmacro()


file(GLOB TEST_LIST "*Test.cc")

foreach(TEST_FILE ${TEST_LIST})
  get_filename_component(TEST_NAME "${TEST_FILE}" NAME_WE)
  get_filename_component(TEST_FILE_ONLY "${TEST_FILE}" NAME)
  add_gtest("${TEST_NAME}"
    "${TEST_FILE_ONLY}")
endforeach()

