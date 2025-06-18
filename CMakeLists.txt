cmake_minimum_required(VERSION 3.15)
project(pyrjmcmc) # Project name can match the top-level directory

# Needed for original C code to compile
# Consider making these target-specific instead of global if possible for better CMake practice.
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

# --- macOS Specific Branch ---
if (APPLE)
    message(STATUS "Detected Apple platform (macOS/iOS/etc.). Applying macOS specific settings.")
    # For macOS, especially when building Python extensions, it's often good
    # to explicitly set the deployment target to ensure compatibility with older macOS versions.
    # This value might be overridden by CIBW_CMAKE_OPTIONS if you're using it.
    if (NOT CMAKE_OSX_DEPLOYMENT_TARGET)
        # Set a reasonable default if not already set (e.g., by cibuildwheel)
        set(CMAKE_OSX_DEPLOYMENT_TARGET "10.13" CACHE STRING "Minimum macOS deployment target")
        message(STATUS "Setting CMAKE_OSX_DEPLOYMENT_TARGET to ${CMAKE_OSX_DEPLOYMENT_TARGET}")
    endif()

    # You might also want to ensure universal binaries for Intel/Apple Silicon
    # if not using cibuildwheel's full matrix capabilities.
    # cibuildwheel usually handles this by building multiple wheels.
    # If building manually for universal, uncomment and adjust:
    # if (NOT CMAKE_OSX_ARCHITECTURES)
    #    set(CMAKE_OSX_ARCHITECTURES "arm64;x86_64" CACHE STRING "Architectures for macOS universal binary")
    #    message(STATUS "Setting CMAKE_OSX_ARCHITECTURES to ${CMAKE_OSX_ARCHITECTURES}")
    # endif()

    # Add any other macOS-specific compiler/linker flags here if needed
    # For example, if you had a specific framework to link:
    # list(APPEND Python3_LIBRARIES "-framework CoreFoundation") # Example, not usually needed for Python
endif()
# --- End macOS Specific Branch ---

# --- Python Finding ---
# Using Python3 is generally recommended for modern Python
find_package(Python3 COMPONENTS Interpreter Development REQUIRED)

# Manually ensure Python_INCLUDE_DIRS and Python_LIBRARIES are set
# using the values passed from CIBW_CMAKE_OPTIONS (which become cache variables).
# Assuming you pass -DPython_INCLUDE_DIR and -DPython_LIBRARY (singular) in CIBW_CMAKE_OPTIONS.
# Note: Assigning to Python3_INCLUDE_DIRS and Python3_LIBRARIES for consistency with FindPython3
if (DEFINED Python_INCLUDE_DIR AND NOT "${Python_INCLUDE_DIR}" STREQUAL "")
    set(Python3_INCLUDE_DIRS "${Python_INCLUDE_DIR}")
    message(STATUS "Manually set Python3_INCLUDE_DIRS from cache: ${Python3_INCLUDE_DIRS}")
endif()

if (DEFINED Python_LIBRARY AND NOT "${Python_LIBRARY}" STREQUAL "")
    set(Python3_LIBRARIES "${Python_LIBRARY}")
    message(STATUS "Manually set Python3_LIBRARIES from cache: ${Python3_LIBRARIES}")
endif()

# Add debugging messages to see what Python was found
if(NOT Python3_FOUND) # Use Python3_FOUND
    message(FATAL_ERROR "CMake could not find Python3. This should not happen with CIBW_CMAKE_OPTIONS correctly set.")
else()
    message(STATUS "Python3 Found (by CMake): YES")
    message(STATUS "  Python3 Executable: ${Python3_EXECUTABLE}")
    message(STATUS "  Python3 Include Dirs: ${Python3_INCLUDE_DIRS}")
    message(STATUS "  Python3 Libraries: ${Python3_LIBRARIES}")
    message(STATUS "  Python3 Version: ${Python3_VERSION}")
endif()
# --- End Python Finding ---

find_package(SWIG REQUIRED)

include(UseSWIG)

# Define the name of your Python module
set(SWIG_MODULE_NAME rjmcmc)

# Set the helper source file. This uses a variable for potential future platform-specific helper files.
# Currently, it points to the same file for all platforms.
set(RJMCMC_HELPER_SOURCE_FILE ${CMAKE_CURRENT_SOURCE_DIR}/rjmcmc/python/swig/rjmcmc_helper.c)

# Specify the C source files (including the helper)
file(GLOB RJMCMC_SOURCES
    ${CMAKE_CURRENT_SOURCE_DIR}/rjmcmc/src/*.c
    ${RJMCMC_HELPER_SOURCE_FILE} # Use the variable here
)

# Specify the SWIG interface file in the python/swig directory
set(SWIG_INTERFACE_FILE ${CMAKE_CURRENT_SOURCE_DIR}/rjmcmc/python/swig/rjmcmc.i)

# Generate the wrapper code using SWIG
SWIG_ADD_LIBRARY(${SWIG_MODULE_NAME} LANGUAGE python SOURCES ${SWIG_INTERFACE_FILE} ${RJMCMC_SOURCES})

# Include directories for your C headers and Python headers
include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}/rjmcmc/include
    ${CMAKE_CURRENT_SOURCE_DIR}/rjmcmc/python/swig
    ${Python3_INCLUDE_DIRS} # Use Python3_INCLUDE_DIRS
)

# Link the generated wrapper with your C library and Python libraries
target_link_libraries(${SWIG_MODULE_NAME} ${Python3_LIBRARIES}) # Use Python3_LIBRARIES

# Install the Python module (the compiled extension) DIRECTLY into site-packages
install(TARGETS ${SWIG_MODULE_NAME}
    LIBRARY DESTINATION .)

# Install the __init__.py file into the pyrjmcmc subdirectory of site-packages
install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/rjmcmc/python/__init__.py
    DESTINATION pyrjmcmc)

# Install the SWIG-generated Python wrapper file into the pyrjmcmc subdirectory of site-packages
install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/rjmcmc/python/swig/rjmcmc.py
    DESTINATION pyrjmcmc)