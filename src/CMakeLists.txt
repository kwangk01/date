
# attempt to find curl
find_package( CURL )
# attempt to find a threading library
find_package( Threads )

# get a list of all headers
file( GLOB_RECURSE  _header_files  RELATIVE  ${CMAKE_CURRENT_LIST_DIR}  "${PROJECT_SOURCE_DIR}/include/*.h" )

set ( TZPATH "${_tzdata_archive_dir}/tzdata" )

file ( READ "${TZPATH}/africa" AFRICA NEWLINE )
file ( READ "${TZPATH}/antarctica" ANTARCTICA NEWLINE )
file ( READ "${TZPATH}/asia" ASIA NEWLINE )
file ( READ "${TZPATH}/australasia" AUSTRALASIA NEWLINE )
file ( READ "${TZPATH}/backward" BACKWARD NEWLINE )
file ( READ "${TZPATH}/etcetera" ETCETERA NEWLINE )
file ( READ "${TZPATH}/europe" EUROPE NEWLINE )
file ( READ "${TZPATH}/pacificnew" PACIFICNEW NEWLINE )
file ( READ "${TZPATH}/northamerica" NORTHAMERICA NEWLINE )
file ( READ "${TZPATH}/southamerica" SOUTHAMERICA NEWLINE )
file ( READ "${TZPATH}/systemv" SYSTEMV NEWLINE )
file ( READ "${TZPATH}/leapseconds" LEAPSECONDS NEWLINE )

STRING ( REGEX REPLACE "\n" "\\\\n" AFRICA ${AFRICA} )
STRING ( REGEX REPLACE "#" "//" AFRICA ${AFRICA} )
STRING ( REGEX REPLACE "\"" "\\\\\"" AFRICA ${AFRICA} )

STRING ( REGEX REPLACE "\n" "\\\\n" ANTARCTICA ${ANTARCTICA} )
STRING ( REGEX REPLACE "#" "//" ANTARCTICA ${ANTARCTICA} )
STRING ( REGEX REPLACE "\"" "\\\\\"" ANTARCTICA ${ANTARCTICA} )

STRING ( REGEX REPLACE "\n" "\\\\n" AUSTRALASIA ${AUSTRALASIA} )
STRING ( REGEX REPLACE "#" "//" AUSTRALASIA ${AUSTRALASIA} )
STRING ( REGEX REPLACE "\"" "\\\\\"" AUSTRALASIA ${AUSTRALASIA} )

STRING ( REGEX REPLACE "\n" "\\\\n" ASIA ${ASIA} )
STRING ( REGEX REPLACE "#" "//" ASIA ${ASIA} )
STRING ( REGEX REPLACE "\"" "\\\\\"" ASIA ${ASIA} )

STRING ( REGEX REPLACE "\n" "\\\\n" BACKWARD ${BACKWARD} )
STRING ( REGEX REPLACE "#" "//" BACKWARD ${BACKWARD} )
STRING ( REGEX REPLACE "\"" "\\\\\"" BACKWARD ${BACKWARD} )

STRING ( REGEX REPLACE "\n" "\\\\n" ETCETERA ${ETCETERA} )
STRING ( REGEX REPLACE "#" "//" ETCETERA ${ETCETERA} )
STRING ( REGEX REPLACE "\"" "\\\\\"" ETCETERA ${ETCETERA} )

STRING ( REGEX REPLACE "\n" "\\\\n" EUROPE ${EUROPE} )
STRING ( REGEX REPLACE "#" "//" EUROPE ${EUROPE} )
STRING ( REGEX REPLACE "\"" "\\\\\"" EUROPE ${EUROPE} )

STRING ( REGEX REPLACE "\n" "\\\\n" PACIFICNEW ${PACIFICNEW} )
STRING ( REGEX REPLACE "#" "//" PACIFICNEW ${PACIFICNEW} )
STRING ( REGEX REPLACE "\"" "\\\\\"" PACIFICNEW ${PACIFICNEW} )

STRING ( REGEX REPLACE "\n" "\\\\n" NORTHAMERICA ${NORTHAMERICA} )
STRING ( REGEX REPLACE "#" "//" NORTHAMERICA ${NORTHAMERICA} )
STRING ( REGEX REPLACE "\"" "\\\\\"" NORTHAMERICA ${NORTHAMERICA} )

STRING ( REGEX REPLACE "\n" "\\\\n" SOUTHAMERICA ${SOUTHAMERICA} )
STRING ( REGEX REPLACE "#" "//" SOUTHAMERICA ${SOUTHAMERICA} )
STRING ( REGEX REPLACE "\"" "\\\\\"" SOUTHAMERICA ${SOUTHAMERICA} )

STRING ( REGEX REPLACE "\n" "\\\\n" SYSTEMV ${SYSTEMV} )
STRING ( REGEX REPLACE "#" "//" SYSTEMV ${SYSTEMV} )
STRING ( REGEX REPLACE "\"" "\\\\\"" SYSTEMV ${SYSTEMV} )

STRING ( REGEX REPLACE "\n" "\\\\n" LEAPSECONDS ${LEAPSECONDS} )
STRING ( REGEX REPLACE "#" "//" LEAPSECONDS ${LEAPSECONDS} )
STRING ( REGEX REPLACE "\"" "\\\\\"" LEAPSECONDS ${LEAPSECONDS} )

configure_file( tz.cpp.in tz.cpp @ONLY )

# add the library target
add_library( date
     ${CMAKE_BINARY_DIR}/src/tz.cpp
    ${_header_files}  # for IDEs
)
unset( _header_files  CACHE )

target_include_directories( date
    PUBLIC   "$<INSTALL_INTERFACE:include>"
    PUBLIC   "$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>"
    PRIVATE  "$<$<BOOL:${CURL_FOUND}>:${CURL_INCLUDE_DIRS}>"
)

target_link_libraries( date
    PRIVATE  "$<$<BOOL:${CURL_FOUND}>:${CURL_LIBRARIES}>"
    PRIVATE  "$<$<BOOL:${MINGW}>:${CMAKE_THREAD_LIBS_INIT}>"
)

# if curl was found, let the preprocessor know
target_compile_definitions( date
    PRIVATE  "$<$<BOOL:${CURL_FOUND}>:HAS_REMOTE_API=1>"
)

## Defines:
# ONLY_C_LOCALE
#
# INSTALL
#   HAS_REMOTE_API
# USE_OS_TZDB
# USE_SHELL_API
#
# TARGET_OS_IPHONE
