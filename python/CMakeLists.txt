cmake_minimum_required(VERSION 3.2)

# Name of the project (will be the name of the plugin)
project (pyBidder CXX)

if ( NOT DEFINED VANILLA_RTB_ROOT )
message(FATAL_ERROR "Please setup VANILLA_RTB_ROOT a path  to vanilla-rtb sources")
endif()



SET(CMAKE_CXX_STANDARD 14)
SET(CMAKE_CXX_STANDARD_REQUIRED ON)
SET(CMAKE_CXX_EXTENSIONS OFF)

# Boost dependency
set(Boost_USE_STATIC_LIBS   ON)
set(Boost_USE_MULTITHREADED ON)
find_package(Threads)

# Find Python installation 
find_package( PythonLibs 2.7 REQUIRED )
include_directories( ${PYTHON_INCLUDE_DIRS} )

#fixes for boost_python with multiple python versions installed 
if(PYTHONLIBS_FOUND)
   find_package(PythonInterp)
   if(PYTHONINTERP_FOUND)
       set(user_jam "using python : : ${PYTHON_EXECUTABLE} : ${PYTHON_INCLUDE_PATH} \;")
       message(${user_jam})
       message("creating file $ENV{HOME}/user-config.jam")
       file(WRITE "$ENV{HOME}/user-config.jam"  ${user_jam})
   endif()
endif()

# Include BoostLib module
file(GLOB_RECURSE boostlib_cmake_path "${CMAKE_CURRENT_SOURCE_DIR}/node_modules" "BoostLib.cmake")
list(GET boostlib_cmake_path 0 boostlib_cmake_path)
get_filename_component(boostlib_cmake_path "${boostlib_cmake_path}" DIRECTORY)
SET(CMAKE_MODULE_PATH "${CMAKE_MODULE_PATH};${boostlib_cmake_path}")
include(BoostLib)

# Locate/Download Boost (semver)
list(APPEND REQUIRED_BOOST_LIBRARIES "log" "program_options" "system" "serialization" "date_time" "regex" "python")
require_boost_libs("1.66.0" "${REQUIRED_BOOST_LIBRARIES}")

include_directories(${Boost_INCLUDE_DIRS})

# Essential include files to build a node addon,
# you should add this line in every CMake.js based project.
set(JSONV_ROOT  ${VANILLA_RTB_ROOT}/jsonv/include)
set(RTB_ROOT   ${VANILLA_RTB_ROOT}/rtb)
include_directories(${CMAKE_JS_INC} ${VANILLA_RTB_ROOT} ${JSONV_ROOT} ${RTB_ROOT})

# Declare the location of the source files
file(GLOB SOURCE_FILES  RELATIVE_PATH "." "*.cpp")
file(GLOB_RECURSE CRUD_SOURCE_FILES ${VANILLA_RTB_ROOT}/CRUD/service/*.cpp)
list(APPEND SOURCE_FILES "${RTB_ROOT}/core/logger.cpp" "${CRUD_SOURCE_FILES}")


# This line will tell CMake that we're building a shared library
# from the above source files
# named after the project's name
add_library(${PROJECT_NAME} SHARED ${SOURCE_FILES})

if (NOT WIN32 AND NOT APPLE)
set(RT_LIB rt)
endif()

# This line will give our library file a .so extension without any "lib" prefix to work for python interpretor 
set_target_properties(${PROJECT_NAME} PROPERTIES PREFIX "" SUFFIX ".so")

target_link_libraries(${PROJECT_NAME} ${CMAKE_JS_LIB} ${Boost_LIBRARIES} ${PYTHON_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT} ${RT_LIB})

message(${Boost_LIBRARIES})
