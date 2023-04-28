# Building G-API with Microsoft ONNX Runtime support

* Build and install the ONNX RT (currently tested with v1.14.1):

  ```bash
  $ git clone --recursive https://github.com/microsoft/onnxruntime.git
  $ cd onnxruntime
  $ git checkout v1.14.1
  $ git submodule update --init
  $ ./build.sh --config Release --build_shared_lib --parallel \
  $     --cmake_extra_defines CMAKE_INSTALL_PREFIX=/path-to-install-dir
  $ cd build/Linux/Release
  $ make install
  ```

* Then specify extra options to OpenCV CMake:    

  ```bash
  $ cmake /path-to-opencv -DWITH_ONNX=ON -DORT_INSTALL_DIR=/path-to-ort-install-dir
  ```

# Running G-API ONNX Tests

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

The following models are known to be required as for OpenCV 4.7:
```bash
git lfs pull --include vision/body_analysis/emotion_ferplus/model/emotion-ferplus-8.onnx --exclude=""
git lfs pull --include vision/classification/squeezenet/model/squeezenet1.0-9.onnx --exclude=""
git lfs pull --include vision/object_detection_segmentation/faster-rcnn/model/FasterRCNN-10.onnx --exclude=""
git lfs pull --include vision/object_detection_segmentation/ssd-mobilenetv1/model/ssd_mobilenet_v1_10.onnx --exclude=""
git lfs pull --include vision/object_detection_segmentation/tiny-yolov2/model/tinyyolov2-8.onnx --exclude=""
git lfs pull --include vision/object_detection_segmentation/yolov3/model/yolov3-10.onnx --exclude=""
```
