# Copyright (c) 2014, Ford Motor Company
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:
#
# Redistributions of source code must retain the above copyright notice, this
# list of conditions and the following disclaimer.
#
# Redistributions in binary form must reproduce the above copyright notice,
# this list of conditions and the following
# disclaimer in the documentation and/or other materials provided with the
# distribution.
#
# Neither the name of the Ford Motor Company nor the names of its contributors
# may be used to endorse or promote products derived from this software
# without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
# AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
# ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
# LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
# CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
# SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
# INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
# CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
# ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
# POSSIBILITY OF SUCH DAMAGE.

include("./set_3rd_party_paths.cmake")

set(3RD_PARTY_SOURCE_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})
set(3RD_PARTY_BINARY_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})

set (install-3rd_party_logger_var "")

if(NO_REBUILD_3RD_PARTY)
  set(NO_REBUILD_3RD_PARTY_LOGGER ON)
endif()

if(FORCE_3RD_PARTY)
  if(NO_REBUILD_3RD_PARTY)
    message(FATAL_ERROR "Please don't turn on both FORCE_3RD_PARTY and NO_REBUILD_3RD_PARTY at the same time.")
  else()
    set(FORCE_3RD_PARTY_LOGGER ON)
  endif()
endif()

if(ENABLE_LOG)
  # --- libexpat
  add_subdirectory(expat-2.1.0)
  set(EXPAT_LIBS_DIRECTORY ${3RD_PARTY_INSTALL_PREFIX_ARCH}/lib PARENT_SCOPE)

  if(NO_REBUILD_3RD_PARTY_LOGGER)
    message(STATUS "Not rebuilding logger.")
  else()
    set(LOGGER_INSTALL_COMMAND make install)
    if (${USE_DEFAULT_3RD_PARTY_PATH} STREQUAL "true")
      set(LOGGER_INSTALL_COMMAND sudo make install)
    endif()
    if(FORCE_3RD_PARTY_LOGGER)
      message(STATUS "Force to rebuild logger.")

      #build logger
      add_custom_target(3rd_party_logger
        make
        DEPENDS apr-util
        WORKING_DIRECTORY ${3RD_PARTY_BINARY_DIRECTORY}/apache-log4cxx-cmake
      )

      #install logger
      #install either to default place with sudo or non-default plase without sudo.
      #to install with sudo to non-default place use manual installation
      add_custom_target(install-3rd_party_logger
        COMMAND /bin/bash -c \"USE_DEFAULT_3RD_PARTY_PATH=${USE_DEFAULT_3RD_PARTY_PATH}\;
                               if [ \\$$USE_DEFAULT_3RD_PARTY_PATH == "true" ]\; then
                                 sudo -k \;
                               fi\;
                               cd ${3RD_PARTY_BINARY_DIRECTORY}/apr-cmake\;
                               ${LOGGER_INSTALL_COMMAND}\;
                               cd ${3RD_PARTY_BINARY_DIRECTORY}/apr-util-cmake\;
                               ${LOGGER_INSTALL_COMMAND}\;
                               cd ${3RD_PARTY_BINARY_DIRECTORY}/apache-log4cxx-cmake\;
                               ${LOGGER_INSTALL_COMMAND}\"
        DEPENDS 3rd_party_logger
        WORKING_DIRECTORY ${3RD_PARTY_BINARY_DIRECTORY}/apache-log4cxx-cmake
      )
    else()
      #build logger
      add_custom_target(3rd_party_logger
        COMMAND /bin/bash -c \"cd ${CMAKE_CURRENT_SOURCE_DIR} &&
                               grep .commit_hash ${3RD_PARTY_INSTALL_PREFIX_ARCH}/lib/liblog4cxx.so 1>/dev/null 2>&1\;
                               if [ \\$$? == 0 ]\; then
                                 VAR1=\\$$\( readelf -p .commit_hash ${3RD_PARTY_INSTALL_PREFIX_ARCH}/lib/liblog4cxx.so 2>/dev/null\)\;
                                 VAR1=\\$$\(echo \\$$VAR1 | awk '{print \\$$NF}'\)\;
                                 VAR2=-1\;
                                 cd ${CMAKE_CURRENT_SOURCE_DIR}\;
                                 git log . 1>/dev/null 2>&1\;
                                 if [ \\$$? == 0 ]; then
                                     VAR2=\\$$\(git log --pretty=\"format:%H\" -1 ${3RD_PARTY_SOURCE_DIRECTORY}/apache-log4cxx-0.10.0\)\;
                                 fi\;
                                 if [ \\$$VAR1 != \\$$VAR2 ]\; then
                                   echo " Need to rebuild logger. " \;
                                   cd ${3RD_PARTY_BINARY_DIRECTORY}/apache-log4cxx-cmake\;
                                   make\;
                                 else
                                   echo " Logger is actual. " \;
                                 fi\;
                               else
                                 echo " Need to build logger. " \;
                                 cd ${3RD_PARTY_BINARY_DIRECTORY}/apache-log4cxx-cmake\;
                                 make\;
                               fi\"
        DEPENDS apr-util
        WORKING_DIRECTORY ${3RD_PARTY_BINARY_DIRECTORY}/apache-log4cxx-build
      )

      #install logger
      #install either to default place with sudo or non-default plase without sudo.
      #to install with sudo to non-default place use manual installation
      add_custom_target(install-3rd_party_logger
        COMMAND /bin/bash -c \"cd ${CMAKE_CURRENT_SOURCE_DIR} &&
                               grep .commit_hash ${3RD_PARTY_INSTALL_PREFIX_ARCH}/lib/liblog4cxx.so 1>/dev/null 2>&1\;
                               if [ \\$$? == 0 ]\; then
                                 VAR1=\\$$\( readelf -p .commit_hash ${3RD_PARTY_INSTALL_PREFIX_ARCH}/lib/liblog4cxx.so 2>/dev/null\)\;
                                 VAR1=\\$$\(echo \\$$VAR1 | awk '{print \\$$NF}'\)\;
                                 VAR2=-1\;
                                 cd ${CMAKE_CURRENT_SOURCE_DIR}\;
                                 git log . 1>/dev/null 2>&1\;
                                 if [ \\$$? == 0 ]; then
                                     VAR2=\\$$\(git log --pretty=\"format:%H\" -1 ${3RD_PARTY_SOURCE_DIRECTORY}/apache-log4cxx-0.10.0\)\;
                                 fi\;
                                 if [ \\$$VAR1 != \\$$VAR2 ]\; then
                                   USE_DEFAULT_3RD_PARTY_PATH=${USE_DEFAULT_3RD_PARTY_PATH}\;
                                   if [ \\$$USE_DEFAULT_3RD_PARTY_PATH == "true" ]\; then
                                     sudo -k \;
                                   fi\;
                                   cd ${3RD_PARTY_BINARY_DIRECTORY}/apr-cmake\;
                                   ${LOGGER_INSTALL_COMMAND}\;
                                   cd ${3RD_PARTY_BINARY_DIRECTORY}/apr-util-cmake\;
                                   ${LOGGER_INSTALL_COMMAND}\;
                                   cd ${3RD_PARTY_BINARY_DIRECTORY}/apache-log4cxx-cmake\;
                                   ${LOGGER_INSTALL_COMMAND}\;
                                 fi\;
                               else
                                 USE_DEFAULT_3RD_PARTY_PATH=${USE_DEFAULT_3RD_PARTY_PATH}\;
                                 if [ \\$$USE_DEFAULT_3RD_PARTY_PATH == "true" ]\; then
                                   sudo -k \;
                                 fi\;
                                 cd ${3RD_PARTY_BINARY_DIRECTORY}/apr-cmake\;
                                 ${LOGGER_INSTALL_COMMAND}\;
                                 cd ${3RD_PARTY_BINARY_DIRECTORY}/apr-util-cmake\;
                                 ${LOGGER_INSTALL_COMMAND}\;
                                 cd ${3RD_PARTY_BINARY_DIRECTORY}/apache-log4cxx-cmake\;
                                 ${LOGGER_INSTALL_COMMAND}\;
                               fi\"
        DEPENDS 3rd_party_logger
        WORKING_DIRECTORY ${3RD_PARTY_BINARY_DIRECTORY}/apache-log4cxx-build
      )
    endif()

  set(install-3rd_party_logger_var "install-3rd_party_logger")
  endif()

  set(APR_LIBS_DIRECTORY ${3RD_PARTY_INSTALL_PREFIX_ARCH}/lib PARENT_SCOPE)
  set(APR_INCLUDE_DIRECTORY ${3RD_PARTY_INSTALL_PREFIX}/include PARENT_SCOPE)
  set(APR_UTIL_LIBS_DIRECTORY ${3RD_PARTY_INSTALL_PREFIX_ARCH}/lib PARENT_SCOPE)
  set(LOG4CXX_INCLUDE_DIRECTORY ${3RD_PARTY_INSTALL_PREFIX}/include PARENT_SCOPE)
  set(LOG4CXX_LIBS_DIRECTORY ${3RD_PARTY_INSTALL_PREFIX_ARCH}/lib PARENT_SCOPE)

  # --- libapr-1
  add_subdirectory(apr-cmake)

  # --- apr-util
  add_subdirectory(apr-util-cmake)

  # --- log4cxx
  add_subdirectory(apache-log4cxx-cmake)
endif()

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}")
set(CMAKE_PREFIX_PATH ${CMAKE_PREFIX_PATH} "${3RD_PARTY_INSTALL_PREFIX}/etc")
set(ENV{PKG_CONFIG_PATH} $ENV{CMAKE_PREFIX_PATH} "${3RD_PARTY_INSTALL_PREFIX}/etc")
find_package(PkgConfig)
pkg_check_modules(BSON libbson)
message (STATUS "bson installed in " ${BSON_LIBDIR} ", " ${BSON_INCLUDEDIR})

if ((NOT "${BSON_FOUND}") OR ("${BSON_VERSION}" VERSION_LESS "1.2.0"))
  message (STATUS "Building bson required")
  set(BSON_LIB_SOURCE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/bson_c_lib CACHE INTERNAL "Sources of bson library" FORCE)
  set(BSON_LIBS_DIRECTORY ${3RD_PARTY_INSTALL_PREFIX}/lib CACHE INTERNAL "Installation path of bson libraries" FORCE)
  set(BSON_INCLUDE_DIRECTORY ${3RD_PARTY_INSTALL_PREFIX}/include CACHE INTERNAL "Installation path of bson headers" FORCE)
  set(EMHASHMAP_LIBS_DIRECTORY ${3RD_PARTY_INSTALL_PREFIX}/lib CACHE INTERNAL "Installation path of emashmap libraries" FORCE)
  set(EMHASHMAP_INCLUDE_DIRECTORY ${3RD_PARTY_INSTALL_PREFIX}/include CACHE INTERNAL "Installation path of emashmap headers" FORCE)

  set(BSON_INSTALL_COMMAND make install)
  if (${3RD_PARTY_INSTALL_PREFIX} MATCHES "/usr/local")
    set(BSON_INSTALL_COMMAND sudo make install)
  endif()
  include(ExternalProject)
  ExternalProject_Add(libbson
      GIT_REPOSITORY "https://github.com/smartdevicelink/bson_c_lib.git"
      GIT_TAG "master"
      BINARY_DIR ${BSON_LIB_SOURCE_DIRECTORY}
      INSTALL_DIR ${3RD_PARTY_INSTALL_PREFIX}
      DOWNLOAD_DIR ${BSON_LIB_SOURCE_DIRECTORY}
      SOURCE_DIR ${BSON_LIB_SOURCE_DIRECTORY}
      CONFIGURE_COMMAND touch aclocal.m4 configure.ac Makefile.am Makefile.in configure config.h.in libbson.pc.in && ./configure --prefix=${3RD_PARTY_INSTALL_PREFIX}
      BUILD_COMMAND make
      INSTALL_COMMAND ${BSON_INSTALL_COMMAND}
      UPDATE_COMMAND git pull)
else()
  set(BSON_LIBS_DIRECTORY ${BSON_LIBDIR} CACHE INTERNAL "Installation path of bson libraries" FORCE)
  set(EMHASHMAP_LIBS_DIRECTORY ${BSON_LIBDIR} CACHE INTERNAL "Installation path of emashmap libraries" FORCE)
  add_custom_target(
      libbson
      DEPENDS ${BSON_LIBS_DIRECTORY}
      DEPENDS ${EMHASHMAP_LIBS_DIRECTORY}
      )
endif()

set(BOOST_ROOT ${3RD_PARTY_INSTALL_PREFIX})
set(Boost_NO_BOOST_CMAKE ON)
find_package(Boost 1.72.0 COMPONENTS system thread date_time filesystem regex) 
if (NOT ${Boost_FOUND})
  set(BOOST_LIB_SOURCE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/boost_src)
  set(BOOST_LIBS_DIRECTORY ${3RD_PARTY_INSTALL_PREFIX}/lib)
  set(BOOST_INCLUDE_DIR ${3RD_PARTY_INSTALL_PREFIX}/include PARENT_SCOPE)
  SET_PROPERTY(GLOBAL PROPERTY GLOBAL_BOOST_LIBS ${BOOST_LIBS_DIRECTORY})
  message(STATUS "Did not find boost. Downloading and installing boost 1.72")
  set(BOOST_INSTALL_COMMAND ./b2 install)
  if (${3RD_PARTY_INSTALL_PREFIX} MATCHES "/usr/local")
    set(BOOST_INSTALL_COMMAND sudo ./b2 install)
  endif()
  include(ExternalProject)
  ExternalProject_Add(
    Boost
    URL https://dl.bintray.com/boostorg/release/1.72.0/source/boost_1_72_0.tar.gz
    URL_HASH SHA256=c66e88d5786f2ca4dbebb14e06b566fb642a1a6947ad8cc9091f9f445134143f
    DOWNLOAD_DIR ${BOOST_LIB_SOURCE_DIRECTORY}
    SOURCE_DIR ${BOOST_LIB_SOURCE_DIRECTORY}
    CONFIGURE_COMMAND ./bootstrap.sh --with-libraries=system,thread,date_time,filesystem,regex --prefix=${3RD_PARTY_INSTALL_PREFIX}
    BUILD_COMMAND ./b2
    INSTALL_COMMAND ${BOOST_INSTALL_COMMAND} --clean --prefix=${3RD_PARTY_INSTALL_PREFIX} && ${BOOST_INSTALL_COMMAND} --with-system --with-thread --with-date_time --with-filesystem --with-regex --prefix=${3RD_PARTY_INSTALL_PREFIX} > boost_install.log
    INSTALL_DIR ${3RD_PARTY_INSTALL_PREFIX}
    BUILD_IN_SOURCE true
  )
else()
  message (STATUS "Boost installed in " ${Boost_LIBRARY_DIRS} ", " ${Boost_INCLUDE_DIRS})
  set(BOOST_LIBS_DIRECTORY ${Boost_LIBRARY_DIRS})
  set(BOOST_INCLUDE_DIR ${Boost_INCLUDE_DIRS} PARENT_SCOPE)
  SET_PROPERTY(GLOBAL PROPERTY GLOBAL_BOOST_LIBS ${BOOST_LIBS_DIRECTORY})
  add_custom_target(Boost) # empty target, Boost is already installed
endif()

add_custom_target(install-3rd_party
  DEPENDS libbson
  DEPENDS Boost
  DEPENDS ${install-3rd_party_logger_var}
  WORKING_DIRECTORY ${3RD_PARTY_BINARY_DIRECTORY}
)
