cmake_minimum_required(VERSION 3.5)

project(tinyxml2_vendor)

option(FORCE_BUILD_VENDOR_PKG
  "Build TinyXML2 from source, even if system-installed package is available"
  OFF)

if(NOT FORCE_BUILD_VENDOR_PKG)
  list(INSERT CMAKE_MODULE_PATH 0 "${CMAKE_CURRENT_SOURCE_DIR}/cmake/Modules")
  find_package(TinyXML2 REQUIRED)
endif()

if (NOT TinyXML2_FOUND)
  set(POCO_CXX_FLAGS ${CMAKE_CXX_FLAGS})

  if(DEFINED CMAKE_BUILD_TYPE)
    list(APPEND extra_cmake_args -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE})
  endif()
  if(WIN32)
    list(APPEND POCO_CXX_FLAGS "/wd4244 /wd4530 /wd4577")
  else()
    list(APPEND extra_cmake_args "-DCMAKE_C_FLAGS=-Wno-shift-negative-value")
    list(APPEND extra_cmake_args "-DCMAKE_CXX_STANDARD=14")
    if(NOT DEFINED CMAKE_TOOLCHAIN_FILE AND NOT ANDROID)
      list(APPEND extra_cmake_args "-DPOCO_UNBUNDLED:BOOL=ON")
    endif()
  endif()
  if(DEFINED CMAKE_TOOLCHAIN_FILE)
    list(APPEND extra_cmake_args "-DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE}")
    if(ANDROID)
      if(DEFINED ANDROID_ABI)
        list(APPEND extra_cmake_args "-DANDROID_ABI=${ANDROID_ABI}")
      endif()
      if(DEFINED ANDROID_CPP_FEATURES)
        list(APPEND extra_cmake_args "-DANDROID_CPP_FEATURES=${ANDROID_CPP_FEATURES}")
      endif()
      if(DEFINED ANDROID_FUNCTION_LEVEL_LINKING)
        list(APPEND extra_cmake_args
          "-DANDROID_FUNCTION_LEVEL_LINKING=${ANDROID_FUNCTION_LEVEL_LINKING}")
      endif()
      if(DEFINED ANDROID_NATIVE_API_LEVEL)
        list(APPEND extra_cmake_args "-DANDROID_NATIVE_API_LEVEL=${ANDROID_NATIVE_API_LEVEL}")
      endif()
      if(DEFINED ANDROID_NDK)
        list(APPEND extra_cmake_args "-DANDROID_NDK=${ANDROID_NDK}")
      endif()
      if(DEFINED ANDROID_STL)
        list(APPEND extra_cmake_args "-DANDROID_STL=${ANDROID_STL}")
      endif()
      if(DEFINED ANDROID_TOOLCHAIN_NAME)
        list(APPEND extra_cmake_args "-DANDROID_TOOLCHAIN_NAME=${ANDROID_TOOLCHAIN_NAME}")
      endif()
    endif()
  endif()
  list(APPEND extra_cmake_args "-DCMAKE_CXX_FLAGS=${POCO_CXX_FLAGS}")
  list(APPEND extra_cmake_args "-DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}")
  list(APPEND extra_cmake_args "-DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}")
  include(ExternalProject)

  ExternalProject_Add(tinyxml2-7.1.0
    PREFIX tinyxml2-7.1.0
    URL https://github.com/leethomason/tinyxml2/archive/7.1.0.tar.gz
    TIMEOUT 600
    CMAKE_ARGS
      -DBUILD_SHARED_LIBS=ON
      -DCMAKE_INSTALL_PREFIX=${CMAKE_CURRENT_BINARY_DIR}/tinyxml2_install
      ${extra_cmake_args}
      -Wno-dev
  )

  # The external project will install to the build folder, but we'll install that on make install.
  install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/tinyxml2_install/
    DESTINATION ${CMAKE_INSTALL_PREFIX})
else()
  message(STATUS "Found TinyXML2 ${TinyXML_VERSION}")
endif()

configure_file(tinyxml2_vendorConfig.cmake.in
"${PROJECT_BINARY_DIR}/tinyxml2_vendorConfig.cmake" @ONLY)
configure_file(tinyxml2_vendorConfig-version.cmake.in
"${PROJECT_BINARY_DIR}/tinyxml2_vendorConfig-version.cmake" @ONLY)

install(DIRECTORY cmake DESTINATION share/${PROJECT_NAME})

install(FILES
  package.xml
  DESTINATION share/${PROJECT_NAME})

install(FILES
  "${PROJECT_BINARY_DIR}/tinyxml2_vendorConfig.cmake"
  "${PROJECT_BINARY_DIR}/tinyxml2_vendorConfig-version.cmake"
  DESTINATION share/${PROJECT_NAME}/cmake)
