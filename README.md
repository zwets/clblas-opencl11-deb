# clblas-opencl11-deb

This repo has the `debian` tree that resolves the "Error -11" in `libclBLAS2`
that prevents clBLAS (and much of ArrayFire) from working over Clover
(`mesa-opencl-icd` and `libclc`) on AMD GPUs.

The patch for this issue is now attached to this
[Ubuntu bug report](https://bugs.launchpad.net/ubuntu/+source/clblas/+bug/1816887),
and in this
[upstream PR](https://github.com/clMathLibraries/clBLAS/pull/342).  Until
those go mainline this repo can be used to build working `libclblas2` and
`libclblas-dev` debs.


## Background

As I found out after a lot of debugging, this elusive error:

    OpenCL error -11 [...]
    .../clblas-2.12/src/library/blas/xgemm.cc:244: void makeGemmKernel(_cl_kernel**, cl_command_queue, const char*, const char*, const unsigned char**, size_t*, const char*): Assertion `false' failed.

which indicates a failure in `clBuildProgram()` but inconveniently returns
an empty build log in subsequent calls to `clGetProgramBuildInfo()`, was
the simple result of build option `-cl-std="CL1.2"` being passed to
`clBuildProgram()` while the targeted device was OpenCL 1.1.  

The `man` page for `clBuildProgram` states that "if the `-cl-std` build option
is not specified, then the device's `CL_DEVICE_OPENCL_C_VERSION` determines
the OpenCL version used for compilation".  This is precisely what we want, and
the `remove-cl-std-build-option` patch in this repo fixes this.

In addition, this repo has `cherrypicked-develop-patches`, a handful of patches
picked from the [upstream](https://github.com/clMathLibraries/clBLAS) develop
branch.  The patches can be applied independently (comment either out in
`debian/patches/series` to disable it).


## Instructions

*Note:* the debian tree in this repo is not meant to replace the one in the
Debian git sources.  It is a stripped down version that builds just the
`libclblas2` and `libclblas-dev` debs.

* Installed all packages needed to build Debian packages (see elsewhere)
* Clone this repository: `git clone git@github.com:zwets/clblas-opencl11-deb.git`
* Subsequent steps all in the root of the cloned repo
* Install build dependencies; see `Build-Dependencies` in `debian/control`
* Get the 2.12 clblas tarball from https://github.com/clMathLibraries/clBLAS/releases
* Unpack: `tar --strip-components=1 -xzf /path/to/tarball.tgz`
* Recover overwritten README.md: `git checkout .`
* Apply patches: `while dquilt push; do dquilt refresh; done`
* Build: `debuild -b -uc -us`
* Install: `sudo apt install ../libclblas2_*.deb ../libclblas-dev_*.deb`

