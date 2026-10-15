# CAPD-Py

Python library for computer assisted proofs in dynamical systems

## Requirements

The package can be installed via pip. Minimum version of Python supported is 3.9.

As the package is currently built from source CMake and python3-dev package are also needed.

## Installation
 
 If you have the requirements satisfied run the following command
 ``` 
 pip install  uj-capdpy 
 ```

## Building from source

Alternatively you can clone and build the library yourself.
Make sure to include submodules when cloning the library. Use the following command
```
git clone --recurse-submodules https://github.com/tomasz-kapela/CAPD-Py
```
Then make the build directory and build the library like so
```
cmake -B build
cmake --build build
```
After that the shared object file with name starting with ```capdpy``` should appear in the build directory.
This is a package which can be imported into python by writing ```import capdpy```.

## Running examples

Examples of computer assisted proofs from [this repository](https://github.com/CAPDGroup/CAPDexercises) were written in Python using CAPD-Py. They are available in the examples directory. 
If you installed library via pip, examples should work just by writing (substitute example-name with the name of the program you want to run):
```
python example-name.py
```

If you want to use the library built from source the simplest way is to copy the shared object file resulting from the build to examples directory.
Then you can run examples with the command above. 
