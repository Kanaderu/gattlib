#
#  GattLib - GATT Library
#
#  Copyright (C) 2017  Olivier Martin <olivier@labapart.org>
#
#
#  This program is free software; you can redistribute it and/or modify
#  it under the terms of the GNU General Public License as published by
#  the Free Software Foundation; either version 2 of the License, or
#  (at your option) any later version.
#
#  This program is distributed in the hope that it will be useful,
#  but WITHOUT ANY WARRANTY; without even the implied warranty of
#  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#  GNU General Public License for more details.
#
#  You should have received a copy of the GNU General Public License
#  along with this program; if not, write to the Free Software
#  Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301  USA
#

cmake_minimum_required(VERSION 2.6)

find_package(PkgConfig REQUIRED)

# Added Glib support to ensure we have 'gdbus-codegen'
pkg_search_module(GLIB REQUIRED glib-2.0)

# For DBus support
pkg_search_module(GIO_UNIX REQUIRED gio-unix-2.0)

# Needed by 'bluez5/lib/uuid.c'
pkg_search_module(BLUEZ REQUIRED bluez)

add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/org-bluez-adaptater1.c
                   COMMAND gdbus-codegen --interface-prefix org.bluez.Adapter1. --generate-c-code ${CMAKE_CURRENT_BINARY_DIR}/org-bluez-adaptater1 ${CMAKE_CURRENT_SOURCE_DIR}/interfaces/org.bluez.Adapter1.xml
                   DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/interfaces/org.bluez.Adapter1.xml
                   COMMENT "Generate D-Bus 'org.bluez.Adapter1.xml'"
                   )

add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/org-bluez-device1.c
                   COMMAND gdbus-codegen --interface-prefix org.bluez.Device1. --generate-c-code ${CMAKE_CURRENT_BINARY_DIR}/org-bluez-device1 ${CMAKE_CURRENT_SOURCE_DIR}/interfaces/org.bluez.Device1.xml
                   DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/interfaces/org.bluez.Device1.xml
                   COMMENT "Generate D-Bus 'org.bluez.Device1.xml'"
                   )

add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/org-bluez-gattservice1.c
                   COMMAND gdbus-codegen --interface-prefix org.bluez.GattService1. --generate-c-code ${CMAKE_CURRENT_BINARY_DIR}/org-bluez-gattservice1 ${CMAKE_CURRENT_SOURCE_DIR}/interfaces/org.bluez.GattService1.xml
                   DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/interfaces/org.bluez.GattService1.xml
                   COMMENT "Generate D-Bus 'org.bluez.GattService1.xml'"
                   )

add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/org-bluez-gattcharacteristic1.c
                   COMMAND gdbus-codegen --interface-prefix org.bluez.Characteristic1. --generate-c-code ${CMAKE_CURRENT_BINARY_DIR}/org-bluez-gattcharacteristic1 ${CMAKE_CURRENT_SOURCE_DIR}/interfaces/org.bluez.GattCharacteristic1.xml
                   DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/interfaces/org.bluez.GattCharacteristic1.xml
                   COMMENT "Generate D-Bus 'org.bluez.GattCharacteristic1.xml'"
                   )

add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/org-bluez-gattdescriptor1.c
                   COMMAND gdbus-codegen --interface-prefix org.bluez.Descriptor1. --generate-c-code ${CMAKE_CURRENT_BINARY_DIR}/org-bluez-gattdescriptor1 ${CMAKE_CURRENT_SOURCE_DIR}/interfaces/org.bluez.GattDescriptor1.xml
                   DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/interfaces/org.bluez.GattDescriptor1.xml
                   COMMENT "Generate D-Bus 'org.bluez.GattDescriptor1.xml'"
                   )

# Extract Bluez version
string(REPLACE "." ";" BLUEZ_VERSIONS "${BLUEZ_VERSION}")
list(GET BLUEZ_VERSIONS 0 BLUEZ_VERSION_MAJOR)
list(GET BLUEZ_VERSIONS 1 BLUEZ_VERSION_MINOR)
add_definitions(-DBLUEZ_VERSION_MAJOR=${BLUEZ_VERSION_MAJOR} -DBLUEZ_VERSION_MINOR=${BLUEZ_VERSION_MINOR})
message("Build gattlib for Bluez v${BLUEZ_VERSION_MAJOR}.${BLUEZ_VERSION_MINOR}")

include_directories(. ${CMAKE_CURRENT_BINARY_DIR} ${GIO_UNIX_INCLUDE_DIRS} ${BLUEZ_INCLUDE_DIRS})

set(gattlib_SRCS gattlib.c
                 bluez5/lib/uuid.c
                 ../gattlib_common.c
                 ${CMAKE_CURRENT_BINARY_DIR}/org-bluez-adaptater1.c
                 ${CMAKE_CURRENT_BINARY_DIR}/org-bluez-device1.c
                 ${CMAKE_CURRENT_BINARY_DIR}/org-bluez-gattcharacteristic1.c
                 ${CMAKE_CURRENT_BINARY_DIR}/org-bluez-gattdescriptor1.c
                 ${CMAKE_CURRENT_BINARY_DIR}/org-bluez-gattservice1.c)

set(gattlib_LIBS ${GLIB_LIBRARIES} ${GIO_UNIX_LIBRARIES})

# Gattlib
add_library(gattlib SHARED ${gattlib_SRCS})
target_link_libraries(gattlib ${gattlib_LIBS})

install(TARGETS gattlib LIBRARY DESTINATION lib)