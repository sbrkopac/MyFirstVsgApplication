cmake_minimum_required(VERSION 3.11)

include(ExternalProject)

project(MyFirstVsgApplication
        VERSION 0.0.0
        DESCRIPTION "Template of how to create a program using VulkanSceneGraph and CMake"
        LANGUAGES CXX
)

# build all examples into the bin directory
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin)

option(USE_EXTERNAL_VSG "Enable this to include VSG without a local install" ON)

if(USE_EXTERNAL_VSG)

    ExternalProject_Add(vsg
        GIT_REPOSITORY https://github.com/vsg-dev/VulkanSceneGraph.git
        SOURCE_DIR "${CMAKE_BINARY_DIR}/vsg-src"
        BINARY_DIR "${CMAKE_BINARY_DIR}/vsg-build"
        CONFIGURE_COMMAND ""
        BUILD_COMMAND ""
        INSTALL_COMMAND ""
        TEST_COMMAND ""
    )
    
    ExternalProject_Get_Property (vsg install_dir)
    
    execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" . WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/vsg-download)
    execute_process(COMMAND ${CMAKE_COMMAND} --build . WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/vsg-download)
    
    add_subdirectory(${CMAKE_BINARY_DIR}/vsg-src
                     ${CMAKE_BINARY_DIR}/vsg-build)
                     
    
endif()

# Change the default build type to Release
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel." FORCE)
endif(NOT CMAKE_BUILD_TYPE)

# find the vsg
if (VULKAN_SDK)
    set(ENV{VULKAN_SDK} ${VULKAN_SDK})
endif()

find_package(vsg REQUIRED)

# find the optional vsgXchange that can be used for reading and range of image and 3d model formats and shader compilation
find_package(vsgXchange QUIET)

# set the use of C++17 globally as all examples require it
set(CMAKE_CXX_STANDARD 17)

# add clobber build target to clear all the non git registered files/directories
add_custom_target(MyFirstVSGApp_clobber
    COMMAND git clean -d -f -x
)

set(SOURCES
    src/main.cpp
)

add_executable(myfirstvsgapplication ${SOURCES})

target_link_libraries(myfirstvsgapplication vsg::vsg)

if (vsgXchange_FOUND)
    target_compile_definitions(myfirstvsgapplication PRIVATE vsgXchange_FOUND)
    target_link_libraries(myfirstvsgapplication vsgXchange::vsgXchange)
endif()

install(TARGETS myfirstvsgapplication
        RUNTIME DESTINATION bin
)

