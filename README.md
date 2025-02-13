# FTorch

**_A library for coupling (Py)Torch machine learning models to Fortran_**

![GitHub](https://img.shields.io/github/license/Cambridge-ICCS/FTorch)

This repository contains code, utilities, and examples for directly calling PyTorch ML
models from Fortran.


## Contents
- [Description](#description)
- [Installation](#installation)
- [Usage](#usage)
- [Examples](#examples)
- [License](#license)
- [Contributions](#contributions)
- [Authors and Acknowledgment](#authors-and-acknowledgment)
- [Users](#used-by)

## Description

It is desirable to be able to run machine learning (ML) models directly in Fortran.
Such models are often trained in some other language (say Python) using popular frameworks (say PyTorch) and saved.
We want to run inference on this model without having to call a Python executable.
To achieve this we use the existing Torch C++ interface.

This project provides a library enabling a user to directly couple their PyTorch models to Fortran code.
We provide installation instructions for the library as well as instructions and examples for performing coupling.

Project status: This project is currently in pre-release with documentation and code being prepared for a first release.
As such breaking changes may be made.
If you are interested in using this library please get in touch.

_For a similar approach to calling TensorFlow models from Fortran please see [Fortran-TF-lib](https://github.com/Cambridge-ICCS/fortran-tf-lib)._

## Installation

### Dependencies

To install the library requires the following to be installed on the system:

* cmake >= 3.1
* [libtorch](https://pytorch.org/cppdocs/installing.html)<sup>*</sup> or [PyTorch](https://pytorch.org/)
* Fortran, C++ (must fully support C++17), and C compilers

<sup>*</sup> _The minimal example provided downloads the CPU-only Linux Nightly binary. [Alternative versions](https://pytorch.org/get-started/locally/) may be required._

#### Windows Support

If building in a windows environment then you can either:

1) Use [Windows Subsystem for Linux](https://learn.microsoft.com/en-us/windows/wsl/) (WSL)  
   In this case the build process is the same as for Linux environment.
2) Use Visual Studio and the Intel Fortran Compiler  
   In this case you must install [Visual Studio](https://visualstudio.microsoft.com/) followed by [Intel OneAPI Base and HPC toolkit](https://www.intel.com/content/www/us/en/developer/tools/oneapi/base-toolkit-download.html) (ensure that the Intel Fortran compiler and VS integration is selected in the latter). 

Note that libTorch is not supported for the GNU Fortran compiler with MinGW.

#### Apple Silicon Support

At the time of writing, libtorch is currently only officially available for x86 architectures (according to https://pytorch.org/). However, the version of PyTorch provided by `pip install` provides an ARM binary for libtorch which works on Apple Silicon. Therefore you should `pip install` PyTorch in this situation and follow the guidance below on locating Torch for cmake.

### Library installation

To build and install the library:

1. Navigate to the location in which you wish to install the source and run:  
    ```bash
    git clone git@github.com:Cambridge-ICCS/FTorch.git
    ```
    to clone via ssh, or  
    ```bash
    git clone https://github.com/Cambridge-ICCS/FTorch.git
    ```
    to clone via https.  
2. Navigate into the library directory by running:  
    ```bash
    cd FTorch/src/
    ```
3. Create a `build` directory and execute cmake from within it using the relevant options (see below):  
    ```bash
    mkdir build
    cd build
    cmake .. -DCMAKE_BUILD_TYPE=Release
    ```
    If building on Windows you will need to add `-G "NMake Makefiles"` to the `cmake` command. Also you will need to load the
    intel fortran compilers using `setvars.bat` ([see Intel docs](https://www.intel.com/content/www/us/en/docs/oneapi/programming-guide/2023-2/use-the-setvars-script-with-windows.html)) which is found in the Intel compiler install directory.
    
    The following table of CMake options are available to be passed as arguments to `cmake` through `-D<Option>=<Value>`.  
    It is likely that you will need to provide at least `CMAKE_PREFIX_PATH`.  
    | Option                                                                                            | Value                        | Description                                                   |
    | ------------------------------------------------------------------------------------------------- | ---------------------------- | --------------------------------------------------------------|
    | [`CMAKE_Fortran_COMPILER`](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_COMPILER.html) | `ifort` / `gfortran`         | Specify a Fortran compiler to build the library with. This should match the Fortran compiler you're using to build the code you are calling this library from.<sup>1</sup>        |
    | [`CMAKE_C_COMPILER`](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_COMPILER.html)       | `icc` / `gcc`                | Specify a C compiler to build the library with.<sup>1</sup>                |
    | [`CMAKE_CXX_COMPILER`](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_COMPILER.html)     | `icpc` / `g++`               | Specify a C++ compiler to build the library with.<sup>1</sup>              |
    | [`CMAKE_PREFIX_PATH`](https://cmake.org/cmake/help/latest/variable/CMAKE_PREFIX_PATH.html)        | `</path/to/libTorch/>`       | Location of Torch installation<sup>2</sup>                    |
    | [`CMAKE_INSTALL_PREFIX`](https://cmake.org/cmake/help/latest/variable/CMAKE_INSTALL_PREFIX.html)  | `</path/to/install/lib/at/>` | Location at which the library files should be installed. By default this is `/usr/local` |
    | [`CMAKE_BUILD_TYPE`](https://cmake.org/cmake/help/latest/variable/CMAKE_BUILD_TYPE.html)          | `Release` / `Debug`          | Specifies build type. The default is `Debug`, use `Release` for production code|
    | `ENABLE_CUDA`                                                                                     | `TRUE` / `FALSE`             | Specifies whether to check for and enable CUDA<sup>2</sup> |

    <sup>1</sup> _On Windows this may need to be the full path to the compiler if CMake cannot locate it by default._  

    <sup>2</sup> _The path to the Torch installation needs to allow cmake to locate the relevant Torch cmake files.  
          If Torch has been [installed as libtorch](https://pytorch.org/cppdocs/installing.html)
          then this should be the absolute path to the unzipped libtorch distribution.
          If Torch has been installed as PyTorch in a python [venv (virtual environment)](https://docs.python.org/3/library/venv.html),
          e.g. with `pip install torch`, then this should be `</path/to/venv/>lib/python<3.xx>/site-packages/torch/`.  
		  You can find the location of your torch install by importing torch from your python environment (`import torch`) and running `print(torch.__file__)`_
	  
4. Make and install the library to the desired location with either:
	```bash
    cmake --build . --target install
    ```
    or, if you want to separate these steps:
    ```bash
    cmake --build .
    cmake --install .
    ```

    Note: If you are using cmake<3.15 then you will need to build and install separately
    using the make system specific commands.
    For example, if using `make` on UNIX this would be:
    ```bash
    make
    make install
    ```

    Installation will place the following directories at the install location:  
    * `CMAKE_INSTALL_PREFIX/include/` - contains header and mod files
    * `CMAKE_INSTALL_PREFIX/lib/` - contains `cmake` directory and `.so` files  
    _Note: depending on your system and architecture `lib` may be `lib64`, and 
    you may have `.dll` files or similar._  
	_Note: In a Windows environment this will require administrator privileges for the default install location._


## Usage

In order to use FTorch users will typically need to follow these steps:

1. Save a PyTorch model as [TorchScript](https://pytorch.org/docs/stable/jit.html).
2. Write Fortran using the FTorch bindings to use the model from within Fortran.
3. Build and compile the code, linking against the FTorch library


### 1. Saving the model as TorchScript

The trained PyTorch model needs to be exported to [TorchScript](https://pytorch.org/docs/stable/jit.html).
This can be done from within your code using the [`jit.script`](https://pytorch.org/docs/stable/generated/torch.jit.script.html#torch.jit.script) or [`jit.trace`](https://pytorch.org/docs/stable/generated/torch.jit.trace.html#torch.jit.trace) functionalities from within python.

If you are not familiar with these we provide a tool [`pt2ts.py`](utils/pt2ts.py) as part of this distribution which contains an easily adaptable script to save your PyTorch model as TorchScript.


### 2. Using the model from Fortran

To use the trained Torch model from within Fortran we need to import the `ftorch` module and use the binding routines to load the model, convert the data, and run inference.

A very simple example is given below.
For more detailed documentation please consult the API documentation, source code, and examples.

This minimal snippet loads a saved Torch model, creates an input consisting of a `10x10` matrix of ones, and runs the model to infer output.  
This is illustrative only, and we recommend following the [examples](examples/) before writing your own code to explore more features.

```fortran
! Import library for interfacing with PyTorch
use ftorch

implicit none

! Generate an object to hold the Torch model
type(torch_module) :: model

! Set up types of input and output data
integer, parameter :: n_inputs = 1
type(torch_tensor), dimension(n_inputs) :: model_input_arr
type(torch_tensor) :: model_output

! Set up the model input and output as Fortran arrays
real, dimension(10,10), target  :: input
real, dimension(5), target   :: output

! Set up number of dimensions of input tensor and axis order
integer, parameter :: in_dims = 2
integer :: in_layout(in_dims) = [1,2]
integer, parameter :: out_dims = 1
integer :: out_layout(out_dims) = [1]

! Initialise the Torch model to be used
model = torch_module_load("/path/to/saved/model.pt")

! Initialise the inputs as Fortran array of ones
input = 1.0

! Wrap Fortran data as no-copy Torch Tensors
! There may well be some reshaping required depending on the 
! structure of the model which is not covered here (see examples)
model_input_arr(1) = torch_tensor_from_array(input, in_layout, torch_kCPU)
model_output = torch_tensor_from_array(output, out_layout, torch_kCPU)

! Run model and Infer
! Again, there may be some reshaping required depending on model design
call torch_module_forward(model, model_input_arr, n_inputs, model_output)

! Write out the result of running the model
write(*,*) output

! Clean up
call torch_module_delete(model)
call torch_tensor_delete(model_input_arr(1))
call torch_tensor_delete(model_output)
```

### 3. Build the code

The code now needs to be compiled and linked against our installed library.
Here we describe how to do this for two build systems, cmake and make.

#### CMake
If our project were using cmake we would need the following in the `CMakeLists.txt` file to find the FTorch installation and link it to the executable.

This can be done by adding the following to the `CMakeLists.txt` file:
```CMake
find_package(FTorch)
target_link_libraries( <executable> PRIVATE FTorch::ftorch )
message(STATUS "Building with Fortran PyTorch coupling")
```
and using the `-DCMAKE_PREFIX_PATH=</path/to/install/location>` flag when running cmake.  
_Note: If you used the `CMAKE_INSTALL_PREFIX` argument when
[building and installing the library](#library-installation) above then you should use
the same path for `</path/to/install/location>`._

#### Make
To build with make we need to include the library when compiling and link the executable
against it.

To compile with make we need add the following compiler flag when compiling files that
use ftorch:
```makefile
FCFLAGS += -I<path/to/install/location>/include/ftorch
```

When compiling the final executable add the following link flag:
```makefile
LDFLAGS += -L<path/to/install/location>/lib -lftorch
```

You may also need to add the location of the `.so` files to your `LD_LIBRARY_PATH`
unless installing in a default location:
```bash
export LD_LIBRARY_PATH = $LD_LIBRARY_PATH:<path/to/install/location>/lib
```
_Note: depending on your system and architecture `lib` may be `lib64` or something similar._

### 4. Running on GPUs

In order to run a model on GPU, two main changes are required:

1. When saving your TorchScript model, ensure that it is on the GPU. For example, when using [pt2ts.py](utils/pt2ts.py), this can be done by uncommenting the following lines:

```
device = torch.device('cuda')
trained_model = trained_model.to(device)
trained_model.eval()
trained_model_dummy_input_1 = trained_model_dummy_input_1.to(device)
trained_model_dummy_input_2 = trained_model_dummy_input_2.to(device)
```

Note: this also moves the dummy input tensors to the GPU. This is not necessary for saving the model, but the tensors must also be on the GPU to test that the models runs.


2. When calling `torch_tensor_from_blob` in Fortran, the device for the input tensor(s), but not the output tensor(s),
   should be set to `torch_kCUDA`, rather than `torch_kCPU`. This ensures that the inputs are on the same device as the model.


## Examples

Examples of how to use this library are provided in the [examples directory](examples/).  
They demonstrate different functionalities and are provided with instructions to modify, build, and run as necessary.

## License

Copyright &copy; ICCS

*FTorch* is distributed under the [MIT Licence](https://github.com/Cambridge-ICCS/FTorch/blob/main/LICENSE).


## Contributions

Contributions and collaborations are welcome.

For bugs, feature requests, and clear suggestions for improvement please
[open an issue](https://github.com/Cambridge-ICCS/FTorch/issues).

If you have built something upon _FTorch_ that would be useful to others, or can
address an [open issue](https://github.com/Cambridge-ICCS/FTorch/issues), please
[fork the repository](https://github.com/Cambridge-ICCS/FTorch/fork) and open a
pull request.


### Code of Conduct
Everyone participating in the _FTorch_ project, and in particular in the
issue tracker, pull requests, and social media activity, is expected to treat other
people with respect and, more generally, to follow the guidelines articulated in the
[Python Community Code of Conduct](https://www.python.org/psf/codeofconduct/).


## Authors and Acknowledgment

*FTorch* is written and maintained by the [ICCS](https://github.com/Cambridge-ICCS)

Notable contributors to this project are:

* [**@athelaf**](https://github.com/athelaf)
* [**@jatkinson1000**](https://github.com/jatkinson1000)
* [**@SimonClifford**](https://github.com/SimonClifford)
* [**@ElliottKasoar**](https://github.com/ElliottKasoar)
* [**@TomMelt**](https://github.com/TomMelt)

See [Contributors](https://github.com/Cambridge-ICCS/FTorch/graphs/contributors)
for a full list.


## Used by
The following projects make use of this code or derivatives in some way:

* [DataWave - MiMA ML](https://github.com/DataWaveProject/MiMA-machine-learning)

Are we missing anyone? Let us know.



