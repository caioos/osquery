# Copyright (c) 2014-present, Facebook, Inc.
# All rights reserved.
#
# This source code is licensed in accordance with the terms specified in
# the LICENSE file found in the root directory of this source tree.

cmake_minimum_required(VERSION 3.13.3)

include(ProcessorCount)

function(cppcheckMain)
  find_package(cppcheck)
  if(NOT CPPCHECK_FOUND)
    message(STATUS "cppcheck: The executable was not found")
    return()
  endif()

  if(NOT CMAKE_EXPORT_COMPILE_COMMANDS)
    message(STATUS "cppcheck: CMAKE_EXPORT_COMPILE_COMMANDS must be enabled")
    return()
  endif()

  message(STATUS "cppcheck: Enabled with ${CPPCHECK_EXECUTABLE}")

  ProcessorCount(job_count)
  math(EXPR job_count "${job_count} + 1")
  set(multithread_options
    -j ${job_count}
  )

  file(READ "${CMAKE_CURRENT_SOURCE_DIR}/suppressions_list.txt"
    cppcheck_suppressions_list
  )

  string(REPLACE "\n" ", " cppcheck_suppressions_list "${cppcheck_suppressions_list}")

  # We don't care if multithreading disables the unusedFunction check, since that would turn off
  # suppressions (and we don't want to waste time analyzing the source modules)
  add_custom_target(
    cppcheck
    COMMAND "${CMAKE_COMMAND}" -E echo "Suppression list: '${cppcheck_suppressions_list}'"
    COMMAND cppcheck::cppcheck "--suppressions-list=${CMAKE_CURRENT_SOURCE_DIR}/suppressions_list.txt" "--library=${CMAKE_CURRENT_SOURCE_DIR}/libraries/googletest.cfg" ${multithread_options} --enable=all --platform=unix64 "--project=${CMAKE_BINARY_DIR}/compile_commands.json" --quiet
    WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}"
    COMMENT "Running Cppcheck ${CPPCHECK_VERSION_STRING}"
    VERBATIM
  )
endfunction()

cppcheckMain()
