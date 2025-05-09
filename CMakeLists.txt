#------------------------------------------------------------------------------
#   Global settings
#------------------------------------------------------------------------------

cmake_minimum_required( VERSION 3.5 )

cmake_policy( SET CMP0048 NEW )

#------------------------------------------------------------------------------
#   Define the project
#------------------------------------------------------------------------------

set( EXTPKG_NAME  "crypto"                                                  )
set( EXTPKG_VERS  "1.0.0"                                                   )
set( EXTPKG_DESC  "Simple AES/DES encryption and SHA1/SHA2 hashing library" )

project( ${EXTPKG_NAME} VERSION ${EXTPKG_VERS} LANGUAGES C )
set( PROJECT_DESCRIPTION "${EXTPKG_DESC}" CACHE PATH "Project description" FORCE )

#------------------------------------------------------------------------------
#   Load some handy CMake modules
#------------------------------------------------------------------------------

if( NOT EXISTS "${CMAKE_SOURCE_DIR}/cmake/modules" )
    message( FATAL_ERROR "CMake modules directory not found! ${CMAKE_SOURCE_DIR}/cmake/modules" )
else()

    set( CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/modules" )

    include( Vdump )
    include( Trace )

endif()

#------------------------------------------------------------------------------
#   Parse the build directory and set needed variables.
#------------------------------------------------------------------------------

include( ParseBinaryDir )
ParseBinaryDir()


#------------------------------------------------------------------------------
#   Define package VERSION strings
#------------------------------------------------------------------------------

include( Version )


#------------------------------------------------------------------------------
#   Construct a list of this project's public headers
#------------------------------------------------------------------------------

include( headers.txt )


#------------------------------------------------------------------------------
#   Construct a list of this project's source files
#------------------------------------------------------------------------------

include( sources.txt )


#------------------------------------------------------------------------------
#   Required headers
#------------------------------------------------------------------------------

include( CheckIncludeFile )
include( CheckHeader )

check_header( stdbool.h )       # defines HAVE_STDBOOL_H
check_header( stdint.h )        # defines HAVE_STDINT_H


#------------------------------------------------------------------------------
#   Always generate the platform.h header
#------------------------------------------------------------------------------

if( NOT EXISTS ${CMAKE_SOURCE_DIR}/platform.h.in )
    message( FATAL_ERROR "Unable to find platform.h.in!" )
else()
    configure_file( ${CMAKE_SOURCE_DIR}/platform.h.in
                    ${CMAKE_BINARY_DIR}/platform.h )
endif()


#------------------------------------------------------------------------------
#   Update the INCLUDE directories search order
#------------------------------------------------------------------------------

include( includes.txt )


#------------------------------------------------------------------------------
#   Adjust needed compile flags for this project/package
#------------------------------------------------------------------------------

include( cflags.txt )


#------------------------------------------------------------------------------
#   Set the build architecture      (32 bit or 64- bit)
#------------------------------------------------------------------------------

if( WIN32 )
  # (the toolchain being used (i.e. vstools.cmd 32/64) defines this on Windows)
else()
  if( BITNESS STREQUAL "32" )
    set( CMAKE_C_FLAGS_DEBUG          "${CMAKE_C_FLAGS_DEBUG}          ${m32} -fPIC" )
    set( CMAKE_C_FLAGS_RELWITHDEBINFO "${CMAKE_C_FLAGS_RELWITHDEBINFO} ${m32} -fPIC" )
  else()
    set( CMAKE_C_FLAGS_DEBUG          "${CMAKE_C_FLAGS_DEBUG}          ${m64} -fPIC" )
    set( CMAKE_C_FLAGS_RELWITHDEBINFO "${CMAKE_C_FLAGS_RELWITHDEBINFO} ${m64} -fPIC" )
  endif()
endif()

trace( CMAKE_C_FLAGS_DEBUG )
trace( CMAKE_C_FLAGS_RELWITHDEBINFO )


#------------------------------------------------------------------------------
#   Define the package's build and install targets
#------------------------------------------------------------------------------

add_library( ${FULLNAME}  STATIC  ${${PROJECT_NAME}_SRCS} )

set_target_properties( ${FULLNAME} PROPERTIES
                       POSITION_INDEPENDENT_CODE  TRUE
                       PUBLIC_HEADER              "${PUBLIC_HEADERS}"
                       OUTPUT_NAME                 ${FULLNAME}
                       COMPILE_PDB_NAME            ${FULLNAME} )

install( TARGETS ${FULLNAME}
         PUBLIC_HEADER DESTINATION include
         ARCHIVE       DESTINATION ${LIB_INSTALL_DIR} )


#------------------------------------------------------------------------------
#   Define additional files to be installed
#------------------------------------------------------------------------------

include( extra.txt )


#------------------------------------------------------------------------------
#   Create an uninstall target
#------------------------------------------------------------------------------

#  First, create the cmake script that will do the actual uninstall.

configure_file( "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
                "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake" @ONLY )

#  Now simply define an uninstall target that will run the above script.

add_custom_target( uninstall
                   COMMAND ${CMAKE_COMMAND} -P
                   "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake" )


#------------------------------------------------------------------------------
#   CMake debugging: dump variables at exit
#------------------------------------------------------------------------------

get_filename_component( CURRENT_LIST_FILE "${CMAKE_CURRENT_LIST_FILE}" NAME )
vdump( "${CURRENT_LIST_FILE}" "exit" )
