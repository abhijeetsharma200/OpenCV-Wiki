[Intel's Deep Learning Inference Engine](https://software.intel.com/en-us/inference-engine-devguide) is a part of 
[Intel&reg; Computer Vision SDK](https://software.intel.com/en-us/computer-vision-sdk). You can use it as a computational backend for OpenCV deep learning module.

* Download and install [Intel's Deep Learning Deployment Toolkit](https://software.seek.intel.com/deep-learning-deployment).

* Build OpenCV specifying path to installed libraries and plugins.

  * Ubuntu
  ```
  cmake \
    -DWITH_INF_ENGINE=ON \
    -DINTEL_CVSDK_DIR=/opt/intel/deeplearning_deploymenttoolkit/deployment_tools/ \
    -DIE_PLUGINS_PATH=/opt/intel/deeplearning_deploymenttoolkit/deployment_tools/inference_engine/lib/ubuntu_16.04/intel64/ \
    -DENABLE_CXX11=ON \
    ...
  ```

  * Microsoft Windows
  ```
  cmake ^
    -DWITH_INF_ENGINE=ON ^
    -DINTEL_CVSDK_DIR=C:\\Intel\\DeepLearning-DeploymentToolkit_1.0.5852 ^
    -DIE_PLUGINS_PATH=C:\\Intel\\DeepLearning-DeploymentToolkit_1.0.5852\\lib\\intel64\\Release ^
    -DENABLE_CXX11=ON ^
    ...
  ```

  Add path to Intel's Inference Engine plugins into PATH variable:
  ```
  set PATH=C:\Intel\DeepLearning-DeploymentToolkit_1.0.5852\bin\intel64\Release;%PATH% 
  ```
* Enable Intel's Inference Engine backend right after `cv::dnn::readNetFrom*` invocation:
  ```cpp
  net.setPreferableBackend(DNN_BACKEND_INFERENCE_ENGINE);
  ```