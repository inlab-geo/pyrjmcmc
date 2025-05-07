cmake_minimum_required(VERSION 3.0)

project(rjmcmc_swig)

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


find_package(PythonLibs REQUIRED)
find_package(SWIG REQUIRED)

include(UseSWIG)

# Define the name of your Python module
set(SWIG_MODULE_NAME _rjmcmc)

# Specify the C source files
set_source_files(RJMCMC_SOURCES src/*.c)

# Specify the SWIG interface file in the python/swig directory
set(SWIG_INTERFACE_FILE ${CMAKE_CURRENT_SOURCE_DIR}/python/swig/rjmcmc.i)

# Generate the wrapper code using SWIG
SWIG_ADD_MODULE(${SWIG_MODULE_NAME} python ${SWIG_INTERFACE_FILE} ${RJMCMC_SOURCES})

# Include directories for your C headers and Python headers
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include ${PYTHON_INCLUDE_DIRS})

# Link the generated wrapper with your C library and Python libraries
target_link_libraries(${SWIG_MODULE_NAME} ${PYTHON_LIBRARIES})

# Install the Python module into the Python site-packages directory within the pyrjmcmc package
install(TARGETS ${SWIG_MODULE_NAME} LIBRARY DESTINATION ${PYTHON_SITE_PACKAGES}/pyrjmcmc)
install(FILES python/__init__.py DESTINATION ${PYTHON_SITE_PACKAGES}/pyrjmcmc)