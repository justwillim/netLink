cmake_minimum_required(VERSION 2.8)
#cmake_policy(SET CMP0054 NEW)
project(netLink)

set(OUTPUT_DIR out)
set(SOURCE_DIR src)
set(EXAMPLES_DIR ${SOURCE_DIR}/examples)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${OUTPUT_DIR})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${OUTPUT_DIR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${OUTPUT_DIR})



include_directories(PUBLIC include)

aux_source_directory(${SOURCE_DIR} SOURCES)
add_library(shared SHARED ${SOURCES})
add_library(static STATIC ${SOURCES})
set_target_properties(shared PROPERTIES OUTPUT_NAME ${PROJECT_NAME})
set_target_properties(static PROPERTIES OUTPUT_NAME ${PROJECT_NAME})
set_target_properties(shared PROPERTIES DEBUG_POSTFIX -d)
set_target_properties(static PROPERTIES DEBUG_POSTFIX -d)

if(WIN32)
    target_link_libraries(shared ws2_32)
    # https://msdn.microsoft.com/en-us/library/6sehtctf.aspx
    if(${CMAKE_SYSTEM_VERSION} EQUAL "6.1") # Windows 7
        add_definitions(-D_WIN32_WINNT=0x0601 -DWINVER=0x0601)
    elseif(${CMAKE_SYSTEM_VERSION} EQUAL "6.2") # Windows 8
        add_definitions(-D_WIN32_WINNT=0x0602 -DWINVER=0x0602)
    elseif(${CMAKE_SYSTEM_VERSION} EQUAL "6.3") # Windows 8.1
        add_definitions(-D_WIN32_WINNT=0x0603 -DWINVER=0x0603)
    elseif(${CMAKE_SYSTEM_VERSION} EQUAL "10.0") # Windows 10
        add_definitions(-D_WIN32_WINNT=0x0A00 -DWINVER=0x0A00)
    else()
        message(SEND_ERROR "Unknown windows version")
    endif()
endif(WIN32)

if(NOT "${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
    add_definitions(-O3 -std=c++11)
endif()

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
    add_definitions(-stdlib=libc++)
endif()

add_executable(example_tcp ${EXAMPLES_DIR}/tcp.cpp)
add_executable(example_udp ${EXAMPLES_DIR}/udp.cpp)
target_link_libraries(example_tcp shared)
target_link_libraries(example_udp shared)

file(GLOB INCLUDE_HEADERS "include/*.h")

add_custom_target(headers SOURCES ${INCLUDE_HEADERS})

install(FILES ${INCLUDE_HEADERS} DESTINATION include/${CMAKE_PROJECT_NAME})
install(TARGETS shared DESTINATION lib)
install(TARGETS static DESTINATION lib)

set(PROJECT_VERSION_MAJOR 0)
set(PROJECT_VERSION_MINOR 1)
set(PROJECT_VERSION_PATCH 0)
set(PROJECT_VERSION ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH})

include(CMakePackageConfigHelpers)
set(${PROJECT_NAME}_INCLUDE_DIR include/
  CACHE PATH "The directory where netLink headers are installed")
set(${PROJECT_NAME}_CONFIG_CMAKE_PATH lib${LIB_SUFFIX}/cmake/${PROJECT_NAME}
  CACHE PATH "The directory where the CMake files are installed")

set(${PROJECT_NAME}_USE_FILE ${${PROJECT_NAME}_CONFIG_CMAKE_PATH}/Use${PROJECT_NAME}.cmake)

configure_package_config_file(cmake/${PROJECT_NAME}Config.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
  INSTALL_DESTINATION ${${PROJECT_NAME}_CONFIG_CMAKE_PATH}
  PATH_VARS ${PROJECT_NAME}_INCLUDE_DIR ${PROJECT_NAME}_USE_FILE)
write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY SameMajorVersion)

# Install targets

install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
  ${CMAKE_CURRENT_SOURCE_DIR}/cmake/Use${PROJECT_NAME}.cmake
  DESTINATION ${${PROJECT_NAME}_CONFIG_CMAKE_PATH})
