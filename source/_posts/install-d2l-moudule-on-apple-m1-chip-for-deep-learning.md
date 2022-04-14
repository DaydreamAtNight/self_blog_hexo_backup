---
title: install d2l module on apple m1 chip for deep learning
author: daydreamatnight
toc: true
declare: true
date: 2022-02-28 22:30:11
tags:
  - m1 mac
  - deep learning
---

> [d2l](https://pypi.org/project/d2l/) is a small python module wheel that needed when read the useful deep learning book "[dive into deeplearning](https://d2l.ai/)", which provide interactive code examples implemented with [MXNet](https://mxnet.apache.org/versions/1.9.0/), PyTorch, and Tensorflow. 
>
> But it took me ton's of time installing this module on the new M1 MacBook Air. Actually its easy, just to record this.

<!-- more -->

### How to install

1.install miniforge

already did, easy.

2.create a new environment with python=3.8

m1 Mac officially support python>=3.9, but 3.8 can be installed.

```shell
conda create -n d2l python=3.8
conda info --env
conda activate d2l
```

3.install torch

torch==1.8.1 and torchvision==0.9.1 is recommended and tested in the book, but [# macOS is not currently supported for lts](https://pytorch.org/). 

So the most convenient choice for mac is pytorch==1.10.2, torchvision==0.2.2

```python
conda install pytorch torchvision -c pytorch
```

#4.try install d2l directly

```shell
clear
pip install d2l==0.17.3
```

thousands lines of terrifying error will come out:

```shell
$ pip install d2l==0.17.3
Collecting d2l==0.17.3
  Using cached d2l-0.17.3-py3-none-any.whl (82 kB)
Collecting jupyter==1.0.0
  Using cached jupyter-1.0.0-py2.py3-none-any.whl (2.7 kB)
Collecting numpy==1.18.5
  Using cached numpy-1.18.5.zip (5.4 MB)
  Installing build dependencies ... done
  Getting requirements to build wheel ... done
  Preparing metadata (pyproject.toml) ... done
Collecting pandas==1.2.2
  Using cached pandas-1.2.2.tar.gz (5.5 MB)
  Installing build dependencies ... error
  error: subprocess-exited-with-error
  
  × pip subprocess to install build dependencies did not run successfully.
  │ exit code: 1
  ╰─> [3659 lines of output]
      Ignoring numpy: markers 'python_version == "3.7" and platform_system != "AIX"' don't match your environment
      Ignoring numpy: markers 'python_version == "3.7" and platform_system == "AIX"' don't match your environment
      Ignoring numpy: markers 'python_version == "3.8" and platform_system == "AIX"' don't match your environment
      Ignoring numpy: markers 'python_version >= "3.9"' don't match your environment
      Collecting setuptools
        Using cached setuptools-60.9.3-py3-none-any.whl (1.1 MB)
      Collecting wheel
        Using cached wheel-0.37.1-py2.py3-none-any.whl (35 kB)
      Collecting Cython<3,>=0.29.21
        Using cached Cython-0.29.28-py2.py3-none-any.whl (983 kB)
      Collecting numpy==1.17.3
        Using cached numpy-1.17.3.zip (6.4 MB)
        Preparing metadata (setup.py): started
        Preparing metadata (setup.py): finished with status 'done'
      Building wheels for collected packages: numpy
        Building wheel for numpy (setup.py): started
        Building wheel for numpy (setup.py): finished with status 'error'
        error: subprocess-exited-with-error
      
        × python setup.py bdist_wheel did not run successfully.
        │ exit code: 1
        ╰─> [3286 lines of output]
            Running from numpy source directory.
            blas_opt_info:
            blas_mkl_info:
            customize UnixCCompiler
              libraries mkl_rt not found in ['/opt/homebrew/Caskroom/miniforge/base/envs/d2d/lib', '/usr/local/lib', '/usr/lib']
              NOT AVAILABLE
      
            blis_info:
            customize UnixCCompiler
              libraries blis not found in ['/opt/homebrew/Caskroom/miniforge/base/envs/d2d/lib', '/usr/local/lib', '/usr/lib']
              NOT AVAILABLE
      
            openblas_info:
            customize UnixCCompiler
            customize UnixCCompiler
              libraries openblas not found in ['/opt/homebrew/Caskroom/miniforge/base/envs/d2d/lib', '/usr/local/lib', '/usr/lib']
              NOT AVAILABLE
      
            atlas_3_10_blas_threads_info:
            Setting PTATLAS=ATLAS
            customize UnixCCompiler
              libraries tatlas not found in ['/opt/homebrew/Caskroom/miniforge/base/envs/d2d/lib', '/usr/local/lib', '/usr/lib']
              NOT AVAILABLE
      
            atlas_3_10_blas_info:
            customize UnixCCompiler
              libraries satlas not found in ['/opt/homebrew/Caskroom/miniforge/base/envs/d2d/lib', '/usr/local/lib', '/usr/lib']
              NOT AVAILABLE
      
            atlas_blas_threads_info:
            Setting PTATLAS=ATLAS
            customize UnixCCompiler
              libraries ptf77blas,ptcblas,atlas not found in ['/opt/homebrew/Caskroom/miniforge/base/envs/d2d/lib', '/usr/local/lib', '/usr/lib']
              NOT AVAILABLE
      
            atlas_blas_info:
            customize UnixCCompiler
              libraries f77blas,cblas,atlas not found in ['/opt/homebrew/Caskroom/miniforge/base/envs/d2d/lib', '/usr/local/lib', '/usr/lib']
              NOT AVAILABLE
      
            accelerate_info:
            customize UnixCCompiler
              libraries accelerate not found in ['/opt/homebrew/Caskroom/miniforge/base/envs/d2d/lib', '/usr/local/lib', '/usr/lib']
            Library accelerate was not found. Ignoring
            customize UnixCCompiler
              libraries veclib not found in ['/opt/homebrew/Caskroom/miniforge/base/envs/d2d/lib', '/usr/local/lib', '/usr/lib']
            Library veclib was not found. Ignoring
              FOUND:
                extra_compile_args = ['-faltivec', '-I/System/Library/Frameworks/vecLib.framework/Headers']
                extra_link_args = ['-Wl,-framework', '-Wl,Accelerate']
                define_macros = [('NO_ATLAS_INFO', 3), ('HAVE_CBLAS', None)]
      
              FOUND:
                extra_compile_args = ['-faltivec', '-I/System/Library/Frameworks/vecLib.framework/Headers']
                extra_link_args = ['-Wl,-framework', '-Wl,Accelerate']
                define_macros = [('NO_ATLAS_INFO', 3), ('HAVE_CBLAS', None)]
      
            /bin/sh: svnversion: command not found
            non-existing path in 'numpy/distutils': 'site.cfg'
            lapack_opt_info:
            lapack_mkl_info:
            customize UnixCCompiler
              libraries mkl_rt not found in ['/opt/homebrew/Caskroom/miniforge/base/envs/d2d/lib', '/usr/local/lib', '/usr/lib']
              NOT AVAILABLE
      
            openblas_lapack_info:
            customize UnixCCompiler
            customize UnixCCompiler
              libraries openblas not found in ['/opt/homebrew/Caskroom/miniforge/base/envs/d2d/lib', '/usr/local/lib', '/usr/lib']
              NOT AVAILABLE
      
            openblas_clapack_info:
            customize UnixCCompiler
            customize UnixCCompiler
              libraries openblas,lapack not found in ['/opt/homebrew/Caskroom/miniforge/base/envs/d2d/lib', '/usr/local/lib', '/usr/lib']
              NOT AVAILABLE
      
            flame_info:
            customize UnixCCompiler
              libraries flame not found in ['/opt/homebrew/Caskroom/miniforge/base/envs/d2d/lib', '/usr/local/lib', '/usr/lib']
              NOT AVAILABLE
      
            atlas_3_10_threads_info:
            Setting PTATLAS=ATLAS
            customize UnixCCompiler
              libraries lapack_atlas not found in /opt/homebrew/Caskroom/miniforge/base/envs/d2d/lib
...
...
...
	
						None - nothing done with h_files = ['build/src.macosx-11.0-arm64-3.8/numpy/core/src/npymath/npy_math_internal.h']
            building library "npysort" sources
              adding 'build/src.macosx-11.0-arm64-3.8/numpy/core/src/common' to include_dirs.
            None - nothing done with h_files = ['build/src.macosx-11.0-arm64-3.8/numpy/core/src/common/npy_sort.h', 'build/src.macosx-11.0-arm64-3.8/numpy/core/src/common/npy_partition.h', 'build/src.macosx-11.0-arm64-3.8/numpy/core/src/common/npy_binsearch.h']
            building extension "numpy.core._dummy" sources
              adding 'build/src.macosx-11.0-arm64-3.8/numpy/core/include/numpy/config.h' to sources.
              adding 'build/src.macosx-11.0-arm64-3.8/numpy/core/include/numpy/_numpyconfig.h' to sources.
            executing numpy/core/code_generators/generate_numpy_api.py
              adding 'build/src.macosx-11.0-arm64-3.8/numpy/core/include/numpy/__multiarray_api.h' to sources.
            numpy.core - nothing done with h_files = ['build/src.macosx-11.0-arm64-3.8/numpy/core/include/numpy/config.h', 'build/src.macosx-11.0-arm64-3.8/numpy/core/include/numpy/_numpyconfig.h', 'build/src.macosx-11.0-arm64-3.8/numpy/core/include/numpy/__multiarray_api.h']
            building extension "numpy.core._multiarray_tests" sources
            building extension "numpy.core._multiarray_umath" sources
              adding 'build/src.macosx-11.0-arm64-3.8/numpy/core/include/numpy/config.h' to sources.
              adding 'build/src.macosx-11.0-arm64-3.8/numpy/core/include/numpy/_numpyconfig.h' to sources.
            executing numpy/core/code_generators/generate_numpy_api.py
              adding 'build/src.macosx-11.0-arm64-3.8/numpy/core/include/numpy/__multiarray_api.h' to sources.
            executing numpy/core/code_generators/generate_ufunc_api.py
              adding 'build/src.macosx-11.0-arm64-3.8/numpy/core/include/numpy/__ufunc_api.h' to sources.
              adding 'build/src.macosx-11.0-arm64-3.8/numpy/core/src/umath' to include_dirs.
              adding 'build/src.macosx-11.0-arm64-3.8/numpy/core/src/npymath' to include_dirs.
              adding 'build/src.macosx-11.0-arm64-3.8/numpy/core/src/common' to include_dirs.
            numpy.core - nothing done with h_files = ['build/src.macosx-11.0-arm64-3.8/numpy/core/src/umath/funcs.inc', 'build/src.macosx-11.0-arm64-3.8/numpy/core/src/umath/simd.inc', 'build/src.macosx-11.0-arm64-3.8/numpy/core/src/umath/loops.h', 'build/src.macosx-11.0-arm64-3.8/numpy/core/src/umath/matmul.h', 'build/src.macosx-11.0-arm64-3.8/numpy/core/src/umath/clip.h', 'build/src.macosx-11.0-arm64-3.8/numpy/core/src/npymath/npy_math_internal.h', 'build/src.macosx-11.0-arm64-3.8/numpy/core/src/common/templ_common.h', 'build/src.macosx-11.0-arm64-3.8/numpy/core/include/numpy/config.h', 'build/src.macosx-11.0-arm64-3.8/numpy/core/include/numpy/_numpyconfig.h', 'build/src.macosx-11.0-arm64-3.8/numpy/core/include/numpy/__multiarray_api.h', 'build/src.macosx-11.0-arm64-3.8/numpy/core/include/numpy/__ufunc_api.h']
            building extension "numpy.core._umath_tests" sources
            building extension "numpy.core._rational_tests" sources
            building extension "numpy.core._struct_ufunc_tests" sources
            building extension "numpy.core._operand_flag_tests" sources
            building extension "numpy.fft._pocketfft_internal" sources
            building extension "numpy.linalg.lapack_lite" sources
              adding 'numpy/linalg/lapack_lite/python_xerbla.c' to sources.
            building extension "numpy.linalg._umath_linalg" sources
              adding 'numpy/linalg/lapack_lite/python_xerbla.c' to sources.
            building extension "numpy.random.mt19937" sources
            building extension "numpy.random.philox" sources
            building extension "numpy.random.pcg64" sources
            building extension "numpy.random.sfc64" sources
            building extension "numpy.random.common" sources
            building extension "numpy.random.bit_generator" sources
            building extension "numpy.random.generator" sources
            building extension "numpy.random.bounded_integers" sources
            building extension "numpy.random.mtrand" sources
            building data_files sources
            build_src: building npy-pkg config files
            running build_py
            copying numpy/version.py -> build/lib.macosx-11.0-arm64-3.8/numpy
            copying build/src.macosx-11.0-arm64-3.8/numpy/__config__.py -> build/lib.macosx-11.0-arm64-3.8/numpy
            copying build/src.macosx-11.0-arm64-3.8/numpy/distutils/__config__.py -> build/lib.macosx-11.0-arm64-3.8/numpy/distutils
            running build_clib
            customize UnixCCompiler
            customize UnixCCompiler using build_clib
            running build_ext
            customize UnixCCompiler
            customize UnixCCompiler using build_ext
            building 'numpy.core._multiarray_umath' extension
            compiling C sources
            C compiler: gcc -Wno-unused-result -Wsign-compare -Wunreachable-code -DNDEBUG -g -fwrapv -O3 -Wall -Wstrict-prototypes -I/opt/homebrew/Caskroom/miniforge/base/envs/d2d/include -arch arm64 -I/opt/homebrew/Caskroom/miniforge/base/envs/d2d/include -arch arm64
      
            compile options: '-DNPY_INTERNAL_BUILD=1 -DHAVE_NPY_CONFIG_H=1 -D_FILE_OFFSET_BITS=64 -D_LARGEFILE_SOURCE=1 -D_LARGEFILE64_SOURCE=1 -DNO_ATLAS_INFO=3 -DHAVE_CBLAS -Ibuild/src.macosx-11.0-arm64-3.8/numpy/core/src/umath -Ibuild/src.macosx-11.0-arm64-3.8/numpy/core/src/npymath -Ibuild/src.macosx-11.0-arm64-3.8/numpy/core/src/common -Inumpy/core/include -Ibuild/src.macosx-11.0-arm64-3.8/numpy/core/include/numpy -Inumpy/core/src/common -Inumpy/core/src -Inumpy/core -Inumpy/core/src/npymath -Inumpy/core/src/multiarray -Inumpy/core/src/umath -Inumpy/core/src/npysort -I/opt/homebrew/Caskroom/miniforge/base/envs/d2d/include/python3.8 -Ibuild/src.macosx-11.0-arm64-3.8/numpy/core/src/common -Ibuild/src.macosx-11.0-arm64-3.8/numpy/core/src/npymath -Ibuild/src.macosx-11.0-arm64-3.8/numpy/core/src/common -Ibuild/src.macosx-11.0-arm64-3.8/numpy/core/src/npymath -c'
            extra options: '-faltivec -I/System/Library/Frameworks/vecLib.framework/Headers'
            gcc: numpy/core/src/multiarray/alloc.c
            gcc: numpy/core/src/multiarray/array_assign_scalar.c
            gcc: numpy/core/src/multiarray/buffer.c
            gcc: numpy/core/src/multiarray/common.c
            gcc: numpy/core/src/multiarray/conversion_utils.c
            gcc: numpy/core/src/multiarray/datetime_strings.c
            gcc: numpy/core/src/multiarray/descriptor.c
            gcc: build/src.macosx-11.0-arm64-3.8/numpy/core/src/multiarray/einsum.c
            clang: error: the clang compiler does not support 'faltivec', please use -maltivec and include altivec.h explicitly
            clang: error: the clang compiler does not support 'faltivec', please use -maltivec and include altivec.h explicitly
            gcc: numpy/core/src/multiarray/hashdescr.c
            clang: error: the clang compiler does not support 'faltivec', please use -maltivec and include altivec.h explicitly
            clang: error: the clang compiler does not support 'faltivec', please use -maltivec and include altivec.h explicitly
            gcc: build/src.macosx-11.0-arm64-3.8/numpy/core/src/multiarray/lowlevel_strided_loops.c
            gcc: numpy/core/src/multiarray/multiarraymodule.c
            gcc: numpy/core/src/multiarray/nditer_constr.c
            clang: error: the clang compiler does not support 'faltivec', please use -maltivec and include altivec.h explicitly
            gcc: numpy/core/src/multiarray/refcount.c
            clang: error: the clang compiler does not support 'faltivec', please use -maltivec and include altivec.h explicitly
            clang: error: the clang compiler does not support 'faltivec', please use -maltivec and include altivec.h explicitly
            gcc: numpy/core/src/multiarray/scalarapi.c
            clang: error: the clang compiler does not support 'faltivec', please use -maltivec and include altivec.h explicitly
            gcc: numpy/core/src/multiarray/temp_elide.c
            gcc: numpy/core/src/multiarray/vdot.c
            clang: error: the clang compiler does not support 'faltivec', please use -maltivec and include altivec.h explicitly
            clang: error: the clang compiler does not support 'faltivec', please use -maltivec and include altivec.h explicitly
            clang: error: the clang compiler does not support 'faltivec', please use -maltivec and include altivec.h explicitly
            gcc: build/src.macosx-11.0-arm64-3.8/numpy/core/src/umath/loops.c
            gcc: numpy/core/src/umath/ufunc_object.c
            gcc: build/src.macosx-11.0-arm64-3.8/numpy/core/src/umath/scalarmath.c
            clang: error: the clang compiler does not support 'faltivec', please use -maltivec and include altivec.h explicitly
            clang: error: the clang compiler does not support 'faltivec', please use -maltivec and include altivec.h explicitly
            clang: error: the clang compiler does not support 'faltivec', please use -maltivec and include altivec.h explicitly
            gcc: numpy/core/src/npymath/npy_math.c
            gcc: numpy/core/src/common/npy_longdouble.c
            clang: error: the clang compiler does not support 'faltivec', please use -maltivec and include altivec.h explicitly
            gcc: numpy/core/src/npymath/halffloat.c
            clang: error: the clang compiler does not support 'faltivec', please use -maltivec and include altivec.h explicitly
            gcc: numpy/core/src/common/numpyos.c
            gcc: /private/var/folders/5y/pqfqz2md0njg2slq29yxp12w0000gn/T/pip-install-mxyh83f9/numpy_51614e6143884c3bbd246341eeb3b857/numpy/_build_utils/src/apple_sgemv_fix.c
            clang: error: the clang compiler does not support 'faltivec', please use -maltivec and include altivec.h explicitly
            clang: error: the clang compiler does not support 'faltivec', please use -maltivec and include altivec.h explicitly
            clang: error: the clang compiler does not support 'faltivec', please use -maltivec and include altivec.h explicitly
            clang: error: the clang compiler does not support 'faltivec', please use -maltivec and include altivec.h explicitly
            clang: error: the clang compiler does not support 'faltivec', please use -maltivec and include altivec.h explicitlyclang: error: the clang compiler does not support 'faltivec', please use -maltivec and include altivec.h explicitly
      
            clang: error: the clang compiler does not support 'faltivec', please use -maltivec and include altivec.h explicitly
            clang: error: the clang compiler does not support 'faltivec', please use -maltivec and include altivec.h explicitly
            error: Command "gcc -Wno-unused-result -Wsign-compare -Wunreachable-code -DNDEBUG -g -fwrapv -O3 -Wall -Wstrict-prototypes -I/opt/homebrew/Caskroom/miniforge/base/envs/d2d/include -arch arm64 -I/opt/homebrew/Caskroom/miniforge/base/envs/d2d/include -arch arm64 -DNPY_INTERNAL_BUILD=1 -DHAVE_NPY_CONFIG_H=1 -D_FILE_OFFSET_BITS=64 -D_LARGEFILE_SOURCE=1 -D_LARGEFILE64_SOURCE=1 -DNO_ATLAS_INFO=3 -DHAVE_CBLAS -Ibuild/src.macosx-11.0-arm64-3.8/numpy/core/src/umath -Ibuild/src.macosx-11.0-arm64-3.8/numpy/core/src/npymath -Ibuild/src.macosx-11.0-arm64-3.8/numpy/core/src/common -Inumpy/core/include -Ibuild/src.macosx-11.0-arm64-3.8/numpy/core/include/numpy -Inumpy/core/src/common -Inumpy/core/src -Inumpy/core -Inumpy/core/src/npymath -Inumpy/core/src/multiarray -Inumpy/core/src/umath -Inumpy/core/src/npysort -I/opt/homebrew/Caskroom/miniforge/base/envs/d2d/include/python3.8 -Ibuild/src.macosx-11.0-arm64-3.8/numpy/core/src/common -Ibuild/src.macosx-11.0-arm64-3.8/numpy/core/src/npymath -Ibuild/src.macosx-11.0-arm64-3.8/numpy/core/src/common -Ibuild/src.macosx-11.0-arm64-3.8/numpy/core/src/npymath -c numpy/core/src/multiarray/array_assign_scalar.c -o build/temp.macosx-11.0-arm64-3.8/numpy/core/src/multiarray/array_assign_scalar.o -MMD -MF build/temp.macosx-11.0-arm64-3.8/numpy/core/src/multiarray/array_assign_scalar.o.d -faltivec -I/System/Library/Frameworks/vecLib.framework/Headers" failed with exit status 1
            [end of output]
      
        note: This error originates from a subprocess, and is likely not a problem with pip.
      error: legacy-install-failure
      
      × Encountered error while trying to install package.
      ╰─> numpy
      
      note: This is an issue with the package mentioned above, not pip.
      hint: See above for output from the failure.
      [end of output]
  
  note: This error originates from a subprocess, and is likely not a problem with pip.
error: subprocess-exited-with-error

× pip subprocess to install build dependencies did not run successfully.
│ exit code: 1
╰─> See above for output.

note: This error originates from a subprocess, and is likely not a problem with pip.
```

Scroll to the top, it looks like `jupyter==1.0.0`, `pandas==1.2.2`, `numpy==1.18.5` are required, and numpy==1.18.5 is where the error comes from.

4.As a result, install `jupyter==1.0.0` and `pandas==1.2.2` first

```shell
pip install jupyter==1.0.0
```

conda can't be installed by pip, but can by conda

```shell
conda install pandas=1.2.2
```

5.numpy==1.18.5 can not be installed by pip or conda

thanks to [tensorflow's wheel](https://github.com/apple/tensorflow_macos/releases/tag/v0.1alpha0), numpy==1.18.5's wheel for mac is included in the addons

download and unzip [tensorflow_macos-0.1alpha0.tar.gz](https://github.com/apple/tensorflow_macos/releases/download/v0.1alpha0/tensorflow_macos-0.1alpha0.tar.gz)

go to the unzipped folder in terminal, and run

```shell
pip install arm64/numpy-1.18.5-cp38-cp38-macosx_11_0_arm64.whl
```

check

```shell
pip list |grep pandas
pip list |grep jupyter
pip list |grep numpy
```

```shell
pandas               1.2.2
jupyter              1.0.0
jupyter-client       7.1.2
jupyter-console      6.4.0
jupyter-core         4.9.2
jupyterlab-pygments  0.1.2
jupyterlab-widgets   1.0.2
numpy                1.18.5
```

5.install d2l

```shell
pip install d2l
```

Sucess!

### Reference  

https://zh-v2.d2l.ai/chapter_installation/index.html

https://parthiban-kannan.medium.com/install-tensorflow-on-apple-macbook-m1-release-c1ce7e65cd0

https://github.com/apple/tensorflow_macos/issues/48





