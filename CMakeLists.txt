cmake_minimum_required(VERSION 3.15)
project(pyrjmcmc) # Project name can match the top-level directory

# Needed for original C code to compile
SET(CMAKE_C_FLAGS "-Wno-incompatible-pointer-types")
SET(CMAKE_CXX_FLAGS "-Wno-incompatible-pointer-types")

# Add proper -fPIC and MINGW handling
if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    if (NOT MINGW)
        set(common "-fPIC")
    else()
        set(common "-D_hypot=hypot -Doff64_t=_off64_t")
        if("${CMAKE_SIZEOF_VOID_P}" STREQUAL "8")
            set(common "${common} -DMS_WIN64")
        endif()
    endif()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${common}")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${common}")
endif ()
if (CMAKE_CXX_COMPILER_ID MATCHES Clang)
    set(common "-fPIC")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${common}")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${common}")
endif ()

find_package(Python REQUIRED COMPONENTS Interpreter Development)
find_package(SWIG REQUIRED)

include(UseSWIG)

# Define the name of your Python module
set(SWIG_MODULE_NAME rjmcmc)

# Specify the C source files (including the helper)
file(GLOB RJMCMC_SOURCES
    ${CMAKE_CURRENT_SOURCE_DIR}/rjmcmc/src/*.c
    ${CMAKE_CURRENT_SOURCE_DIR}/rjmcmc/python/swig/rjmcmc_helper.c
)

# Specify the SWIG interface file in the python/swig directory
set(SWIG_INTERFACE_FILE ${CMAKE_CURRENT_SOURCE_DIR}/rjmcmc/python/swig/rjmcmc.i)

# Generate the wrapper code using SWIG
SWIG_ADD_LIBRARY(${SWIG_MODULE_NAME} LANGUAGE python SOURCES ${SWIG_INTERFACE_FILE} ${RJMCMC_SOURCES})

# Include directories for your C headers and Python headers
include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}/rjmcmc/include
    ${CMAKE_CURRENT_SOURCE_DIR}/rjmcmc/python/swig
    ${Python_INCLUDE_DIRS}
)

# Link the generated wrapper with your C library and Python libraries
target_link_libraries(${SWIG_MODULE_NAME} ${Python_LIBRARIES})

# Install the Python module (the compiled extension) DIRECTLY into site-packages
install(TARGETS ${SWIG_MODULE_NAME}
    LIBRARY DESTINATION .)

# Install the __init__.py file into the pyrjmcmc subdirectory of site-packages
install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/rjmcmc/python/__init__.py
    DESTINATION pyrjmcmc)

# Install the SWIG-generated Python wrapper file into the pyrjmcmc subdirectory of site-packages
install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/rjmcmc/python/swig/rjmcmc.py
    DESTINATION pyrjmcmc)
