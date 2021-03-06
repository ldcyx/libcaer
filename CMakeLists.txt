# Init CMake (require at least version 2.6)
CMAKE_MINIMUM_REQUIRED(VERSION 2.6)

# General build settings
IF (NOT CMAKE_BUILD_TYPE)
	SET(CMAKE_BUILD_TYPE "Release" CACHE STRING "Possible build types: None Debug Release RelWithDebInfo MinSizeRel")
ENDIF()

IF (NOT ENABLE_SERIALDEV)
	SET(ENABLE_SERIALDEV 0 CACHE BOOL "Enable support for serial port devices using libserialport")
ENDIF()

IF (NOT ENABLE_OPENCV)
	SET(ENABLE_OPENCV 0 CACHE BOOL "Enable support for frame enhancements using OpenCV")
ENDIF()

# Project name and version
PROJECT(libcaer C CXX)
SET(PROJECT_VERSION_MAJOR 2)
SET(PROJECT_VERSION_MINOR 4)
SET(PROJECT_VERSION_PATCH 3)
SET(PROJECT_VERSION ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH})

# Define installation paths
INCLUDE(GNUInstallDirs)

# Custom CMake base module
SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_SOURCE_DIR})
INSTALL(FILES caer-base.cmake DESTINATION ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_DATAROOTDIR}/caer)

# Basic setup for cAER
INCLUDE(caer-base)
CAER_SETUP(FALSE)

# Search for external libraries with pkg-config.
INCLUDE(FindPkgConfig)

# Required: basic USB devices support
PKG_CHECK_MODULES(LIBUSB1 REQUIRED libusb-1.0>=1.0.17)
SET(CAER_PKGCONFIG_REQUIRES_PRIVATE "libusb-1.0 >= 1.0.17")

SET(LIBCAER_INCDIRS ${LIBUSB1_INCLUDE_DIRS})
SET(LIBCAER_LIBDIRS ${LIBUSB1_LIBRARY_DIRS})
SET(LIBCAER_LIBS  ${CAER_BASE_LIBS} ${LIBUSB1_LIBRARIES})

# Optional: serial devices support
IF (ENABLE_SERIALDEV)
	# Require libserialport, minimum 0.1.1 version.
	PKG_CHECK_MODULES(SERIALPORT REQUIRED libserialport>=0.1.1)
	SET(CAER_PKGCONFIG_REQUIRES_PRIVATE "${CAER_PKGCONFIG_REQUIRES_PRIVATE}, libserialport >= 0.1.1")

	SET(LIBCAER_INCDIRS ${LIBCAER_INCDIRS} ${SERIALPORT_INCLUDE_DIRS})
	SET(LIBCAER_LIBDIRS ${LIBCAER_LIBDIRS} ${SERIALPORT_LIBRARY_DIRS})
	SET(LIBCAER_LIBS ${LIBCAER_LIBS} ${SERIALPORT_LIBRARIES})
ENDIF()

# Optional: OpenCV support for frame enhancement
IF (ENABLE_OPENCV)
	# Require new OpenCV 3.1 at least.
	PKG_CHECK_MODULES(OPENCV3 REQUIRED opencv>=3.1.0)
	SET(CAER_PKGCONFIG_REQUIRES_PRIVATE "${CAER_PKGCONFIG_REQUIRES_PRIVATE}, opencv >= 3.1.0")

	SET(LIBCAER_INCDIRS ${LIBCAER_INCDIRS} ${OPENCV3_INCLUDE_DIRS})
	SET(LIBCAER_LIBDIRS ${LIBCAER_LIBDIRS} ${OPENCV3_LIBRARY_DIRS})
	SET(LIBCAER_LIBS ${LIBCAER_LIBS} ${OPENCV3_LIBRARIES})
ENDIF()

# Add local directory to include paths
SET(LIBCAER_INCDIRS ${LIBCAER_INCDIRS} ${CMAKE_SOURCE_DIR}/include/)
SET(LIBCAER_INCDIRS ${LIBCAER_INCDIRS} ${CMAKE_SOURCE_DIR}/includecpp/)

INCLUDE_DIRECTORIES(${LIBCAER_INCDIRS})
LINK_DIRECTORIES(${LIBCAER_LIBDIRS})

# Subdirectories
ADD_SUBDIRECTORY(include)
ADD_SUBDIRECTORY(includecpp)
ADD_SUBDIRECTORY(src)

# Generate pkg-config file
CONFIGURE_FILE(libcaer.pc.in libcaer.pc @ONLY)

INSTALL(FILES ${CMAKE_BINARY_DIR}/libcaer.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)

# Automatic documentation generation support
FIND_PACKAGE(Doxygen)

IF (DOXYGEN_FOUND)
	ADD_SUBDIRECTORY(docs)
ENDIF()

# Support automatic RPM generation
SET(CPACK_PACKAGE_NAME ${PROJECT_NAME})
SET(CPACK_PACKAGE_VERSION ${PROJECT_VERSION})
SET(CPACK_PACKAGE_RELEASE 1)
SET(CPACK_PACKAGE_CONTACT "support@inivation.com")
SET(CPACK_PACKAGE_VENDOR "iniVation AG")
SET(CPACK_PACKAGE_DESCRIPTION "Minimal C library to access, configure and get/send AER data from sensors or to/from neuromorphic processors.")
SET(CPACK_PACKAGE_DESCRIPTION_SUMMARY ${CPACK_PACKAGE_DESCRIPTION})
SET(CPACK_PACKAGING_INSTALL_PREFIX ${CMAKE_INSTALL_PREFIX})
SET(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}-${CPACK_PACKAGE_RELEASE}.${CMAKE_SYSTEM_PROCESSOR}")
SET(CPACK_GENERATOR "RPM" "DEB")
SET(CPACK_RPM_PACKAGE_AUTOREQ 1)
#SET(CPACK_RPM_PACKAGE_REQUIRES "libusbx >= 1.0.17, libserialport >= 0.1.1, opencv >= 3.1.0")
SET(CPACK_DEBIAN_PACKAGE_SHLIBDEPS 1)
#SET(CPACK_DEBIAN_PACKAGE_DEPENDS "libusb-1.0-0 (>= 1.0.17), libserialport0 (>= 0.1.1), libopencv-contrib3.1 (>= 3.1.0)")

INCLUDE(CPack)
