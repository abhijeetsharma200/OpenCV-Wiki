Previous Change Logs:
* [OpenCV v2.2-v4.10](OpenCV-Change-Logs-v2.2%E2%80%90v4.10)
* [OpenCV v1.0-v2.1](OpenCV-Change-Logs-v1.0%E2%80%90v2.1)

# version:5.0-alpha

*December, 2024*

The alpha release for the new OpenCV generation. The release is designed as technology preview and not ready for production usage yet.

The following notation is used to mark various items below:
* 5+4.x: the feature is not quite new, it's already put into the latest 4.x, but it's important enough to mention it here as well.
* 5.0: the item will be finished in 5.0 release; currently it's planned or in progress.
* 5.x: the item will be put in some form into 5.0 release, but will mostly be finished in subsequent 5.x releases.

## Release highlights

### General changes, improvements

- 5+4.x: Recent OpenCV 4.x (since OpenCV 4.5.0) and also OpenCV 5.0 are now distributed under Apache 2 license. We used BSD license before, but switched to Apache 2, because the latter is more refined and provides somewhat better protection (not ideal though) from patent-related issues.
- OpenCV now requires C++ 17 as a minimum C++ standard. It is built by default with C++ 17, but we also plan to make it compatible with C++ 20 and C++ 23.
- Python 2 support is removed. OpenCV now requires Python 3 (3.6+) and we only build Python 3 bindings.

### Big OpenCV cleanup

- C API has been removed. We still use some `CV_` macros, such as `CV_8U`, but all C functions (like `cvCreateMat()` or `cvFindContours()`) and structures (such as `CvMat`) have been removed. Goodbye, good old OpenCV 1.x API, RIP :)
- OpenVX support has been removed. If some vendors provide OpenVX kernels and want to use them for OpenCV acceleration, they can create a custom “non-CPU” HAL for that, see below.
- `Graph API` (`G-API`) module has been moved to `opencv_contrib`.
- Classic `ML` module has been moved to `opencv_contrib`. If you use Python, [scikit-learn](https://scikit-learn.org/) is a much better alternative.
- `Features2D` module has been renamed to `Features`. Scope of the renamed module has been extended to process feature vectors produced by the modern deep nets. Several obsolete feature detectors/descriptors have been moved to opencv_contrib. `SIFT`, `ORB`, `FAST`, `GoodFeaturesToTrack`, `MSER` are still available, though.
- 5.0: `FLANN` as a separate module will be gone. Annoy-based ANN (*approximate nearest neighbor*) search algorithm, which has been already added to `Features` module, will replace it.
- Cleaned `objdetect` module: Haar-based and HOG-based detectors have been moved to `opencv_contrib`, `xobjdetect` module. There are modern deep learning-based object/face detectors that are both faster and more accurate.
- `Calib3d` module has been split into 3 modules:
  - `3d` — basic 3D geometry and 3D vision functionality
  - `calib` — camera calibration
  - `stereo` — algorithms for depth map estimation via stereo correspondence.
- Many obsolete samples (~50% of C++ samples and 5% of Python samples) have been removed. Many of the remaining samples have been revised. 

### Updated Core module

- OpenCV now supports the extended set of data types:
  - `uint8_t: CV_8U`, `int8_t: CV_8S`, `uint16_t: CV_16U`, `int16_t: CV_16S`, `int32_t: CV_32S`, `float: CV_32F`, `double: CV_64F`, `hfloat` (a.k.a. `half` or `float16_t` or `__fp16`): `CV_16F`.
  - (new in OpenCV 5): `bfloat` (a.k.a. `bfloat16_t`): `CV_16BF`, `uint32_t: CV_32U`, `uint64_t: CV_64U`, `int64_t: CV_64S`, `bool: CV_Bool`.
  - `bool` type takes 1 byte per value, not 1 bit. Any non-zero byte is considered `true`, zero byte means `false`. `cv::Mat` of type `bool (CV_Bool)` can now be used as a mask for all functions where we used `cv::Mat` of `uchar/uint8_t` or `schar/int8_t` before.
  - Operations on `hfloat` and `bfloat` are always available, even on hardware that does not support those types natively. If necessary, internally we use efficient inline functions for scalar and vector float⇔float16/bfloat16 conversion.
	Support for the new types has been added to `cv::Mat`, `cv::UMat`,
            InputArray/OutputArray, key modules (core, dnn, 5.x imgproc etc.), FileStorage,
            various programming language bindings etc.
- OpenCV now supports arrays of lower than 2 dimensionality, i.e. 1D (vectors) and 0D (scalars). `std::vector<T>` wrapped into `Mat` or `InputArray/OutputArray` is interpreted as a 1D array, not 2D Nx1 or 1xN array (as in OpenCV 4.x). For 1D arrays `Mat::dims == Mat::rows == 1`, `Mat::cols == Mat::total() == <number_of_elements>`. For 0D arrays `Mat::dims == 0`, `Mat::rows == Mat::cols == Mat::total() == 1`.
- Lapack is now always available within OpenCV. In particular, it’s used for more efficient SVD and eigenvalue/eigenvector decomposition and the USAC framework. When there is no external Lapack library installed in the system, OpenCV builds and uses the internal small subset of Lapack.
- 5.0, 5.x: Further refactoring and improvements are expected before and after 5.0 release: https://github.com/opencv/opencv/issues/25011.

### Updated Imgproc module

- Accelerated image warping functions: warpAffine, warpPerspective, remap. Acceleration factor varies from 10% to more than 300%, depending on the platform, image size, type and operation flags. More optimizations for those functions, as well as other important image processing functions are expected in 5.0 gold.
- Text rendering now uses STB-based truetype engine and the embedded variable truetype font. Users can also load and use custom fonts. Consequently, many Unicode symbols are now supported, with a few caveats though: 1) some scripts, such as Arabic or Devanagari, are not rendered properly, we need to add Harfbuzz for that; 2) some compound Unicode symbols are not rendered properly for the same reason; 3) color emoji are not supported, as STB is a simple black-n-white engine.
- 5.0, 5.x: Further refactoring and improvements are expected before and after 5.0 release: https://github.com/opencv/opencv/issues/25012

