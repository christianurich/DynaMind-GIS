cmake_minimum_required(VERSION 2.8)
project(GDALModelBuilder)

ADD_DEFINITIONS(-DDYNAMIND_VERSION_CORE="0.6.0")

IF(${CMAKE_CXX_COMPILER_ID} STREQUAL MSVC)
	message(STATUS "Using Visual C compilers")
	set(CMAKE_CXX_FLAGS "/EHsc /bigobj")
	ADD_DEFINITIONS(-D_VARIADIC_MAX=9 -DCGAL_CFG_NO_CPP0X_ISFINITE)
ENDIF()


OPTION(DYNAMIND_UPDATE "update before compile" OFF)
OPTION(WITH_UNIT_TESTS "build unit tests" OFF)
OPTION(WITH_PLUGIN_BASICMODULES "if with BASICMODULES plugin" ON)
OPTION(WITH_PLUGIN_GDALMODULE "if with GDALMODULES plugin" ON)
OPTION(WITH_STANDALONE_EXE "if with standalon executeable" ON)
OPTION(WITH_EXTENSIONS "if with Extensions plugin" ON)


find_package(Git REQUIRED)

set(TOOLBOX DynaMind)


if(WITH_PLUGIN_BASICMODULES)
    set(TOOLBOX ${TOOLBOX} DynaMind-BasicModules)
endif(WITH_PLUGIN_BASICMODULES)

if(WITH_GUI)
    set(TOOLBOX ${TOOLBOX} DynaMind-Gui)
endif(WITH_GUI)

if(WITH_EXTENSIONS)
    set(TOOLBOX ${TOOLBOX} DynaMind-Extensions)
endif(WITH_EXTENSIONS)

if(WITH_PLUGIN_GDALMODULE)
    set(TOOLBOX ${TOOLBOX} DynaMind-GDALModules)
endif(WITH_PLUGIN_GDALMODULE)


if(WITH_STANDALONE_EXE)
    set(TOOLBOX ${TOOLBOX} DynaMind-Executable)
endif(WITH_STANDALONE_EXE)

file(MAKE_DIRECTORY ${GDALModelBuilder_BINARY_DIR}/output)

foreach (tool ${TOOLBOX})
    if(EXISTS ${CMAKE_SOURCE_DIR}/${tool}/src)
        message(STATUS "${tool} submodule already exists")
    else (EXISTS ${CMAKE_SOURCE_DIR}/${tool}/src)
        message(STATUS "preparing ${tool} submodule for you")
        execute_process(COMMAND ${GIT_EXECUTABLE} submodule update --init --recursive ${tool}
                    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})
    endif (EXISTS ${CMAKE_SOURCE_DIR}/${tool}/src)

    if(DYNAMIND_UPDATE)
            execute_process(COMMAND ${GIT_EXECUTABLE} submodule update --recursive ${tool}
                            WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})
    endif(DYNAMIND_UPDATE)
endforeach(tool ${TOOLBOX}) 

SET(DYNAMIND_INCLUDE_DIR ${CMAKE_SOURCE_DIR}/DynaMind/src/core) 
SET(DYNAMIND_INCLUDE_DIR ${DYNAMIND_INCLUDE_DIR} ${CMAKE_SOURCE_DIR}/DynaMind/src/DMToolboxes) 
SET(DYNAMIND_INCLUDE_DIR ${DYNAMIND_INCLUDE_DIR} ${CMAKE_SOURCE_DIR}/DynaMind/src/python-swig) 
SET(DYNAMIND_EXTENSIONS_INCLUDE_DIR ${CMAKE_SOURCE_DIR}/DynaMind-Extensions/src) 
SET(DYNAMIND_TOOLBOX_BUILD_DIR ${GDALModelBuilder_BINARY_DIR}/output)


foreach (tool ${TOOLBOX})
    add_subdirectory(${tool})
endforeach(${tool}) 

