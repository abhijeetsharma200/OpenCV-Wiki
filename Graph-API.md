What it is?
===========

* OpenCV 4.0 comes with an experimental Graph API module (see [opencv/modules/gapi](https://github.com/opencv/opencv/tree/master/modules/gapi)). This is a new API which allows to enable offload and optimizations for image processing / CV algorithms on pipeline level.

* The idea behind G-API is to declare image processing task in form of expressions and then submit it for execution – using a number of available backends. At the moment, there’s reference “CPU” (OpenCV-based), "GPU" (also OpenCV T-API-based), and experimental “Fluid’ backends available, with other backends coming up next.

* G-API is an uncommon OpenCV module since it acts as a framework: it provides means for declaring operations, building graphs of operations, and finally implementing the operations for a particular backend. G-API model enforces separation between interfaces and implementations, so once an algorithm is expressed in G-API terms, it can be scaled/ported/offloaded to a new platform easily.

* G-API CPU (OpenCV) backend implements G-API standard functions using OpenCV itself (core/imgproc modules) and acts as a quick prototyping/porting/testing backend. If you have an image processing algorithm composed of OpenCV-like functions already, you would be able to switch quickly to G-API by using this backend.

* G-API Fluid backend implements a cache-efficient execution model and allows to save memory dramatically – e.g. a 1.5GB image processing pipeline fits into 750KB memory footprint with G-API/Fluid. G-API comes with a number of operations implemented for Fluid backend, so one can switch OpenCV/Fluid operations within a graph easily and even mix both in the same graph.

* G-API GPU backend implements the majority of available functions and allows to run OpenCL kernels on available OpenCL-programmable devices. At the moment, GPU backend is based on OpenCV Transparent API; in future versions it will be extended to support integration of arbitrary OpenCL kernels (and likely be renamed to "OpenCL backend").

* G-API ONNX backend implements [ONNX models](https://github.com/onnx/models) inference operations on input data and outputs the results. At the moment, ONNX backend is based on [ONNX Runtime](https://github.com/microsoft/onnxruntime) C/C++ API.

# Building G-API

G-API is built with OpenCV by default, however some features may require additional options or dependencies enabled.

## Building with OpenVINO Toolkit support
* Get `openvino` from https://docs.openvinotoolkit.org/
* Configure enviroment:
```bash
source path-to-unpacked-openvino/bin/setupvars.sh
```
* Build G-API with OpenVINO support:
```bash
cmake /path-to-opencv -DWITH_INF_ENGINE=ON -DINF_ENGINE_RELEASE=2021030000
```

*NOTE*: Set `INF_ENGINE_RELEASE` to the proper version, depending on the package you use.

## Building with PlaidML support
* Follow instruction to build PlaidML2: https://plaidml.github.io/plaidml/docs/building
```bash
bazelisk build //plaidml2:wheel
pip install -U bazel-bin/plaidml2/wheel.pkg/tmp/dist/*
```
* Setup PLAIDML_DEVICE
```bash
python3 /path-to-plaidml/plaidml/plaidml2/plaidml_setup.py
export `cat ~/.plaidml2`
```
* Build G-API with PlaidML2 support:
```bash
cmake /path-to-opencv -DPlaidML2_DIR=path-to-miniconda3/share/plaidml2 -DWITH_PLAIDML=ON
```
* Run tests:
```bash
/path-to-opencv-build/bin/opencv_test_gapi --gtest_filter=*GAPI_PlaidML_Pipelines*
```

## Building with Microsoft ONNX Runtime support

* Build and install the ONNX RT (currently tested with v1.5.1):

```bash
 $ git clone --recursive https://github.com/microsoft/onnxruntime.git
 $ cd onnxruntime
 $ git checkout v1.5.1
 $ git submodule update --init
 $ ./build.sh --config Release --build_shared_lib --parallel \
 $     --cmake_extra_defines CMAKE_INSTALL_PREFIX=install
 $ cd build/Linux/Release
 $ make install
```

* Then specify extra options to OpenCV CMake:    

```bash
 $ cmake /path-to-opencv -DWITH_ONNX=ON -DORT_INSTALL_DIR=/path-to-ort-install-dir
```

Testing G-API
=============

By default, the OpenCV G-API comes with its own test suite (`opencv_test_gapi`). Note that extra (external) G-API modules may introduce their own test suites. G-API tests are built and run in a regular way:

## Linux

```
$ make -j4 opencv_test_gapi
$ bin/opencv_test_gapi
```

## Windows

```
$ cmake --build . --target opencv_test_gapi --config Release -- /maxcpucount:4
$ bin\Release\opencv_test_gapi.exe
```

A tiny fraction of G-API tests requires external test data to be available. This data is taken from the [opencv_extra](http://github.com/opencv/opencv_extra) repo:

```
export OPENCV_TEST_DATA_PATH=/path/to/opencv_extra/testdata
```
## With OpenVINO Inference Engine

When you build G-API with OpenVINO Inference Engine support (`-DInferenceEngine_DIR=...` `-DWITH_INF_ENGINE=ON`), some extra tests for inference are enabled and require `OPENCV_DNN_TEST_DATA_PATH` to be set and **models downloaded** using the command below!

```bash
export OPENCV_DNN_TEST_DATA_PATH=/path/to/opencv_extra/testdata/dnn
openvino$ ./tools/downloader/downloader.py -o ${OPENCV_DNN_TEST_DATA_PATH}/omz_intel_models/2020.3.0 \
    --cache_dir ${OPENCV_DNN_TEST_DATA_PATH}/.omz_cache/ \
    --name age-gender-recognition-retail-0013
```
## With ONNX Runtime

When you build G-API with ONNX Runtime support, tests for inference are enabled and require `OPENCV_GAPI_ONNX_MODEL_PATH` to be set:
```bash
$ export OPENCV_TEST_DATA_PATH=/path-to-opencv-extra/testdata
$ export OPENCV_GAPI_ONNX_MODEL_PATH=/path-to-onnx-models/
```
and [models](https://github.com/onnx/models) are downloaded using the commands:
```bash
$ git clone --recursive https://github.com/onnx/models.git
$ cd models
$ git lfs pull --include=path-to-desired-onnx-model --exclude=""
```

Submitting G-API PRs
====================

G-API supports so-called STANDALONE mode build which is not validated by default.
Please put the below lines to the PR description to validate that this mode is not broken by the PR:

```
force_builders=Custom,Custom Win,Custom Mac
build_gapi_standalone:Linux x64=ade-0.1.1f
build_gapi_standalone:Win64=ade-0.1.1f
build_gapi_standalone:Mac=ade-0.1.1f
build_gapi_standalone:Linux x64 Debug=ade-0.1.1f

Xbuild_image:Custom=centos:7
Xbuildworker:Custom=linux-1
build_gapi_standalone:Custom=ade-0.1.1f

build_image:Custom=ubuntu-openvino-2021.4.0:20.04
build_image:Custom Win=openvino-2021.4.0
build_image:Custom Mac=openvino-2021.4.0

test_modules:Custom=gapi,python2,python3,java
test_modules:Custom Win=gapi,python2,python3,java
test_modules:Custom Mac=gapi,python2,python3,java

buildworker:Custom=linux-1
# disabled due high memory usage: test_opencl:Custom=ON
test_opencl:Custom=OFF
test_bigdata:Custom=1
test_filter:Custom=*
```

Notes:
- ADE version may change, refer to the latest correct one (see `DownloadADE.cmake`).

G-API build depends on a number of external components which may not be built by default, e.g. [PlaidML](https://github.com/plaidml/plaidml):

```
force_builders=Custom
buildworker:Custom=linux-1
build_image:Custom=plaidml2
test_modules:Custom=gapi
test_filter:Custom=*ML*
```

or [ONNX Runtime](https://github.com/microsoft/onnxruntime):

```
force_builders=Custom
buildworker:Custom=linux-1
build_image:Custom=ubuntu-onnx:20.04
test_modules:Custom=gapi
test_filter:Custom=*ONNX*
```

or [FreeType](https://www.freetype.org/):

```
force_builders=Custom
buildworker:Custom=linux-1
build_image:Custom=ubuntu-gapi-freetype:16.04
```

It is also worth testing if any internal API changes are made.

Materials
============

* A [tutorial](https://docs.opencv.org/4.0.0/df/d7e/tutorial_table_of_content_gapi.html) and some [documentation chapters](https://docs.opencv.org/4.0.0/d0/d1e/gapi.html) are already available!

* Check out [these slides](files/2020-09-04-GAPI_Overview.pdf) as a compact introduction to G-API.
