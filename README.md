# clblas-opencl11-deb

This repo has the `debian` tree for building the clBLAS debs
 (`libclblas-dev`, `libclblas2`) with support for OpenCL 1.1.

## Background

As I found out after a lot of debugging, this elusive error:

    OpenCL error -11 [...]
    .../clblas-2.12/src/library/blas/xgemm.cc:244: void makeGemmKernel(_cl_kernel**, cl_command_queue, const char*, const char*, const unsigned char**, size_t*, const char*): Assertion `false' failed.

which indicates a failure in `clBuildProgram()` but inconveniently returns
an empty build log in a subsequent call to `clGetProgramBuildInfo()`, was
the simple result of build option `-cl-std="CL1.2"` being passed while the
targeted device only supported OpenCL 1.1.

The debian tree in this repository fixes this issue by configuring clblas
with `-DOPENCL_VERSION=1.1`.  It also applies a handful of patches picked from
the `develop` branch of [clBLAS](https://github.com/clMathLibraries/clBLAS).

> Note that this is a stopgap solution.  The proper fix would be to not pass
> any `-cl-std=...` build option to `clBuildProgram`.  Its man page states
> that "if the `-cl-std` build option is not specified, then the device's
> `CL_DEVICE_OPENCL_C_VERSION` determines the OpenCL version used for
> compilation".  That would be ideal, but alas my (quick) attempt resulted
> in a segfault.  This needs more digging - and a possible bug report against
> the OpenCL backend.  Note to self: also report the empty build log bug.

This repo generates only the `libclblas2` and `libclblas-dev` debs.  I
stripped out the `-client` and `-doc` packages, as they have huge dependencies.

## Building

@@TODO@@ details

* Get the 2.12 release from https://github.com/clMathLibraries/clBLAS/releases
* Unpack 
* Copy `./debian` into the unpacked release directory
* Install all Build-Dependencies listed in `debian/control`
* Run `debuild -b -uc -us`
* `sudo apt install ../libclblas2_*.deb ../libclblas-dev_*.deb`

