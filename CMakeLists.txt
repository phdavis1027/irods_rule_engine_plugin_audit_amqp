cmake_minimum_required(VERSION 3.5.0 FATAL_ERROR) #CPACK_DEBIAN_<COMPONENT>_PACKAGE_NAME

find_package(IRODS 4.3.0 EXACT REQUIRED)
set(IRODS_PLUGIN_REVISION "0")
set(IRODS_PLUGIN_VERSION "${IRODS_VERSION}.${IRODS_PLUGIN_REVISION}")

set(IRODS_PACKAGE_REVISION "1")

set(PLUGIN irods_rule_engine_plugin-audit_amqp)

include(IrodsCXXCompiler)
set(CMAKE_CXX_STANDARD ${IRODS_CXX_STANDARD})
set(CMAKE_MODULE_LINKER_FLAGS_INIT "-Wl,--enable-new-dtags -Wl,--as-needed -Wl,-z,defs")
set(CMAKE_SHARED_LINKER_FLAGS_INIT "-Wl,--enable-new-dtags -Wl,--as-needed -Wl,-z,defs")
set(CMAKE_MODULE_LINKER_FLAGS_RELEASE_INIT "-Wl,--gc-sections -Wl,-z,combreloc")
set(CMAKE_SHARED_LINKER_FLAGS_RELEASE_INIT "-Wl,--gc-sections -Wl,-z,combreloc")

include(IrodsExternals)

IRODS_MACRO_CHECK_DEPENDENCY_SET_FULLPATH_ADD_TO_IRODS_PACKAGE_DEPENDENCIES_LIST(JANSSON jansson2.7-0)
IRODS_MACRO_CHECK_DEPENDENCY_SET_FULLPATH_ADD_TO_IRODS_PACKAGE_DEPENDENCIES_LIST(QPID_PROTON qpid-proton0.36.0-0)

string(REPLACE ";" ", " ${PLUGIN}_PACKAGE_DEPENDENCIES_STRING "${IRODS_PACKAGE_DEPENDENCIES_LIST}")
unset(IRODS_PACKAGE_DEPENDENCIES_LIST)

project(irods_rule_engine_plugin-audit_amqp
  VERSION "${IRODS_PLUGIN_VERSION}"
  LANGUAGES C CXX)

set(CMAKE_SKIP_BUILD_RPATH OFF)
set(CMAKE_SKIP_INSTALL_RPATH OFF)
set(CMAKE_SKIP_RPATH OFF)
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH ON)
set(CMAKE_BUILD_WITH_INSTALL_RPATH OFF)
set(CMAKE_BUILD_RPATH_USE_ORIGIN ON)

include(${IRODS_TARGETS_PATH})

include(GNUInstallDirs)
include(UseLibCXX)

find_package(Threads REQUIRED)
find_package(OpenSSL REQUIRED COMPONENTS Crypto SSL)
find_package(nlohmann_json "3.6.1" REQUIRED)
find_package(Threads REQUIRED)

add_library(${PLUGIN} MODULE ${CMAKE_SOURCE_DIR}/libirods_rule_engine_plugin-audit_amqp.cpp)
target_compile_options(${PLUGIN} PRIVATE -Wno-write-strings)
target_compile_definitions(${PLUGIN} PRIVATE ${IRODS_COMPILE_DEFINITIONS} ${IRODS_COMPILE_DEFINITIONS_PRIVATE})
target_include_directories(
  ${PLUGIN}
  PRIVATE
  ${IRODS_INCLUDE_DIRS}
  ${IRODS_EXTERNALS_FULLPATH_ARCHIVE}/include
  ${IRODS_EXTERNALS_FULLPATH_BOOST}/include
  ${IRODS_EXTERNALS_FULLPATH_FMT}/include
  ${IRODS_EXTERNALS_FULLPATH_JANSSON}/include
  ${IRODS_EXTERNALS_FULLPATH_QPID_PROTON}/include
  )

target_link_libraries(
  ${PLUGIN}
  PRIVATE
  irods_server
  irods_common
  ${IRODS_EXTERNALS_FULLPATH_JANSSON}/lib/libjansson.so
  ${IRODS_EXTERNALS_FULLPATH_QPID_PROTON}/lib/libqpid-proton.so
  ${IRODS_EXTERNALS_FULLPATH_BOOST}/lib/libboost_regex.so
  ${IRODS_EXTERNALS_FULLPATH_BOOST}/lib/libboost_system.so
  dl
  )

install(TARGETS ${PLUGIN} LIBRARY DESTINATION ${IRODS_PLUGINS_DIRECTORY}/rule_engines)

install(
  FILES
  ${CMAKE_SOURCE_DIR}/packaging/test_plugin_audit_amqp.py
  DESTINATION ${IRODS_HOME_DIRECTORY}/scripts/irods/test
  PERMISSIONS OWNER_READ OWNER_WRITE GROUP_READ WORLD_READ
  )

install(
  FILES
  ${CMAKE_SOURCE_DIR}/packaging/test_audit_plugin.py
  DESTINATION ${IRODS_HOME_DIRECTORY}/scripts/irods/test
  PERMISSIONS OWNER_READ OWNER_WRITE GROUP_READ WORLD_READ
  )

