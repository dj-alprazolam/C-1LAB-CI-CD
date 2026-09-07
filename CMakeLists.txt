cmake_minimum_required(VERSION 3.29)

if(NOT PATCH_VERSION)
    set(PATCH_VERSION "1")
endif()

set(PROJECT_VERSION 1.0.${PATCH_VERSION})
project(helloworld VERSION ${PROJECT_VERSION})

set(Boost_USE_STATIC_LIBS OFF)
set(Boost_USE_MULTITHREADED ON)
set(Boost_USE_STATIC_RUNTIME OFF)

if(MINGW)
    set(Boost_COMPILER "-gcc")
endif()

find_package(Boost COMPONENTS unit_test_framework REQUIRED)

message(STATUS "Boost include dirs: ${Boost_INCLUDE_DIRS}")
message(STATUS "Boost library: ${Boost_UNIT_TEST_FRAMEWORK_LIBRARY}")

configure_file(version.h.in version.h)

add_executable(helloworld_cli main.cpp)
add_library(helloworld lib.cpp)
add_executable(test_version test_version.cpp)

set_target_properties(helloworld_cli helloworld test_version PROPERTIES
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED ON
)

target_include_directories(helloworld
    PRIVATE "${CMAKE_BINARY_DIR}"
)

target_include_directories(test_version PRIVATE
    ${Boost_INCLUDE_DIRS}
    ${CMAKE_BINARY_DIR}
)

target_link_libraries(helloworld_cli PRIVATE
    helloworld
)
target_link_libraries(test_version
    ${Boost_UNIT_TEST_FRAMEWORK_LIBRARY}
    helloworld
)

if(MINGW)
    target_compile_definitions(test_version PRIVATE
        BOOST_ALL_DYN_LINK=1
    )
    find_file(BOOST_DLL
        NAMES libboost_unit_test_framework-mt.dll
        PATHS ${BOOST_ROOT}/bin
              ${Boost_LIBRARY_DIR}/../bin
              C:/msys64/mingw64/bin
        NO_DEFAULT_PATH
    )
    
    message(STATUS "Boost DLL found at: ${BOOST_DLL}")

    if(BOOST_DLL)
        add_custom_command(TARGET test_version POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy_if_different
                ${BOOST_DLL}
                $<TARGET_FILE_DIR:test_version>
            COMMENT "Copying Boost DLL to output directory"
        )
    else()
        message(WARNING "Boost DLL not found. Please ensure libboost_unit_test_framework-mt.dll is in your PATH.")
    endif()
endif()

if (MSVC)
    target_compile_options(helloworld_cli PRIVATE /W4)
    target_compile_options(helloworld PRIVATE /W4)
    target_compile_options(test_version PRIVATE /W4)
else ()
    target_compile_options(helloworld_cli PRIVATE -Wall -Wextra -pedantic -Werror)
    target_compile_options(helloworld PRIVATE -Wall -Wextra -pedantic -Werror)
    target_compile_options(test_version PRIVATE -Wall -Wextra -pedantic -Werror)
endif()

install(TARGETS helloworld_cli RUNTIME DESTINATION bin)

set(CPACK_GENERATOR DEB)
set(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")
set(CPACK_PACKAGE_CONTACT example@example.com)
include(CPack)

enable_testing()
add_test(test_version test_version)