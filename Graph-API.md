What is it?
===========

* OpenCV 4.0 introduced an experimental Graph API module (see [opencv/modules/gapi](https://github.com/opencv/opencv/tree/master/modules/gapi)). This is a new API which allows to enable offload and optimizations for image processing / CV algorithms on pipeline level.

* The idea behind G-API is to declare image processing task in form of expressions and then submit it for execution – using a number of available backends. At the moment, there’s reference “CPU” (OpenCV-based), "GPU" (also OpenCV T-API-based), and experimental “Fluid’ backends available, with other backends coming up next.

* G-API is an uncommon OpenCV module since it acts as a framework: it provides means for declaring operations, building graphs of operations, and finally implementing the operations for a particular backend. G-API model enforces separation between interfaces and implementations, so once an algorithm is expressed in G-API terms, it can be scaled/ported/offloaded to a new platform easily.

## Quick tour of G-API Backends

* G-API **CPU** (OpenCV) backend implements G-API standard functions using OpenCV itself (core/imgproc modules) and acts as a quick prototyping/porting/testing backend. If you have an image processing algorithm composed of OpenCV-like functions already, you would be able to switch quickly to G-API by using this backend.

* G-API **Fluid** backend implements a cache-efficient execution model and allows to save memory dramatically – e.g. a 1.5GB image processing pipeline fits into 750KB memory footprint with G-API/Fluid. G-API comes with a number of operations implemented for Fluid backend, so one can switch OpenCV/Fluid operations within a graph easily and even mix both in the same graph.

* G-API **OpenCL** (also known as GPU) backend implements the majority of available functions and allows to run OpenCL kernels on available OpenCL-programmable devices. At the moment, GPU backend is based on OpenCV Transparent API; in future versions it will be extended to support integration of arbitrary OpenCL kernels (and likely be renamed to "OpenCL backend").

* G-API **OpenVINO** backend brings [OpenVINO](https://docs.openvino.ai/latest/home.html) inference support in G-API. The OpenVINO backend is the most function-complete G-API inference backend at the moment, utilizing advanced features as OpenVINO-based preprocessing, remote memory and context support, and more.

* G-API **ONNX** backend implements [ONNX models](https://github.com/onnx/models) inference operations on input data and outputs the results. At the moment, ONNX backend is based on [ONNX Runtime](https://github.com/microsoft/onnxruntime) C/C++ API, build & run instruction can be found [here](https://github.com/opencv/opencv/wiki/Using-G-API-with-MS-ONNX-Runtime).

Note you can find more backends in the G-API's source directory. Some of those are used implicitly, and some are experimental and provide specific functionality.

# Building G-API

G-API is built with OpenCV by default, however some features may require additional options or dependencies enabled.
Refer to the following instructions for details:

- [[G-API Python bindings|G-API-Python-Bindings]]
- [[Using G-API with OpenVINO Toolkit|Using-G-API-with-OpenVINO-Toolkit]]
- [[Using G-API with MS ONNX Runtime|Using-G-API-with-MS-ONNX-Runtime]]
- [[Using G API with PlaidML backend|Using-G-API-with-PlaidML-backend]]
- [[Using G API with OpenCV AI Kit (OAK)|Using-G-API-with-OpenCV-AI-Kit-(OAK)]]
- [[Enabling GStreamer source in G-API|Enabling-GStreamer-source-in-G-API]]
- [[Enabling oneVPL source in G-API|Enabling-oneVPL-source-in-G-API]]

Testing G-API
=============

By default, the OpenCV G-API comes with its own test suite (`opencv_test_gapi`). Note that extra (external) G-API modules may introduce their own test suites. G-API tests are built and run in a regular way:

## Requirements

Some G-API tests require test data to be available. This data is taken from the [opencv_extra](http://github.com/opencv/opencv_extra) repo. You have to set the `OPENCV_TEST_DATA_PATH` environment variable to avoid failed tests (due to absence of test data):

```
export OPENCV_TEST_DATA_PATH=/Linux/path/to/opencv_extra/testdata
```
or
```
SET OPENCV_TEST_DATA_PATH=\Windows\path\to\opencv_extra\testdata
```
Some tests require **external** test data to be available. This is test data not included in [opencv_extra](http://github.com/opencv/opencv_extra). ONNX models are an example. The absence of this data doesn't break the tests. Tests are skipped without **external** test data. See the respective subpages to find more details.

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

Materials
============

* A [tutorial](https://docs.opencv.org/4.0.0/df/d7e/tutorial_table_of_content_gapi.html) and some [documentation chapters](https://docs.opencv.org/4.0.0/d0/d1e/gapi.html) are already available!

* Check out [these slides](files/2020-09-04-GAPI_Overview.pdf) as a compact introduction to G-API.
