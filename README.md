# clblas-opencl11-deb

This repo has the `debian` tree for building the clBLAS debs
(`libclblas-dev`, `libclblas2`) with support for OpenCL 1.1 devices, or
indeed any OpenCL version devices.


## Background

As I found out after a lot of debugging, this elusive error:

    OpenCL error -11 [...]
    .../clblas-2.12/src/library/blas/xgemm.cc:244: void makeGemmKernel(_cl_kernel**, cl_command_queue, const char*, const char*, const unsigned char**, size_t*, const char*): Assertion `false' failed.

which indicates a failure in `clBuildProgram()` but inconveniently returns
an empty build log in subsequent calls to `clGetProgramBuildInfo()`, was
the simple result of build option `-cl-std="CL1.2"` being passed while the
targeted device only supported OpenCL 1.1.

The `man` page for `clBuildProgram` states that "if the `-cl-std` build option
is not specified, then the device's `CL_DEVICE_OPENCL_C_VERSION` determines
the OpenCL version used for compilation".  This is precisely what we want.

The debian tree in this repository has a patch (`remove-cl-std-build-option`)
which resolves the issue.  In addition, it has `cherrypicked-develop-patches`,
which applies a handful of patches picked from
[clBLAS](https://github.com/clMathLibraries/clBLAS)'s develop branch.  The
patches can be applied independently.

I will file a PR to upstream and an Ubuntu bug report, but am also pushing
this repo online for the benefit of others running into the issue, as it
currently blocks anyone on slightly older hardware from using clBLAS or ArrayFire.


## Instructions

Note: the debian tree in this repo is not meant to replace the one in the
Debian git sources.  It is a stripped down version that builds just the
`libclblas2` and `libclblas-dev` debs.

Having said that, to mainline the changes, all that should be needed is to
add the patch(es) to the dquilt series.

#### Build steps

* Installed all packages needed for building a Debian package (see elsewhere)
* Clone this repository: `git clone git@github.com:zwets/clblas-opencl11-deb.git`
* Subsequent steps all in the root of the cloned repo
* Install build dependencies; see `Build-Dependencies` in `debian/control`
* Get the 2.12 clblas tarball from https://github.com/clMathLibraries/clBLAS/releases
* Unpack: `tar --strip-components=1 -xzf /path/to/tarball.tgz`
* Recover overwritten README.md: `git checkout .`
* Refresh patches: `while dquilt push; do dquilt refresh; done`
* Build: `debuild -b -uc -us`
* Install: `sudo apt install ../libclblas2_*.deb ../libclblas-dev_*.deb`

