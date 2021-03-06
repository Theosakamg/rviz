cmake_minimum_required(VERSION 3.5)

project(rviz_ogre_vendor)

find_package(ament_cmake REQUIRED)

# TODO(wjwood): remove when windows build is fixed
if(DEFINED ENV{RVIZ_BUILD_ON_WINDOWS_OVERRIDE})
  set(RVIZ_BUILD_ON_WINDOWS_OVERRIDE "$ENV{RVIZ_BUILD_ON_WINDOWS_OVERRIDE}")
endif()
if(WIN32 AND NOT DEFINED RVIZ_BUILD_ON_WINDOWS_OVERRIDE)
  message(STATUS "${PROJECT_NAME} is not supported on Windows, skipping...")
  return()
endif()

set(PACKAGE_VERSION "1.0.0")

macro(build_ogre)
  set(extra_cmake_args)
  if(DEFINED CMAKE_BUILD_TYPE)
    list(APPEND extra_cmake_args -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE})
  endif()
  if(NOT WIN32 AND NOT APPLE)
    list(APPEND extra_cmake_args "-DCMAKE_CXX_FLAGS=-std=c++14 -w")
  elseif(APPLE)
    list(APPEND extra_cmake_args "-DCMAKE_CXX_FLAGS=-std=c++14 -stdlib=libc++ -w")
    list(APPEND extra_cmake_args "-DCMAKE_OSX_ARCHITECTURES='x86_64'")
  endif()
  list(APPEND extra_cmake_args "-DCMAKE_C_FLAGS=-w")
  list(APPEND extra_cmake_args "-DOGRE_BUILD_RENDERSYSTEM_GL:BOOL=TRUE")
  list(APPEND extra_cmake_args "-DOGRE_BUILD_RENDERSYSTEM_D3D11:BOOL=OFF")
  list(APPEND extra_cmake_args "-DOGRE_BUILD_RENDERSYSTEM_D3D9:BOOL=OFF")

  # Uncomment this line to enable the GL3PLUS render engine
  # list(APPEND extra_cmake_args "-DOGRE_BUILD_RENDERSYSTEM_GL3PLUS:BOOL=TRUE")

  # These next lines can be used to disable some of the components or plugins for OGRE
  # list(APPEND extra_cmake_args "-DOGRE_BUILD_COMPONENT_BITES:BOOL=OFF")
  # list(APPEND extra_cmake_args "-DOGRE_BUILD_COMPONENT_HLMS:BOOL=ON")
  list(APPEND extra_cmake_args "-DOGRE_CONFIG_ENABLE_ZIP:BOOL=OFF")

  if(WIN32)
    set(patch_exe patch.exe)
  else()
    set(patch_exe patch)
  endif()

  include(ExternalProject)
  ExternalProject_Add(ogre-master-ca665a6
    URL https://github.com/OGRECave/ogre/archive/ca665a69a5ca47608ee0ad304b2596ff866616da.zip
    URL_MD5 a627f447d0013a9cf0889117ac76bc5f
    TIMEOUT 600
    CMAKE_ARGS
      -DOGRE_STATIC:BOOL=ON
      -DOGRE_DEPENDENCIES_DIR=${CMAKE_CURRENT_BINARY_DIR}/ogredeps
      -DOGRE_INSTALL_PDB:BOOL=OFF
      -DOGRE_BUILD_DEPENDENCIES:BOOL=OFF
      -DOGRE_BUILD_TESTS:BOOL=OFF
      -DOGRE_INSTALL_DOCS:BOOL=FALSE
      -DOGRE_BUILD_SAMPLES:BOOL=FALSE
      -DOGRE_INSTALL_SAMPLES:BOOL=FALSE
      -DOGRE_INSTALL_SAMPLES_SOURCE:BOOL=FALSE
      -DOGRE_INSTALL_GENERIC_PLUGINS_CFG:BOOL=TRUE
      -DOGRE_CONFIG_THREADS:STRING=0
      -DOGRE_RESOURCEMANAGER_STRICT:BOOL=ON
      -DCMAKE_INSTALL_PREFIX=${CMAKE_CURRENT_BINARY_DIR}/ogre_install
      -DOGRE_BUILD_LIBS_AS_FRAMEWORKS:BOOL=OFF
      ${extra_cmake_args}
      -Wno-dev
    PATCH_COMMAND
      ${patch_exe} -p1 -N < ${CMAKE_CURRENT_SOURCE_DIR}/ogre-ca665a6-patch.diff
  )

  # The external project will install to the build folder, but we'll install that on make install.
  install(
    DIRECTORY
      ${CMAKE_CURRENT_BINARY_DIR}/ogre_install/
    DESTINATION
      ${CMAKE_INSTALL_PREFIX}/opt/rviz_ogre_vendor
  )
endmacro()

build_ogre()

if(WIN32)
  # TODO(wjwwood): do this for Windows too
  # ament_environment_hooks(env_hook/rviz_ogre_vendor_library_path.bat)
else()
  ament_environment_hooks(env_hook/rviz_ogre_vendor_library_path.sh)
endif()

ament_package(
  CONFIG_EXTRAS "rviz_ogre_vendor-extras.cmake.in"
)