### Updated HAL

- 5+4.x: Many new entries have been added to OpenCV HAL, allowing various vendors to provide custom acceleration to OpenCV functions. In 5.0 gold some more new HAL entries will be added.
- 5+4.x: In some cases vendors provide super-fast implementations that are unfortunately not 100% compatible with OpenCV. To let user to choose between speed and accuracy in some critical places, OpenCV introduced the optional `AlgorithmHint hint` parameter to several functions, which is set to `ALGO_HINT_DEFAULT` by default (which is equivalent to `ALGO_HINT_ACCURATE` unless user compiled OpenCV with `ALGO_HINT_DEFAULT=ALGO_HINT_FAST` (`-DOPENCV_ALGO_HINT_DEFAULT=ALGO_HINT_APPROX` option in cmake).
- 5+4.x: Universal intrinsics for math functions (`v_exp`, `v_log`, `v_erf`, `v_sincos`) have been added to accelerate deep learning inference, image processing and other algorithms.
- FP16 (i.e. half-precision) universal intrinsics have been added into ARMv8 NEON and RISC-V RVV backends. In both cases FP16 SIMD arithmetics is not available by default, the code needs to be built with special options. The corresponding parts of OpenCV use our runtime dispatching mechanism, which chooses the best-suited flavors of kernels depending on the actual client hardware.
- 5.0, 5.x: UMat will be extended to be able to store any CPU or non-CPU array/tensor. That is, from an OpenCL-only solution (T-API) we now migrate to a universal heterogeneous API (we might call it U-API or “non-CPU HAL”) and UMat will be the main data container for it. Many other improvements are expected as well: https://github.com/opencv/opencv/issues/25019, https://github.com/opencv/opencv/issues/25025

### Updated DNN (Deep Learning Inference Module)

The new engine has been introduced that now co-exists with the old engine. The new engine provides better support for dynamic shapes and other modern ONNX features. By 5.0 gold it will also provide much better coverage of ONNX specification than the old engine; now the coverage is comparable.
Added parameter `int engine = ENGINE_AUTO` to `cv::dnn::readNet()` to control which engine is used to load and further run the model. By default, we try the new engine first and if it fails to load the model, fall back to the old engine. We can also force the new engine or the old engine. Because of the different internal representation the engine cannot be switched after the model is loaded. Parsers for ONNX, Caffe, TF and TFLite formats have been updated to support the engine selection.
5.0, 5.x: currently, the new engine only supports the default backend and CPU target. It’s planned to enable more backends and targets by 5.0 gold and even more backends in subsequent 5.x releases [#26198](https://github.com/opencv/opencv/issues/26198).

### Updated 3D/Calib

- 5+4.x: Significantly more efficient USAC framework (especially in the case of noisy datasets) is now used for RANSAC-based algorithms to estimate homography, essential matrix, solve PnP problem etc. See https://docs.opencv.org/5.x/de/d3e/tutorial_usac.html
- New Levenberg–Marquardt algorithm implementation. It's faster and more accurate.
- Efficient multi-camera calibration framework has been added to calib module. It includes smart initialization and USAC-based optimization pipeline to compute relative positions of all simultaneously calibrated cameras w.r.t. the 1st camera. Various camera configurations are supported (all pinhole cameras, all fisheye cameras and the mixed case). See [tutorial](https://docs.opencv.org/5.x/d6/d36/tutorial_multiview_camera_calibration.html) for more details.
- Initiated work on basic mesh processing and point cloud processing algorithms: TSDF, ICP, etc. 
- Added importers and exporters for some popular point cloud formats, such as .ply and .obj.

### Revised Samples

- Revised the following deep learning samples: classification, segmentation, object detection, edge detection, tracking, person reid. All the samples use a collection of models that can be conveniently downloaded with ‘cd <opencv>/samples/dnn && python3 download_models.py’.
- Added experimental samples for LLM (GPT2) and diffusion (LDM) models.

## Migration for 4.x

TBD