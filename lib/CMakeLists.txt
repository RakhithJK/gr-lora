# Copyright 2011,2012 Free Software Foundation, Inc.
#
# This file is part of GNU Radio
#
# GNU Radio is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 3, or (at your option)
# any later version.
#
# GNU Radio is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with GNU Radio; see the file COPYING.  If not, write to
# the Free Software Foundation, Inc., 51 Franklin Street,
# Boston, MA 02110-1301, USA.

########################################################################
# Setup library
########################################################################
include(GrPlatform) #define LIB_SUFFIX

include_directories(${Boost_INCLUDE_DIR}
                    ${VOLK_INCLUDE_DIRS}
)
link_directories(${Boost_LIBRARY_DIRS}
)

list(APPEND lora_sources
    demod_impl.cc
    decode_impl.cc
    mod_impl.cc
    encode_impl.cc
)

set(lora_sources "${lora_sources}" PARENT_SCOPE)
if(NOT lora_sources)
	MESSAGE(STATUS "No C++ sources... skipping lib/")
	return()
endif(NOT lora_sources)

list(APPEND lora_libs
    ${Boost_LIBRARIES}
    ${GNURADIO_ALL_LIBRARIES}
    ${VOLK_LIBRARIES}
)

add_library(gnuradio-lora SHARED ${lora_sources})
target_link_libraries(gnuradio-lora ${lora_libs})
set_target_properties(gnuradio-lora PROPERTIES DEFINE_SYMBOL "gnuradio_lora_EXPORTS")

if(APPLE)
    set_target_properties(gnuradio-lora PROPERTIES
        INSTALL_NAME_DIR "${CMAKE_INSTALL_PREFIX}/lib"
    )
endif(APPLE)

########################################################################
# Install built library files
########################################################################
install(TARGETS gnuradio-lora
    LIBRARY DESTINATION lib${LIB_SUFFIX} # .so/.dylib file
    ARCHIVE DESTINATION lib${LIB_SUFFIX} # .lib file
    RUNTIME DESTINATION bin              # .dll file
)

########################################################################
# Build and register unit test
########################################################################
include(GrTest)

include_directories(${CPPUNIT_INCLUDE_DIRS})

list(APPEND test_lora_sources
    ${CMAKE_CURRENT_SOURCE_DIR}/test_lora.cc
    ${CMAKE_CURRENT_SOURCE_DIR}/qa_lora.cc
)

add_executable(test-lora ${test_lora_sources})

target_link_libraries(
  test-lora
  ${lora_libs}
  ${CPPUNIT_LIBRARIES}
  gnuradio-lora
)

GR_ADD_TEST(test_lora test-lora)
