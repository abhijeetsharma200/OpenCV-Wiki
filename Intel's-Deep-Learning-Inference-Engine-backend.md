[Intel's Deep Learning Inference Engine](https://software.intel.com/inference-engine-devguide) (DL IE) is a part of [Intel&reg; OpenVINO&trade; toolkit](https://software.intel.com/openvino-toolkit). You can use it as a computational backend for OpenCV deep learning module.

### Download
* Download and install [Intel&reg; OpenVINO&trade; toolkit](https://software.seek.intel.com/openvino-toolkit). 

  **Important note**: if you want to transfer the installed Inference Engine binaries to another machine w/o running OpenVINO installer there, you need [the redistributable files of Intel C++ compiler](https://software.intel.com/en-us/articles/intel-compilers-redistributable-libraries-by-version) (use the latest update, 64-bit version), otherwise the Inference Engine or some of its essential plugins will refuse to load and run, which may result in an app crash.

### OpenCV from OpenVINO
OpenVINO 2018 R2 and later comes with OpenCV built with DL IE support, so you can skip this step if you use fresh enough version.

### Build OpenCV from source

#### Ubuntu

Setup environment variables to detect Inference Engine:
```
source /opt/intel/openvino/bin/setupvars.sh
export ngraph_DIR=/opt/intel/openvino/deployment_tools/ngraph/cmake/
```

Build OpenCV with extra flags:
```
cmake \
  -DWITH_INF_ENGINE=ON \
  -DENABLE_CXX11=ON \
  ...
```

#### Microsoft Windows

Setup environment variables to detect Inference Engine:
```
"C:\Program Files (x86)\IntelSWTools\openvino\bin\setupvars.bat"
```

Build OpenCV with extra flags:
```
cmake ^
  -DWITH_INF_ENGINE=ON ^
  -DENABLE_CXX11=ON ^
  ...
```

#### Raspbian Buster

Use Docker to cross-compile OpenCV for Raspberry Pi. Check that `uname -m` detects `armv7l` CPU architecture (starts from Raspberry Pi 2 model B).

1. Create a folder named `debian_armhf` with a file `Dockerfile` with the following content:

  ```docker
  FROM debian:buster

  USER root

  RUN dpkg --add-architecture armhf && \
      apt-get update && \
      apt-get install -y --no-install-recommends \
      crossbuild-essential-armhf \
      cmake \
      pkg-config \
      wget \
      xz-utils \
      libgtk2.0-dev:armhf \
      libpython-dev:armhf \
      libpython3-dev:armhf \
      python-numpy \
      python3-numpy \
      libgstreamer1.0-dev:armhf \
      libgstreamer-plugins-base1.0-dev:armhf

  # Install Inference Engine
  RUN wget --no-check-certificate https://download.01.org/opencv/2019/openvinotoolkit/R2/l_openvino_toolkit_runtime_raspbian_p_2019.2.242.tgz && \
      tar -xf l_openvino_toolkit_runtime_raspbian_p_2019.2.242.tgz
  ```

2. Build a Docker image

  ```bash
  docker image build -t debian_armhf debian_armhf
  ```

3. Run Docker container mounting source code folder from host.

  ```bash
  docker run -it -v /absolute/path/to/opencv:/opencv debian_armhf /bin/bash
  ```

4. Build

  ```bash
  cd opencv && mkdir opencv_build && mkdir opencv_install && cd opencv_build
  cmake -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_INSTALL_PREFIX="../opencv_install" \
        -DOPENCV_CONFIG_INSTALL_PATH="cmake" \
        -DCMAKE_TOOLCHAIN_FILE="../platforms/linux/arm-gnueabi.toolchain.cmake" \
        -DWITH_IPP=OFF \
        -DBUILD_TESTS=OFF \
        -DBUILD_PERF_TESTS=OFF \
        -DOPENCV_ENABLE_PKG_CONFIG=ON \
        -DPKG_CONFIG_EXECUTABLE="/usr/bin/arm-linux-gnueabihf-pkg-config" \
        -DPYTHON2_INCLUDE_PATH="/usr/include/python2.7" \
        -DPYTHON2_NUMPY_INCLUDE_DIRS="/usr/local/lib/python2.7/dist-packages/numpy/core/include" \
        -DPYTHON3_INCLUDE_PATH="/usr/include/python3.7" \
        -DPYTHON3_NUMPY_INCLUDE_DIRS="/usr/local/lib/python3.7/dist-packages/numpy/core/include" \
        -DPYTHON3_CVPY_SUFFIX=".cpython-37m-arm-linux-gnueabihf.so" \
        -DENABLE_NEON=ON \
        -DCPU_BASELINE="NEON" \
        -DWITH_INF_ENGINE=ON \
        -DINF_ENGINE_LIB_DIRS="/l_openvino_toolkit_runtime_raspbian_p_2019.2.242/inference_engine/lib/armv7l" \
        -DINF_ENGINE_INCLUDE_DIRS="/l_openvino_toolkit_runtime_raspbian_p_2019.2.242/inference_engine/include" \
        -DCMAKE_FIND_ROOT_PATH="/l_openvino_toolkit_runtime_raspbian_p_2019.2.242" \
        -DENABLE_CXX11=ON ..
  make -j4 && make install
  ```

5. Copy `opencv_install` to the board. Follow http://docs.openvinotoolkit.org/latest/_docs_install_guides_installing_openvino_raspbian.html to install OpenVINO distribution for Raspberry Pi. Then type the following commands to specify new location of OpenCV:

  ```bash
  export PYTHONPATH=/path/to/opencv_install/lib/python2.7/dist-packages/:$PYTHONPATH
  export PYTHONPATH=/path/to/opencv_install/lib/python3.7/dist-packages/:$PYTHONPATH
  export LD_LIBRARY_PATH=/path/to/opencv_install/lib/:$LD_LIBRARY_PATH
  ```

### Usage

* Enable Intel's Inference Engine backend right after `cv::dnn::readNet` invocation:
  ```cpp
  net.setPreferableBackend(DNN_BACKEND_INFERENCE_ENGINE);
         // the other possible options are
         // DNN_BACKEND_OPENCV (the default C++ implementation)
         // DNN_BACKEND_HALIDE (Halide-based implementation)
  ```
  note that the inference engine backend is used by default since OpenCV 3.4.2 (OpenVINO 2018.R2) when OpenCV is built with the Inference engine support, so the call above is not necessary. Also, the Inference engine backend is the only available option (also enabled by default) when the loaded model is represented in OpenVINO&trade; Model Optimizer format.

* Then, optionally you can also set the device to use for the inference (by default it will use CPU):
  ```cpp
  net.setPreferableTarget(DNN_TARGET_OPENCL);
         // the possible options are
         // DNN_TARGET_CPU,
         // DNN_TARGET_OPENCL, 
         // DNN_TARGET_OPENCL_FP16
         //   (fall back to OPENCL if the hardware does not support FP16),
         // DNN_TARGET_MYRIAD
  ```

* You may also import [pre-trained models](http://docs.openvinotoolkit.org/latest/_docs_Pre_Trained_Models.html) from [Open Model Zoo](https://github.com/opencv/open_model_zoo) passing paths to `.bin` and `.xml` files to `cv::dnn::readNet` function.

### Troubleshooting

* OpenVINO 2018R5 and older:

  Set `OMP_WAIT_POLICY` to `PASSIVE` to prevent efficiency gaps due networks partitioning if there are layers unsupported by Inference Engine backend or you have several networks with preferable backend `DNN_TARGET_CPU`.