install(
  FILES
  ${CMAKE_SOURCE_DIR}/packaging/audit_plugin_message_worker.py
  DESTINATION ${IRODS_HOME_DIRECTORY}/scripts/irods/test
  PERMISSIONS OWNER_READ OWNER_WRITE GROUP_READ WORLD_READ
  )

install(
  FILES
  ${CMAKE_SOURCE_DIR}/packaging/message_broker_consumer.py
  DESTINATION ${IRODS_HOME_DIRECTORY}/scripts/irods/test
  PERMISSIONS OWNER_READ OWNER_WRITE GROUP_READ WORLD_READ
  )

install(
  FILES
  ${CMAKE_SOURCE_DIR}/packaging/queue_listener.py
  DESTINATION ${IRODS_HOME_DIRECTORY}/scripts/irods/test
  PERMISSIONS OWNER_READ OWNER_WRITE GROUP_READ WORLD_READ
  )

install(
  FILES
  ${CMAKE_SOURCE_DIR}/packaging/run_audit_plugin_test.py
  DESTINATION ${IRODS_HOME_DIRECTORY}/scripts
  PERMISSIONS OWNER_READ OWNER_WRITE GROUP_READ WORLD_READ
  )

install(
  FILES ${CMAKE_SOURCE_DIR}/add_audit_rule_engine_to_rule_engines.py
  DESTINATION ${IRODS_HOME_DIRECTORY}/scripts
  )

include(IrodsCPackCommon)

set(CPACK_INCLUDE_TOPLEVEL_DIRECTORY OFF)
set(CPACK_COMPONENT_INCLUDE_TOPLEVEL_DIRECTORY OFF)
set(CPACK_COMPONENTS_GROUPING IGNORE)
set(CPACK_PACKAGE_VERSION ${IRODS_PLUGIN_VERSION})
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "The integrated Rule-Oriented Data System")

set(CPACK_DEB_COMPONENT_INSTALL OFF)
set(CPACK_DEBIAN_PACKAGE_SHLIBDEPS OFF)
set(CPACK_DEBIAN_PACKAGE_CONTROL_STRICT_PERMISSION ON)
set(CPACK_DEBIAN_PACKAGE_CONTROL_EXTRA "${CMAKE_SOURCE_DIR}/packaging/postinst;")
set(CPACK_DEBIAN_PACKAGE_RELEASE "${IRODS_PACKAGE_REVISION}")
if (IRODS_LINUX_DISTRIBUTION_VERSION_CODENAME)
  set(CPACK_DEBIAN_PACKAGE_RELEASE "${CPACK_DEBIAN_PACKAGE_RELEASE}~${IRODS_LINUX_DISTRIBUTION_VERSION_CODENAME}")
endif()
set(CPACK_DEBIAN_FILE_NAME DEB-DEFAULT)

set(CPACK_RPM_COMPONENT_INSTALL OFF)
set(CPACK_RPM_PACKAGE_LICENSE "BSD-3-Clause")
set(CPACK_RPM_PACKAGE_AUTOREQ 0)
set(CPACK_RPM_PACKAGE_AUTOPROV 0)
set(CPACK_RPM_POST_INSTALL_SCRIPT_FILE "${CMAKE_SOURCE_DIR}/packaging/postinst")
set(CPACK_RPM_PACKAGE_RELEASE "${IRODS_PACKAGE_REVISION}")
set(CPACK_RPM_FILE_NAME RPM-DEFAULT)

set(CPACK_ARCHIVE_COMPONENT_INSTALL OFF)

set(CPACK_DEBIAN_PACKAGE_NAME "irods-rule-engine-plugin-audit-amqp")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "${IRODS_PACKAGE_DEPENDENCIES_STRING}, ${${PLUGIN}_PACKAGE_DEPENDENCIES_STRING}, irods-runtime (= ${IRODS_VERSION}), libc6")

set(CPACK_RPM_PACKAGE_NAME "irods-rule-engine-plugin-audit-amqp")
if (IRODS_LINUX_DISTRIBUTION_NAME STREQUAL "centos" OR IRODS_LINUX_DISTRIBUTION_NAME STREQUAL "centos linux")
  set(CPACK_RPM_PACKAGE_REQUIRES "${IRODS_PACKAGE_DEPENDENCIES_STRING}, ${${PLUGIN}_PACKAGE_DEPENDENCIES_STRING}, irods-runtime = ${IRODS_VERSION}, openssl")
elseif (IRODS_LINUX_DISTRIBUTION_NAME STREQUAL "opensuse")
  set(CPACK_RPM_PACKAGE_REQUIRES "${IRODS_PACKAGE_DEPENDENCIES_STRING}, ${${PLUGIN}_PACKAGE_DEPENDENCIES_STRING}, irods-runtime = ${IRODS_VERSION}, libopenssl1_0_0")
endif()


if (NOT CPACK_GENERATOR)
    set(CPACK_GENERATOR ${IRODS_CPACK_GENERATOR} CACHE STRING "CPack generator to use, e.g. {DEB, RPM, TGZ}." FORCE)
    message(STATUS "Setting unspecified CPACK_GENERATOR to ${CPACK_GENERATOR}. This is the correct setting for normal builds.")
endif()

include(CPack)
