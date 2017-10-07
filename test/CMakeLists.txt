file( GLOB_RECURSE  _pass_sources  RELATIVE  ${CMAKE_CURRENT_LIST_DIR}  "*.pass.cpp" )
list( SORT  _pass_sources )
# create a test for each source file
foreach( _file  ${_pass_sources} )
    # get the filename without an extension for naming the target and test case
    get_filename_component( _base_filename  ${_file}  NAME_WE )
    get_filename_component( _parent_dirname  ${_file}  DIRECTORY )
    while( NOT  "${_parent_dirname}"  STREQUAL  "" )
        set( _test_case_group  ${_parent_dirname} )
        get_filename_component( _parent_dirname  ${_parent_dirname}  DIRECTORY )
    endwhile()
    unset( _parent_dirname  CACHE )
    set( _test_case_name  test_pass__${_test_case_group}__${_base_filename} )
    # create a new executable target
    add_executable( ${_test_case_name}
        EXCLUDE_FROM_ALL
        ${_file}
    )
    # link with our library
    target_link_libraries( ${_test_case_name}
        PRIVATE  date
    )
    set_target_properties( ${_test_case_name}  PROPERTIES
        FOLDER  "Tests"
    )
    # add a build-time test
    add_test(
        NAME  ${_test_case_name}__build
        COMMAND  ${CMAKE_COMMAND} --build ${CMAKE_BINARY_DIR} --target ${_test_case_name}
    )
    # add a runtime test
    add_test(
        NAME  ${_test_case_name}__run
        COMMAND  "$<TARGET_FILE:${_test_case_name}>"
    )
    # the runtime test depends on the build-time test passing
    set_tests_properties( ${_test_case_name}__run  PROPERTIES
        DEPENDS  ${_test_case_name}__build
    )
endforeach()
unset( _pass_sources  CACHE )


file( GLOB_RECURSE  _fail_sources  RELATIVE  ${CMAKE_CURRENT_LIST_DIR}  "*.fail.cpp" )
list( SORT  _fail_sources )
# create a test for each source file
foreach( _file  ${_fail_sources} )
    # get the filename without an extension for naming the target and test case
    get_filename_component( _base_filename  ${_file}  NAME_WE )
    get_filename_component( _parent_dirname  ${_file}  DIRECTORY )
    while( NOT  "${_parent_dirname}"  STREQUAL  "" )
        set( _test_case_group  ${_parent_dirname} )
        get_filename_component( _parent_dirname  ${_parent_dirname}  DIRECTORY )
    endwhile()
    unset( _parent_dirname  CACHE )
    set( _test_case_name  test_fail__${_test_case_group}__${_base_filename} )
    # create a new executable target
    add_executable( ${_test_case_name}
        EXCLUDE_FROM_ALL
        ${_file}
    )
    # link with our library
    target_link_libraries( ${_test_case_name}
        PRIVATE  date
    )
    # add a build-time test
    add_test(
        NAME  ${_test_case_name}
        COMMAND  ${CMAKE_COMMAND} --build ${CMAKE_BINARY_DIR} --target ${_test_case_name}
    )
    # we expect these test cases to all fail
    set_tests_properties( ${_test_case_name}  PROPERTIES
        WILL_FAIL  TRUE
    )
endforeach()
unset( _fail_sources  CACHE )
