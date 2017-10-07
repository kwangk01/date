
# attempt to find curl
find_package( CURL )
# attempt to find a threading library
find_package( Threads )

# get a list of all headers
file( GLOB_RECURSE  _header_files  RELATIVE  ${CMAKE_CURRENT_LIST_DIR}  "${PROJECT_SOURCE_DIR}/include/*.h" )

# add the library target
add_library( date
    tz.cpp
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
# AUTO_DOWNLOAD
# USE_OS_TZDB
# USE_SHELL_API
#
# TARGET_OS_IPHONE
