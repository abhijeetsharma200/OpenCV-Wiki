# Intro

G-API is also available in OpenCV for Python. This page explains how to build and test G-API in Python.

## Requirements

* Python3+ with `numpy`
* [OpenVINO](Using-G-API-with-OpenVINO-Toolkit) - for inference tests with OpenVINO backend;
* [ONNX RT](Using-G-API-with-MS-ONNX-Runtime) - for inference tests with ONNX Runtime backend;
* Working `videoio` module - built with either FFMPEG or [GStreamer](Enabling-GStreamer-source-in-G-API) support - as some tests are using video files as input.

## Building

Use this CMake option to enable Python (make sure to include options for OpenVINO and ONNX RT if necessary):

```bash
$ cmake /path/to/opencv -DWITH_PYTHON=ON -DCMAKE_BUILD_TYPE=Release
$ make -j8 opencv_python3
```

## Testing

Setup environment:

```bash
$ export LD_LIBRARY_PATH=<path-to-opencv-build>/lib/:$LD_LIBRARY_PATH
$ export PYTHONPATH=<path-to-opencv-build>/python_loader/:$PYTHONPATH
$ export OPENCV_TEST_DATA_PATH=<path-to-opencv_extra>/testdata/
$ export OPENCV_DNN_TEST_DATA_PATH=<path-to-opencv_extra>/testdata/dnn
$ export OPENCV_GAPI_ONNX_MODEL_PATH=<path-to-onnx-zoo>                 # Required if built with ONNX support
```

Now run G-API tests:

```bash
$ cd <path-to-opencv-build>
$ OPENCV_PYTEST_FILTER=test_gapi* python3 <path-to-opencv-root>/modules/python/test/test.py
```


