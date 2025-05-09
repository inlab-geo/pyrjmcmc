# pyrjmcmc

Python package using a CMAKE build system for the rjmcmc package developed by Rhys Hawkins that is part of the ANU inversion course 
available here [https://github.com/anu-ilab/ANUInversionCourse/tree/main/anu_inversion_course/rjmcmc](https://github.com/anu-ilab/ANUInversionCourse/tree/main/anu_inversion_course/rjmcmc)

Notes:
- Compiling fomr source requires swig and the libraries and header files needed for Python development to be installed
- For the original c code to compile with gcc 15.x or clang 17.x  it is necessary to set CFLAGS='-Wno-incompatible-pointer-types' this is taken care of in the CMakeLists.txt

## Installation from github
PyFM2D can be directly installed from this repository
```
pip install git+https://github.com/inlab-geo/pyrjmcmc
```

## Installation from cloned local repository
```
git clone https://github.com/inlab-geo/pyrjmcmc.git
cd pyrjmcmc
pip install .
```


