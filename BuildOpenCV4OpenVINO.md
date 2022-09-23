# OpenCV usage with OpenVINO

# Contents
- [Introduction](#introduction)
- [Approach #1: Install prebuilt OpenCV](#approach-1-install-prebuilt-opencv)
   - [Installing on Ubuntu](#installing-on-ubuntu)
   - [Installing on RHEL](#installing-on-rhel)
   - [Installing on Windows](#installing-on-windows)
   - [Installing on macOS](#installing-on-macos)
- [Approach #2: Build OpenCV against specific version of OpenVINO](#approach-2-build-opencv-against-specific-version-of-openvino)
   - [Building on Ubuntu](#building-on-ubuntu)
   - [Building on RHEL](#building-on-rhel)
   - [Building on Windows](#building-on-windows)
   - [Building on macOS](#building-on-macos)

# Introduction
OpenVINO does not provide custom OpenCV drop since 2022.1.1 release.
If OpenVINO user needs OpenCV functionality there are 2 approaches how to get it:
1. Install prebuilt OpenCV from another sources (system repositories, pip, conda, homebrew). It is easy to follow this approach, however it has several disadvantages:
   * OpenCV version is out-of-date  
   * OpenCV does not contain G-API module (e.g. some OMZ demos use G-API functionality)
   * OpenCV does not use available CPU instructions since it has build to cover wide range of hardware
   * OpenCV does not support Intel TBB, Intel Media SDK
   * OpenCV DNN module can not use OpenVINO as computational backend
2. Build OpenCV from source code against specific version of OpenVINO. This approach solves the issues mentioned above.

# Approach #1: Install prebuilt OpenCV

## Installing on Ubuntu
OpenCV is available from the default Ubuntu repositories:
> sudo apt update  
  sudo apt install libopencv-dev python3-opencv

## Installing on RHEL

OpenCV is available from the default RHEL repositories. 
1. Enable CodeReady Linux Builder Repository: 
> subscription-manager repos --enable codeready-builder-for-rhel-8-x86_64-rpms
2. Install OpenCV:
> sudo yum update  
  sudo yum install opencv opencv-devel opencv-python

## Installing on Windows

1. Download OpenCV windows package on [the official OpenCV releases page](https://github.com/opencv/opencv/releases)
Windows archive contains VC postfix, e.g.: `opencv-4.6.0-vc14_vc15.exe`
2. Extract the archive
3. Setup environment variables by running the script `opencv\build\setup_vars_opencv4.cmd`

## Installing on macOS
OpenCV could be installed via Homebrew:
> brew install opencv

# Approach #2: Build OpenCV against specific version of OpenVINO

  > **NOTE**: The instruction below describes how to build OpenCV with the same compilation flags and dependencies as were used for OpenVINO OpenCV build before 2022.1.1 release.  
If you would like to use a minimal set of compilation flags and dependencies to build OpenCV compatible with OpenVINO please follow [another instruction](https://github.com/opencv/opencv/wiki/Intel-OpenVINO-backend#build-opencv-from-source).

## Building on Ubuntu

### Prerequisites 
1. Install OpenVINO according to the [instruction](https://docs.openvino.ai/latest/openvino_docs_install_guides_installing_openvino_linux.html)
2. Install the following packages:
<details>
  <summary>The command</summary>

> sudo apt-get install \  
build-essential \  
cmake \  
ninja-build \  
libgtk-3-dev \  
libpng-dev \  
libjpeg-dev \  
libwebp-dev \  
libtiff5-dev \  
libopenexr-dev \  
libopenblas-dev \  
libx11-dev \  
libavutil-dev \  
libavcodec-dev \  
libavformat-dev \  
libswscale-dev \  
libavresample-dev \  
libtbb2 \  
libssl-dev \  
libva-dev \  
libmfx-dev \  
libgstreamer1.0-dev \  
libgstreamer-plugins-base1.0-dev 
</details>

### Procedure
0. Setup environment variables to detect OpenVINO:
> source /opt/intel/openvino/bin/setupvars.sh
1. Copy OpenCV repository:
> git clone --recurse-submodules https://github.com/opencv/opencv.git
2. Create build directory and enter into it:
> mkdir ~/build-opencv && cd ~/build-opencv
3. Compile and install OpenCV:
<details>
  <summary>The command</summary>

> cmake -G Ninja \  
-D BUILD_INFO_SKIP_EXTRA_MODULES=ON \  
-D BUILD_EXAMPLES=OFF \  
-D BUILD_JASPER=OFF \  
-D BUILD_JAVA=OFF \  
-D BUILD_JPEG=ON \  
-D BUILD_APPS_LIST=version \  
-D BUILD_opencv_apps=ON \  
-D BUILD_opencv_java=OFF \  
-D BUILD_OPENEXR=OFF \  
-D BUILD_PNG=ON \  
-D BUILD_TBB=OFF \  
-D BUILD_WEBP=OFF \  
-D BUILD_ZLIB=ON \  
-D WITH_1394=OFF \  
-D WITH_CUDA=OFF \  
-D WITH_EIGEN=OFF \  
-D WITH_GPHOTO2=OFF \  
-D WITH_GSTREAMER=ON \  
-D OPENCV_GAPI_GSTREAMER=OFF \  
-D WITH_GTK_2_X=OFF \  
-D WITH_IPP=ON \  
-D WITH_JASPER=OFF \  
-D WITH_LAPACK=OFF \  
-D WITH_MATLAB=OFF \  
-D WITH_MFX=ON \  
-D WITH_OPENCLAMDBLAS=OFF \  
-D WITH_OPENCLAMDFFT=OFF \  
-D WITH_OPENEXR=OFF \  
-D WITH_OPENJPEG=OFF \  
-D WITH_QUIRC=OFF \  
-D WITH_TBB=OFF \  
-D WITH_TIFF=OFF \  
-D WITH_VTK=OFF \  
-D WITH_WEBP=OFF \  
-D CMAKE_USE_RELATIVE_PATHS=ON \  
-D CMAKE_SKIP_INSTALL_RPATH=ON \  
-D ENABLE_BUILD_HARDENING=ON \  
-D ENABLE_CONFIG_VERIFICATION=ON \  
-D ENABLE_PRECOMPILED_HEADERS=OFF \  
-D ENABLE_CXX11=ON \  
-D INSTALL_PDB=ON \  
-D INSTALL_TESTS=ON \  
-D INSTALL_C_EXAMPLES=ON \  
-D INSTALL_PYTHON_EXAMPLES=ON \  
-D CMAKE_INSTALL_PREFIX=install \  
-D OPENCV_SKIP_PKGCONFIG_GENERATION=ON \  
-D OPENCV_SKIP_PYTHON_LOADER=OFF \  
-D OPENCV_SKIP_CMAKE_ROOT_CONFIG=ON \  
-D OPENCV_GENERATE_SETUPVARS=OFF \  
-D OPENCV_BIN_INSTALL_PATH=bin \  
-D OPENCV_INCLUDE_INSTALL_PATH=include \  
-D OPENCV_LIB_INSTALL_PATH=lib \  
-D OPENCV_CONFIG_INSTALL_PATH=cmake \  
-D OPENCV_3P_LIB_INSTALL_PATH=3rdparty \  
-D OPENCV_SAMPLES_SRC_INSTALL_PATH=samples \  
-D OPENCV_DOC_INSTALL_PATH=doc \  
-D OPENCV_OTHER_INSTALL_PATH=etc \  
-D OPENCV_LICENSES_INSTALL_PATH=etc/licenses \  
-D OPENCV_INSTALL_FFMPEG_DOWNLOAD_SCRIPT=ON \  
-D BUILD_opencv_world=OFF \  
-D BUILD_opencv_python2=OFF \  
-D BUILD_opencv_python3=ON \  
-D PYTHON3_PACKAGES_PATH=install/python/python3 \  
-D PYTHON3_LIMITED_API=ON \  
-D HIGHGUI_PLUGIN_LIST=all \  
-D OPENCV_PYTHON_INSTALL_PATH=python \  
-D CPU_BASELINE=SSE4_2 \  
-D OPENCV_IPP_GAUSSIAN_BLUR=ON \  
-D WITH_OPENVINO=ON \  
-D VIDEOIO_PLUGIN_LIST=ffmpeg,gstreamer,mfx \  
-D CMAKE_EXE_LINKER_FLAGS=-Wl,--allow-shlib-undefined \  
-D CMAKE_BUILD_TYPE=Release <OpenCV_ROOT_REPO_DIRECTORY> && \  
ninja && cmake --install .
</details>

OpenCV package is available at `~/build-opencv/install` directory. 

To compile application that uses OpenCV, the following environment variables should be specified:
> export OpenCV_DIR="<OpenCV_INSTALL_DIR>/cmake"  
export LD_LIBRARY_PATH="<OpenCV_INSTALL_DIR>/lib${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}"  
export PYTHONPATH="<OpenCV_INSTALL_DIR>/python${PYTHONPATH:+:$PYTHONPATH}"

## Building on RHEL

### Prerequisites 
1. Install OpenVINO according to the [instruction](https://docs.openvino.ai/latest/openvino_docs_install_guides_installing_openvino_docker_linux.html)
2. Install EPEL and RPM Fusion repositories using the following commands:
> yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm https://download1.rpmfusion.org/free/el/rpmfusion-free-release-8.noarch.rpm https://download1.rpmfusion.org/nonfree/el/rpmfusion-nonfree-release-8.noarch.rpm 
3. Install the following packages:
<details>
  <summary>The command</summary>

> sudo yum install \   
cmake \  
gtk3-devel \  
libmfx-devel \  
gstreamer1-devel \  
gstreamer1-plugins-base-devel \  
ffmpeg-devel
</details>

### Procedure
0. Setup environment variables to detect OpenVINO:
> source /opt/intel/openvino/bin/setupvars.sh
1. Copy OpenCV repository:
> git clone --recurse-submodules https://github.com/opencv/opencv.git
2. Create build directory and enter into it:
> mkdir ~/build-opencv && cd ~/build-opencv
3. Compile and install OpenCV:
<details>
  <summary>The command</summary>

> cmake \  
-D BUILD_INFO_SKIP_EXTRA_MODULES=ON \  
-D BUILD_EXAMPLES=OFF \  
-D BUILD_JASPER=OFF \  
-D BUILD_JAVA=OFF \  
-D BUILD_JPEG=ON \  
-D BUILD_APPS_LIST=version \  
-D BUILD_opencv_apps=ON \  
-D BUILD_opencv_java=OFF \  
-D BUILD_OPENEXR=OFF \  
-D BUILD_PNG=ON \  
-D BUILD_TBB=OFF \  
-D BUILD_WEBP=OFF \  
-D BUILD_ZLIB=ON \  
-D WITH_1394=OFF \  
-D WITH_CUDA=OFF \  
-D WITH_EIGEN=OFF \  
-D WITH_GPHOTO2=OFF \  
-D WITH_GSTREAMER=ON \  
-D OPENCV_GAPI_GSTREAMER=OFF \  
-D WITH_GTK_2_X=OFF \  
-D WITH_IPP=ON \  
-D WITH_JASPER=OFF \  
-D WITH_LAPACK=OFF \  
-D WITH_MATLAB=OFF \  
-D WITH_MFX=OFF \  
-D WITH_OPENCLAMDBLAS=OFF \  
-D WITH_OPENCLAMDFFT=OFF \  
-D WITH_OPENEXR=OFF \  
-D WITH_OPENJPEG=OFF \  
-D WITH_QUIRC=OFF \  
-D WITH_TBB=OFF \  
-D WITH_TIFF=OFF \  
-D WITH_VTK=OFF \  
-D WITH_WEBP=OFF \  
-D CMAKE_USE_RELATIVE_PATHS=ON \  
-D CMAKE_SKIP_INSTALL_RPATH=ON \  
-D ENABLE_BUILD_HARDENING=ON \  
-D ENABLE_CONFIG_VERIFICATION=ON \  
-D ENABLE_PRECOMPILED_HEADERS=OFF \  
-D ENABLE_CXX11=ON \  
-D INSTALL_PDB=ON \  
-D INSTALL_TESTS=ON \  
-D INSTALL_C_EXAMPLES=ON \  
-D INSTALL_PYTHON_EXAMPLES=ON \  
-D CMAKE_INSTALL_PREFIX=install \  
-D OPENCV_SKIP_PKGCONFIG_GENERATION=ON \  
-D OPENCV_SKIP_PYTHON_LOADER=OFF \  
-D OPENCV_SKIP_CMAKE_ROOT_CONFIG=ON \  
-D OPENCV_GENERATE_SETUPVARS=OFF \  
-D OPENCV_BIN_INSTALL_PATH=bin \  
-D OPENCV_INCLUDE_INSTALL_PATH=include \  
-D OPENCV_LIB_INSTALL_PATH=lib \  
-D OPENCV_CONFIG_INSTALL_PATH=cmake \  
-D OPENCV_3P_LIB_INSTALL_PATH=3rdparty \  
-D OPENCV_SAMPLES_SRC_INSTALL_PATH=samples \  
-D OPENCV_DOC_INSTALL_PATH=doc \  
-D OPENCV_OTHER_INSTALL_PATH=etc \  
-D OPENCV_LICENSES_INSTALL_PATH=etc/licenses \  
-D OPENCV_INSTALL_FFMPEG_DOWNLOAD_SCRIPT=ON \  
-D BUILD_opencv_world=OFF \  
-D BUILD_opencv_python2=OFF \  
-D BUILD_opencv_python3=ON \  
-D PYTHON3_PACKAGES_PATH=install/python/python3 \  
-D PYTHON3_LIMITED_API=ON \  
-D HIGHGUI_PLUGIN_LIST=all \  
-D OPENCV_PYTHON_INSTALL_PATH=python \  
-D CPU_BASELINE=SSE4_2 \  
-D OPENCV_IPP_GAUSSIAN_BLUR=ON \  
-D WITH_OPENVINO=ON \  
-D VIDEOIO_PLUGIN_LIST=ffmpeg,gstreamer \  
-D CMAKE_EXE_LINKER_FLAGS=-Wl,--allow-shlib-undefined \  
-D CMAKE_BUILD_TYPE=Release <OpenCV_ROOT_REPO_DIRECTORY> && \  
make && cmake -P cmake_install.cmake
</details>

OpenCV package is available at `~/build-opencv/install` directory. 

To compile application that uses OpenCV, the following environment variables should be specified:
> export OpenCV_DIR="<OpenCV_INSTALL_DIR>/cmake"  
export LD_LIBRARY_PATH="<OpenCV_INSTALL_DIR>/lib${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}"  
export PYTHONPATH="<OpenCV_INSTALL_DIR>/python${PYTHONPATH:+:$PYTHONPATH}"


## Building on Windows

### Prerequisites 
1. Install Microsoft Visual Studio 
2. Install [cmake](https://cmake.org/download/)
3. Install OpenVINO according to the [instruction](https://docs.openvino.ai/latest/openvino_docs_install_guides_installing_openvino_windows.html)
4. Install [Intel® Media SDK for Windows](https://www.intel.com/content/www/us/en/developer/tools/media-sdk/choose-download-client.html)

### Procedure
0. Setup environment variables to detect OpenVINO:
> "C:\Program Files (x86)\IntelSWTools\openvino\bin\setupvars.bat"
1. Copy OpenCV repository:
> git clone --recurse-submodules https://github.com/opencv/opencv.git
2. Create build directory and enter into it:
> mkdir "build-opencv" && cd "build-opencv"
3. Setup MSVC environment by running `vcvars64.bat`
4. Compile and install OpenCV:
<details>
  <summary>The command</summary>

> cmake -G Ninja ^  
-DBUILD_INFO_SKIP_EXTRA_MODULES=ON ^  
-DBUILD_EXAMPLES=OFF ^  
-DBUILD_JASPER=OFF ^  
-DBUILD_JAVA=OFF ^  
-DBUILD_JPEG=ON ^  
-DBUILD_APPS_LIST=version ^  
-DBUILD_opencv_apps=ON ^  
-DBUILD_opencv_java=OFF ^  
-DBUILD_OPENEXR=OFF ^  
-DBUILD_PNG=ON ^  
-DBUILD_TBB=OFF ^  
-DBUILD_WEBP=OFF ^  
-DBUILD_ZLIB=ON ^  
-DWITH_1394=OFF ^  
-DWITH_CUDA=OFF ^  
-DWITH_EIGEN=OFF ^  
-DWITH_GPHOTO2=OFF ^  
-DWITH_GSTREAMER=OFF ^  
-DOPENCV_GAPI_GSTREAMER=OFF ^  
-DWITH_GTK_2_X=OFF ^  
-DWITH_IPP=ON ^  
-DWITH_JASPER=OFF ^  
-DWITH_LAPACK=OFF ^  
-DWITH_MATLAB=OFF ^  
-DWITH_MFX=ON ^  
-DWITH_OPENCLAMDBLAS=OFF ^  
-DWITH_OPENCLAMDFFT=OFF ^  
-DWITH_OPENEXR=OFF ^  
-DWITH_OPENJPEG=OFF ^  
-DWITH_QUIRC=OFF ^  
-DWITH_TBB=OFF ^  
-DWITH_TIFF=OFF ^  
-DWITH_VTK=OFF ^  
-DWITH_WEBP=OFF ^  
-DCMAKE_USE_RELATIVE_PATHS=ON ^  
-DCMAKE_SKIP_INSTALL_RPATH=ON ^  
-DENABLE_BUILD_HARDENING=ON ^  
-DENABLE_CONFIG_VERIFICATION=ON ^  
-DENABLE_PRECOMPILED_HEADERS=OFF ^  
-DENABLE_CXX11=ON ^  
-DINSTALL_PDB=ON ^  
-DINSTALL_TESTS=ON ^  
-DINSTALL_C_EXAMPLES=ON ^  
-DINSTALL_PYTHON_EXAMPLES=ON ^  
-DCMAKE_INSTALL_PREFIX=install ^  
-DOPENCV_SKIP_PKGCONFIG_GENERATION=ON ^  
-DOPENCV_SKIP_PYTHON_LOADER=OFF ^  
-DOPENCV_SKIP_CMAKE_ROOT_CONFIG=ON ^  
-DOPENCV_GENERATE_SETUPVARS=OFF ^  
-DOPENCV_BIN_INSTALL_PATH=bin ^  
-DOPENCV_INCLUDE_INSTALL_PATH=include ^  
-DOPENCV_LIB_INSTALL_PATH=lib ^  
-DOPENCV_CONFIG_INSTALL_PATH=cmake ^  
-DOPENCV_3P_LIB_INSTALL_PATH=3rdparty ^  
-DOPENCV_SAMPLES_SRC_INSTALL_PATH=samples ^  
-DOPENCV_DOC_INSTALL_PATH=doc ^  
-DOPENCV_OTHER_INSTALL_PATH=etc ^  
-DOPENCV_LICENSES_INSTALL_PATH=etc/licenses ^  
-DOPENCV_INSTALL_FFMPEG_DOWNLOAD_SCRIPT=ON ^  
-DBUILD_opencv_world=OFF ^  
-DBUILD_opencv_python2=OFF ^  
-DBUILD_opencv_python3=ON ^  
-DPYTHON3_PACKAGES_PATH=install/python/python3 ^  
-DPYTHON3_LIMITED_API=ON ^  
-DOPENCV_PYTHON_INSTALL_PATH=python ^  
-DCPU_BASELINE=SSE4_2 ^  
-DOPENCV_IPP_GAUSSIAN_BLUR=ON ^  
-DWITH_OPENVINO=ON ^   
-DVIDEOIO_PLUGIN_LIST=mfx,msmf ^  
-DCMAKE_BUILD_TYPE=Release <OpenCV_ROOT_REPO_DIRECTORY> &&  
ninja &&  
cmake --install .
</details>

OpenCV package is available at `build-opencv/install` directory. 

To compile application that uses OpenCV, the following environment variables should be specified:
>set "OpenCV_DIR=<OpenCV_INSTALL_DIR>\cmake"  
set "PATH=<OpenCV_INSTALL_DIR>\bin;%PATH%"  
set "PYTHONPATH=<OpenCV_INSTALL_DIR>\python;%PYTHONPATH%"  

## Building on macOS

### Prerequisites 
1. Install OpenVINO according to the [instruction](https://docs.openvino.ai/latest/openvino_docs_install_guides_installing_openvino_from_archive_macos.html#doxid-openvino-docs-install-guides-installing-openvino-from-archive-macos)
2. Install the following packages: 
> brew install cmake \  
 ninja \  
 ffmpeg \  
 gstreamer \  
 gst-devtools  \
 python \  
 numpy  

### Procedure
0. Setup environment variables to detect OpenVINO:
> source /opt/intel/openvino/bin/setupvars.sh
1. Copy OpenCV repository:
> git clone --recurse-submodules https://github.com/opencv/opencv.git
2. Create build directory and enter into it:
> mkdir "build-opencv" && cd "build-opencv"
3. Compile and install OpenCV:
<details>
  <summary>The command</summary>

> cmake \  
-G Ninja \  
-DBUILD_INFO_SKIP_EXTRA_MODULES=ON \  
-DBUILD_EXAMPLES=OFF \  
-DBUILD_JASPER=OFF \  
-DBUILD_JAVA=OFF \  
-DBUILD_JPEG=ON \  
-DBUILD_APPS_LIST=version \  
-DBUILD_opencv_apps=ON \  
-DBUILD_opencv_java=OFF \  
-DBUILD_OPENEXR=OFF \  
-DBUILD_PNG=ON \  
-DBUILD_TBB=OFF \  
-DBUILD_WEBP=OFF \  
-DBUILD_ZLIB=ON \  
-DWITH_1394=OFF \  
-DWITH_CUDA=OFF \  
-DWITH_EIGEN=OFF \  
-DWITH_GPHOTO2=OFF \  
-DWITH_GSTREAMER=ON \  
-DOPENCV_GAPI_GSTREAMER=OFF \  
-DWITH_GTK_2_X=OFF \  
-DWITH_IPP=ON \  
-DWITH_JASPER=OFF \  
-DWITH_LAPACK=OFF \  
-DWITH_MATLAB=OFF \  
-DWITH_MFX=OFF \  
-DWITH_OPENCLAMDBLAS=OFF \  
-DWITH_OPENCLAMDFFT=OFF \  
-DWITH_OPENEXR=OFF \  
-DWITH_OPENJPEG=OFF \  
-DWITH_QUIRC=OFF \  
-DWITH_TBB=OFF \  
-DWITH_TIFF=OFF \  
-DWITH_VTK=OFF \  
-DWITH_WEBP=OFF \  
-DCMAKE_USE_RELATIVE_PATHS=ON \  
-DCMAKE_SKIP_INSTALL_RPATH=ON \  
-DENABLE_BUILD_HARDENING=ON \  
-DENABLE_CONFIG_VERIFICATION=ON \  
-DENABLE_PRECOMPILED_HEADERS=OFF \  
-DENABLE_CXX11=ON \  
-DINSTALL_PDB=ON \  
-DINSTALL_TESTS=ON \  
-DINSTALL_C_EXAMPLES=ON \  
-DINSTALL_PYTHON_EXAMPLES=ON \  
-DCMAKE_INSTALL_PREFIX=install \  
-DOPENCV_SKIP_PKGCONFIG_GENERATION=ON \  
-DOPENCV_SKIP_PYTHON_LOADER=OFF \  
-DOPENCV_SKIP_CMAKE_ROOT_CONFIG=ON \  
-DOPENCV_GENERATE_SETUPVARS=OFF \  
-DOPENCV_BIN_INSTALL_PATH=bin \  
-DOPENCV_INCLUDE_INSTALL_PATH=include \  
-DOPENCV_LIB_INSTALL_PATH=lib \  
-DOPENCV_CONFIG_INSTALL_PATH=cmake \  
-DOPENCV_3P_LIB_INSTALL_PATH=3rdparty \  
-DOPENCV_SAMPLES_SRC_INSTALL_PATH=samples \  
-DOPENCV_DOC_INSTALL_PATH=doc \  
-DOPENCV_OTHER_INSTALL_PATH=etc \  
-DOPENCV_LICENSES_INSTALL_PATH=etc/licenses \  
-DOPENCV_INSTALL_FFMPEG_DOWNLOAD_SCRIPT=ON \  
-DBUILD_opencv_world=OFF \  
-DBUILD_opencv_python2=OFF \  
-DBUILD_opencv_python3=ON \  
-DPYTHON3_PACKAGES_PATH=install/python/python3 \  
-DPYTHON3_LIMITED_API=ON \  
-DOPENCV_PYTHON_INSTALL_PATH=python \  
-DCPU_BASELINE=SSE4_2 \  
-DWITH_AVFOUNDATION=ON \  
-DOPENCV_IPP_GAUSSIAN_BLUR=ON \  
-DWITH_OPENVINO=ON \  
-DVIDEOIO_PLUGIN_LIST=ffmpeg,gstreamer \  
-DCMAKE_BUILD_TYPE=Release <OpenCV_ROOT_REPO_DIRECTORY> &&  
ninja &&  
cmake --install .
</details>

OpenCV package is available at `build-opencv/install` directory. 

To compile application that uses OpenCV, the following environment variables should be specified:
> export OpenCV_DIR="<OpenCV_INSTALL_DIR>/cmake"  