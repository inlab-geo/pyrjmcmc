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

# --- MODIFIED SECTION ---
# This is line 27. Remove explicit COMPONENTS to be less strict.
find_package(Python REQUIRED)
# --- NEW ADDITION BELOW THIS LINE ---
# Manually ensure Python_INCLUDE_DIRS and Python_LIBRARIES are set
# using the values passed from CIBW_CMAKE_OPTIONS (which become cache variables).
# Assuming you pass -DPython_INCLUDE_DIR and -DPython_LIBRARY (singular) in CIBW_CMAKE_OPTIONS.
if (DEFINED Python_INCLUDE_DIR AND NOT "${Python_INCLUDE_DIR}" STREQUAL "")
    set(Python_INCLUDE_DIRS "${Python_INCLUDE_DIR}")
    message(STATUS "Manually set Python_INCLUDE_DIRS from cache: ${Python_INCLUDE_DIRS}")
endif()

if (DEFINED Python_LIBRARY AND NOT "${Python_LIBRARY}" STREQUAL "")
    set(Python_LIBRARIES "${Python_LIBRARY}")
    message(STATUS "Manually set Python_LIBRARIES from cache: ${Python_LIBRARIES}")
endif()
# --- NEW ADDITION ABOVE THIS LINE ---

# Add debugging messages to see what Python was found
if(NOT Python_FOUND)
    message(FATAL_ERROR "CMake could not find Python. This should not happen with CIBW_CMAKE_OPTIONS correctly set.")
else()
    message(STATUS "Python Found (by CMake): YES")
    message(STATUS "  Python Executable: ${Python_EXECUTABLE}")
    message(STATUS "  Python Include Dirs: ${Python_INCLUDE_DIRS}")
    message(STATUS "  Python Libraries: ${Python_LIBRARIES}")
    message(STATUS "  Python Version: ${Python_VERSION}")
endif()
# --- END MODIFIED SECTION ---

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
