## Building with OpenVINO Toolkit support

* Get `openvino` from https://docs.openvinotoolkit.org/
* Configure enviroment:
```bash
source path-to-unpacked-openvino/setupvars.sh
```
* Build G-API with OpenVINO support:
```bash
cmake /path-to-opencv -DWITH_INF_ENGINE=ON
```

*NOTE*: You may need to set `INF_ENGINE_RELEASE` to the proper version, depending on the package you use.

## Testing with OpenVINO Inference Engine

When you build G-API with OpenVINO Inference Engine support (`-DInferenceEngine_DIR=...` `-DWITH_INF_ENGINE=ON`), some extra tests for inference are enabled and require `OPENCV_DNN_TEST_DATA_PATH` to be set and **models downloaded** using the command below!

```bash
$ git clone https://github.com/openvinotoolkit/open_model_zoo.git
$ cd open_model_zoo/tools/model_tools
$ ./downloader.py -o /path/to/opencv_extra/testdata/dnn/omz_intel_models/ \
    --cache_dir /path/to/opencv_extra/testdata/dnn/.omz_cache/ \
    --name age-gender-recognition-retail-0013
$ export OPENCV_DNN_TEST_DATA_PATH=/path/to/opencv_extra/testdata/dnn
```

Note that the `downloader.py` script has the following dependencies:

```
pyyaml
requests
```