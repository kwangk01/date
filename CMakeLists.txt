cmake_minimum_required( VERSION  3.3 )

project( date
         VERSION    2.2.0
         LANGUAGES  CXX
)

set( CMAKE_CXX_STANDARD  14 )
set( CMAKE_CXX_STANDARD_REQUIRED  ON )
set( CMAKE_CXX_EXTENSIONS  OFF )

option( DATE_USE_OS_TZDB "" OFF )
option( DATE_AUTO_DOWNLAOD "" OFF )
option( DATE_MAKE_PORTABLE "" OFF )

# list( APPEND  CMAKE_MODULE_PATH  "${CMAKE_CURRENT_LIST_DIR}/cmake" )

if( NOT  DEFINED  CMAKE_RUNTIME_OUTPUT_DIRECTORY )
  set( CMAKE_RUNTIME_OUTPUT_DIRECTORY  "${CMAKE_BINARY_DIR}/bin" )
endif()
if( NOT  DEFINED  CMAKE_LIBRARY_OUTPUT_DIRECTORY )
  set( CMAKE_LIBRARY_OUTPUT_DIRECTORY  "${CMAKE_BINARY_DIR}/lib" )
endif()
if( NOT  DEFINED  CMAKE_ARCHIVE_OUTPUT_DIRECTORY )
  set( CMAKE_ARCHIVE_OUTPUT_DIRECTORY  "${CMAKE_BINARY_DIR}/lib" )
endif()

enable_testing()

file(
    DOWNLOAD  "http://www.iana.org/time-zones"  "${CMAKE_CURRENT_BINARY_DIR}/time-zones-page.html"
    TLS_VERIFY  ON
    TIMEOUT  10
)
file( STRINGS  "${CMAKE_CURRENT_BINARY_DIR}/time-zones-page.html"  _latest_version_raw  REGEX  "/time-zones/repository/releases/tzdata" )
string( REGEX  MATCH  "/time-zones/repository/releases/tzdata[0-9]+[^\.]"  _latest_version_raw  ${_latest_version_raw} )
string( LENGTH  ${_latest_version_raw}  _latest_version_end )
math( EXPR  _latest_version_begin  "${_latest_version_end} - 5" )
string( SUBSTRING  ${_latest_version_raw}  ${_latest_version_begin}  5  _latest_version )

set( DATE_TZDATA_ARCHIVE  "${CMAKE_BINARY_DIR}/tzdata${_latest_version}.tar.gz"  CACHE  FILEPATH  "" )
get_filename_component( _tzdata_archive_dir  ${DATE_TZDATA_ARCHIVE}  DIRECTORY )
get_filename_component( _tzdata_archive_file  ${DATE_TZDATA_ARCHIVE}  NAME )
get_filename_component( _tzdata_archive_ext  ${DATE_TZDATA_ARCHIVE}  EXT )
set( _tzdata_download_filepath  "${CMAKE_BINARY_DIR}/tzdata${_tzdata_archive_ext}" )

# make sure directory exists
file( MAKE_DIRECTORY  "${_tzdata_archive_dir}/tzdata/" )

# path to where windows zones is placed
set( _windows_zones_download_filepath  "${CMAKE_BINARY_DIR}/tzdata/windowsZones.xml" )

# preprocessor define for path to database install directory
add_definitions( -DINSTALL=${_tzdata_archive_dir} )

if( WIN32 )
    if( NOT  EXISTS  ${_windows_zones_download_filepath} )
        file(
            DOWNLOAD  "https://unicode.org/repos/cldr/trunk/common/supplemental/windowsZones.xml"  ${_windows_zones_download_filepath}
            EXPECTED_HASH  SHA256=61d972860d9f756d7c0e44d81199f18c32bd109822df857b34d07eacc2c3dcf1
            TLS_VERIFY  ON
            TIMEOUT  10
        )
    endif()
endif()

# download and extract the timezone database
if( NOT  EXISTS  "${_tzdata_archive_dir}/tzdata/README" )
    file(
       DOWNLOAD  "https://www.iana.org/time-zones/repository/releases/${_tzdata_archive_file}"  ${_tzdata_download_filepath}
       TLS_VERIFY  ON
       TIMEOUT  10
    )
    execute_process(
        COMMAND  ${CMAKE_COMMAND} -E tar xf ${_tzdata_download_filepath}
        WORKING_DIRECTORY  "${_tzdata_archive_dir}/tzdata/"
    )
endif()

unset( _tzdata_archive_file  CACHE )
unset( _tzdata_archive_ext  CACHE )
unset( _tzdata_download_filepath )



set ( _use_os_tzdb 0 )
set ( _auto_download 0 )
set ( _make_portable 0 )

if ( NOT DATE_MAKE_PORTABLE ) 
    if ( WIN32 )
        set ( _auto_downlaod 1 )
    else()
        set ( _use_os_tzdb 1 )
    endif()
else()
    set ( _make_portable 1 )
endif()

if ( DATE_USE_OS_TZDB ) 
    if ( WIN32 )
        message( FATAL_ERROR "Cannot use OS timezone database option on Windows" )
    else()
        set ( _use_os_tzdb 1 )
    endif()
endif()

if ( DATE_AUTO_DOWNLOAD ) 
    set ( _auto_download 1 )
endif()

add_definitions( -DUSE_OS_TZDB=${_use_os_tzdb} )
add_definitions( -DHAS_REMOTE_API=${_auto_download} )
add_definitions( -DAUTO_DOWNLOAD=${_auto_download} )
add_definitions( -DMAKE_PORTABLE=${_make_portable} )

#add_subdirectory( src )
add_subdirectory( test )
