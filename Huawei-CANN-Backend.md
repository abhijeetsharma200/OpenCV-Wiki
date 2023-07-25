## 📢 News

- [Jun. 29, 2023] 🎉 OpenCV 4.8.0 is released along with patches for CANN backend.
- [Mar. 24, 2023] 💡 Support Sub, PRelu, ConvTranspose. Add one-time warning when backend is switched back to CPU if CANN is not available. This patch will be available in 4.8.0 release. [PR#23401](https://github.com/opencv/opencv/pull/23401).
- [Mar. 14, 2023] 🐛 Fixed some bugs and added some new features to support most of the models in opencv_zoo. **This patch will be released along with 4.8.0**. Build from source with latest code to experience latest features. [PR#23319](https://github.com/opencv/opencv/pull/23319).
- [Dec. 28, 2022] 🎉 We released OpenCV 4.7.0 with CANN backend! [PR#22634](https://github.com/opencv/opencv/pull/22634).

---

CANN (Compute Architecture of Neural Networks), developped by Huawei, is a heterogeneous computing architecture for AI. With CANN backend in OpenCV DNN, you can run your AI models on the Ascend NPU. Learn more about Ascend NPU and the CANN library from [en_doc](https://www.hiascend.com/document), [cn_doc](https://www.hiascend.com/zh/document). ***Please note that OpenCV DNN supports CANN backend on Ascend 310 for now***.

To use OpenCV DNN with CANN backend, read the following sections:

1. Install dependencies,
1. Install CANN,
1. Compile OpenCV with CANN,
1. Python and C++ samples
1. OpenCV Zoo benchmark

## Install dependencies

Before installing CANN, make sure the following packges are installed:

- Python (3.7.x, or 3.8.x, or 3.9.x)
- CMake >= 3.5.1
- make
- gcc & g++ >= 7.3.0

You could also visit [this page](https://www.hiascend.com/document/detail/zh/CANNCommunityEdition/60RC1alpha003/softwareinstall/instg/atlasdeploy_03_0022.html) for a detailed list of dependencies.

### Multiple versions of Python

You will need to specify the Python you just installed in case there are multiple versions of Python in your computer:
```shell
# suppose Python 3.7.5 is installed in default path
export LD_LIBRARY_PATH=/usr/local/python3.7.5/lib:$LD_LIBRARY_PATH
export PATH=/usr/local/python3.7.5/bin:$PATH
```

***NOTE***: You could also append these lines in `~/.bashrc` so that you have the same environtment next time you open the terminal.

## Install CANN

Download `Ascend-cann-toolkit_{version}_{platform}.run` from https://www.hiascend.com/software/cann/community-history.
- `version >= 5.1.RC2.alpha008` is recommanded and tested by us.
- Choose your platform. `linux-x86` and `linux-aarch64` are supported.

Follow instructions from this page ([CN](https://www.hiascend.com/document/detail/zh/CANNCommunityEdition/51RC1alpha005/softwareinstall/instg/atlasdeploy_03_0018.html), [EN](https://www.hiascend.com/document/detail/en/CANNCommunityEdition/51RC1alphaX/softwareinstall/instg/atlasdeploy_03_0018.html)) to install the CANN library. The links to the instruction page is for **version 5.1.RC1.alpha005**. You could switch to your specific version by clicking the top-left drop-down menu.

## Build OpenCV with CANN

After installing CANN, you could find `set_env.sh` under `${cann_install_prefix}/ascend-toolkit`. In default CANN installation, `${cann_install_prefix}` is set to `/usr/local/Ascend`. Run the following command to set up CANN environment for compilation:

```shell
# replace ${cann_install_prefix} with your path
source ${cann_install_prefix}/ascend-toolkit/set_env.sh
```

***NOTE***: You could also append this line to `~/.bashrc` to have the same environment next time you open the terminal.

<!--
TODO: replace link with the one of opencv official after PR is merged
-->
Compile OpenCV with CANN using the following commands:
```shell
git clone https://github.com/fengyuentau/opencv.git
cd opencv
git checkout cann_backend_221010
mkdir build
cd build
cmake -D WITH_CANN=ON \
      -D BUILD_opencv_gapi=OFF \
      -D CMAKE_INSTALL_PREFIX=install ..
# ensure you see 'CANN: YES' in the end of the log
# Note: you could append "-j 8" in the following command for multi-job speedup.
#       More jobs are used, more memory is needed.
cmake --build . --target install
```

If OpenCV with Python interface is needed, use this CMake command instead:
```shell
# replace the value of PYTHON3_EXECUTABLE to your path to python binary
# replace the value of PYTHON3_LIBRARY to your path to python library (where you can find libpython3.x.so)
# replace the value of PYTHON3_INCLUDE_DIR to your path to the python include directory (where you can find Python.h)
cmake -D WITH_CANN=ON\
      -D CMAKE_INSTALL_PREFIX=install \
      -D BUILD_opencv_python2=OFF \
      -D BUILD_opencv_python3=ON \
      -D BUILD_opencv_gapi=OFF \
      -D PYTHON3_EXECUTABLE=/usr/local/python3.7.5/bin/python3.7m \
      -D PYTHON3_LIBRARY=/usr/local/python3.7.5/lib/libpython3.7m.so \
      -D PYTHON3_INCLUDE_DIR=/usr/local/python3.7.5/include/python3.7m \
      ..
```

***NOTE***: If your build is failed at downloading third-party resources, such as ADE, IPP and so on, you may get some help from the third Q&A in https://github.com/opencv/opencv/wiki/FAQ#build--install.

## Samples

In this section, we provide C++ and Python samples for PP-ResNet50, MobileNetV1 & YOLOX from [opencv_zoo](https://github.com/opencv/opencv_zoo).

### Get models

You could download the ONNX format of PP-ResNet50, MobileNetV1 and YOLOX from:

- PP-ResNet50: https://github.com/opencv/opencv_zoo/tree/master/models/image_classification_ppresnet
- MobileNetV1: https://github.com/opencv/opencv_zoo/tree/master/models/image_classification_mobilenet
- YOLOX: https://github.com/opencv/opencv_zoo/tree/master/models/object_detection_yolox

Tips: Visit [this page](https://github.com/opencv/opencv_zoo/wiki/Frequently-Asked-Problems#how-can-i-download-one-of-the-model-from-the-zoo) to learn how to download models in the zoo.

### Samples in Python

Copy and save the attached Python scripts. Instructions to run samples:

1. modify the paths to image and model,
2. enable the OpenCV Python interface
   ```shell
   # Replace '/path/to' with your prefix
   export PYTHONPATH=/path/to/opencv/build/python_loader:$PYTHONPATH
   ```
3. run samples:
   ```shell
   python3 ppresnet50.py
   python3 mobilenetv1.py
   python3 yolox.py
   ```

### Samples in C++

Copy and save the attached `.cpp` files and `CMakeLists.txt`. You will need to

1. modify the paths to image and model in `.cpp` files,
2. use the following commands to build and run the sample:
   ```shell
   # Replace `/path/to` with your prefix
   mkdir build && cd build
   CMAKE_PREFIX_PATH=/path/to/opencv/build/install cmake ..
   cmake --build . -j 8
   ```
3. run samples:
   ```shell
   # assume current working directory is in build
   ./ppresnet50
   ./mobilenetv1
   ./yolox
   ```

## OpenCV Zoo benchmark

We tested PP-ResNet50, MobileNetV1 and YOLOX from [OpenCV Zoo](https://github.com/opencv/opencv_zoo) and the CANN backend is able to make a speedup up to 22X! Free free to try more models on the CANN backend.

| Model       | CANN backend | CPU Backend |
| ----------- | ------------ | ----------- |
| PP-ResNet50 | 3.29         | 69.74       |
| MobileNetV1 | 1.21         | 6.60        |
| YOLOX       | 12.80        | 265.90      |

## Attachments

### Python demos

mobilenetv1.py:

```python
import numpy as np
import cv2 as cv

def preprocess(image):
    out = image.copy()

    out = cv.resize(out, (256, 256))
    out = out[16:240, 16:240, :]
    out = cv.dnn.blobFromImage(out, 1.0/255.0, mean=(0.485, 0.456, 0.406), swapRB=True)
    out = out / np.array([0.229, 0.224, 0.225]).reshape(1, -1, 1, 1)

    return out

def softmax(blob, axis=1):
    out = blob.copy().astype(np.float64)

    e_blob = np.exp(out)
    return e_blob / np.sum(e_blob, axis=axis)


image = cv.imread("/path/to/image") # replace with the path to your image
input_blob = preprocess(image)

net = cv.dnn.readNet("/path/to/image_classification_mobilenetv1_2022apr.onnx") # replace with the path to the model
net.setPreferableBackend(cv.dnn.DNN_BACKEND_CANN)
net.setPreferableTarget(cv.dnn.DNN_TARGET_NPU)

net.setInput(input_blob)
out = net.forward()

prob = softmax(out, axis=1)
_, max_prob, _, max_loc = cv.minMaxLoc(prob)
print("cls = {}, score = {:.4f}".format(max_loc[0], max_prob))
```

ppresnet50.py:

```python
import numpy as np
import cv2 as cv

def preprocess(image):
    out = image.copy()

    out = cv.resize(out, (256, 256))
    out = out[16:240, 16:240, :]
    out = cv.dnn.blobFromImage(out, 1.0/255.0, mean=(0.485, 0.456, 0.406), swapRB=True)
    out = out / np.array([0.229, 0.224, 0.225]).reshape(1, -1, 1, 1)

    return out

def softmax(blob, axis=1):
    out = blob.copy().astype(np.float64)

    e_blob = np.exp(out)
    return e_blob / np.sum(e_blob, axis=axis)


image = cv.imread("/path/to/image") # replace with the path to your image
input_blob = preprocess(image)

net = cv.dnn.readNet("/path/to/image_classification_ppresnet50_2022jan.onnx") # # replace with the path to the model
net.setPreferableBackend(cv.dnn.DNN_BACKEND_CANN)
net.setPreferableTarget(cv.dnn.DNN_TARGET_NPU)

net.setInput(input_blob)
output_blob = net.forward("save_infer_model/scale_0.tmp_0")

prob = softmax(output_blob, axis=1)
_, max_prob, _, max_loc = cv.minMaxLoc(prob)
print("cls = {}, score = {:.4f}".format(max_loc[0], max_prob))
```

yolox.py:

```python
import numpy as np
import cv2 as cv

def postprocess(blob, confidence_threshold=0.5, nms_threshold=0.5):
    out = blob.copy()

    strides = [8, 16, 32]
    hsizes = [80, 40, 20]
    wsizes = [80, 40, 20]

    grids = []
    expanded_strides = []
    for hsize, wsize, stride in zip(hsizes, wsizes, strides):
        xv, yv = np.meshgrid(np.arange(hsize), np.arange(wsize))
        grid = np.stack((xv, yv), 2).reshape(1, -1, 2)
        grids.append(grid)
        shape = grid.shape[:2]
        expanded_strides.append(np.full((*shape, 1), stride))

    grids = np.concatenate(grids, 1)
    expanded_strides = np.concatenate(expanded_strides, 1)

    out[..., :2] = (out[..., :2] + grids) * expanded_strides
    out[..., 2:4] = np.exp(out[..., 2:4]) * expanded_strides

    # retrieve bboxes
    bboxes = out[0, :, :4]
    bboxes_xyxy = np.ones_like(bboxes) # (n, 4)
    bboxes_xyxy[:, 0] = bboxes[:, 0] - bboxes[:, 2] / 2.
    bboxes_xyxy[:, 1] = bboxes[:, 1] - bboxes[:, 3] / 2.
    bboxes_xyxy[:, 2] = bboxes[:, 0] + bboxes[:, 2] / 2.
    bboxes_xyxy[:, 3] = bboxes[:, 1] + bboxes[:, 3] / 2.

    # retrieve scores
    scores = out[0, :, 4:5] * out[0, :, 5:]
    max_scores = np.amax(scores, axis=1)
    max_scores_idx = np.argmax(scores, axis=1)

    out = np.concatenate([bboxes_xyxy, max_scores[:, None], max_scores_idx[:, None]], axis=1)

    # batched-nms
    max_coord = bboxes_xyxy.max()
    offsets = max_scores_idx * (max_coord + 1)
    bboxes_for_nms = bboxes_xyxy + offsets[:, None]
    keep = cv.dnn.NMSBoxes(bboxes_for_nms.tolist(), max_scores.tolist(), confidence_threshold, nms_threshold)

    final_out = out[keep]
    return final_out

image = cv.imread("/path/to/image") # replace with the path to your image
input_blob = cv.dnn.blobFromImage(image, size=(640, 640), swapRB=True)

net = cv.dnn.readNet("/path/to/object_detection_yolox_2022nov.onnx") # replace with the path to the model
net.setPreferableBackend(cv.dnn.DNN_BACKEND_CANN)
net.setPreferableTarget(cv.dnn.DNN_TARGET_NPU)

net.setInput(input_blob)
out = net.forward()

dets = postprocess(out)
for det in dets:
    bbox = det[0:4].astype(np.int32)
    score = det[4]
    clsid = det[5].astype(np.int32)
    print("bbox: {}, score: {:.4f}, clsid: {}".format(bbox, score, clsid))
```

### C++ demos

CMakeLists.txt:

```cmake
cmake_minimum_required(VERSION 3.5.1)
project(cann_demo)

# OpenCV
find_package(OpenCV 4.6.0 REQUIRED)
include_directories(${OpenCV_INCLUDE_DIRS})

# PP-ResNet50
add_executable(ppresnet50 ppresnet50.cpp)
target_link_libraries(ppresnet50 ${OpenCV_LIBS})

# MobileNetV1
add_executable(mobilenetv1 mobilenetv1.cpp)
target_link_libraries(mobilenetv1 ${OpenCV_LIBS})

# YOLOX
add_executable(yolox yolox.cpp)
target_link_libraries(yolox ${OpenCV_LIBS})

```

mobilenetv1.cpp:

```cpp
#include <iostream>
#include <vector>

#include "opencv2/opencv.hpp"

void preprocess(const cv::Mat& src, cv::Mat& dst)
{
    src.convertTo(dst, CV_32FC3);

    cv::cvtColor(dst, dst, cv::COLOR_BGR2RGB);
    // center crop
    cv::resize(dst, dst, cv::Size(256, 256));
    cv::Rect roi(16, 16, 224, 224);
    dst = dst(roi);

    dst = cv::dnn::blobFromImage(dst, 1.0/255.0, cv::Size(), cv::Scalar(0.485, 0.456, 0.406));
    cv::divide(dst, cv::Scalar(0.229, 0.224, 0.225), dst);
}

void softmax(const cv::Mat& src, cv::Mat& dst, int axis=1)
{
    using namespace cv::dnn;

    LayerParams lp;
    Net netSoftmax;
    netSoftmax.addLayerToPrev("softmaxLayer", "Softmax", lp);
    netSoftmax.setPreferableBackend(DNN_BACKEND_OPENCV);

    netSoftmax.setInput(src);
    cv::Mat out = netSoftmax.forward();
    out.copyTo(dst);
}

int main(int argc, char** argv)
{
    using namespace cv;

    Mat image = imread("/path/to/image"); // replace with the path to your image
    Mat input_blob;
    preprocess(image, input_blob);

    dnn::Net net = dnn::readNet("/path/to/image_classification_mobilenetv1_2022apr.onnx"); // replace with the path to the model
    net.setPreferableBackend(dnn::DNN_BACKEND_CANN);
    net.setPreferableTarget(dnn::DNN_TARGET_NPU);

    net.setInput(input_blob);
    Mat out = net.forward();

    Mat prob;
    softmax(out, prob, 1);

    double min_val, max_val;
    Point min_loc, max_loc;
    minMaxLoc(prob, &min_val, &max_val, &min_loc, &max_loc);
    std::cout << cv::format("cls = %d, score = %.4f\n", max_loc.x, max_val);

    return 0;
}
```

PP-ResNet50:

```cpp
#include <iostream>
#include <vector>

#include "opencv2/opencv.hpp"

void preprocess(const cv::Mat& src, cv::Mat& dst)
{
    src.convertTo(dst, CV_32FC3);

    cv::cvtColor(dst, dst, cv::COLOR_BGR2RGB);
    // center crop
    cv::resize(dst, dst, cv::Size(256, 256));
    cv::Rect roi(16, 16, 224, 224);
    dst = dst(roi);

    dst = cv::dnn::blobFromImage(dst, 1.0/255.0, cv::Size(), cv::Scalar(0.485, 0.456, 0.406));
    cv::divide(dst, cv::Scalar(0.229, 0.224, 0.225), dst);
}

void softmax(const cv::Mat& src, cv::Mat& dst, int axis=1)
{
    using namespace cv::dnn;

    LayerParams lp;
    Net netSoftmax;
    netSoftmax.addLayerToPrev("softmaxLayer", "Softmax", lp);
    netSoftmax.setPreferableBackend(DNN_BACKEND_OPENCV);

    netSoftmax.setInput(src);
    cv::Mat out = netSoftmax.forward();
    out.copyTo(dst);
}

int main(int argc, char** argv)
{
    using namespace cv;

    Mat image = imread("/path/to/image"); // replace with the path to your image
    Mat input_blob;
    preprocess(image, input_blob);

    dnn::Net net = dnn::readNet("/path/to/image_classification_ppresnet50_2022jan.onnx"); // replace with the path to the model
    net.setPreferableBackend(dnn::DNN_BACKEND_CANN);
    net.setPreferableTarget(dnn::DNN_TARGET_NPU);

    net.setInput(input_blob);
    Mat out = net.forward("save_infer_model/scale_0.tmp_0");

    Mat prob;
    softmax(out, prob, 1);

    double min_val, max_val;
    Point min_loc, max_loc;
    minMaxLoc(prob, &min_val, &max_val, &min_loc, &max_loc);

    std::cout << cv::format("cls = %d, score = %.4f\n", max_loc.x, max_val);

    return 0;
}
```

yolox.cpp:

```cpp
#include <iostream>
#include <vector>

#include "opencv2/opencv.hpp"
using namespace cv;

cv::Mat postprocess(const cv::Mat& blob, const float confidence_threshold = 0.5, const float nms_threshold = 0.5)
{
    std::vector<int> strides{8, 16, 32};
    std::vector<int> hsizes{80, 40, 20};
    std::vector<int> wsizes{80, 40, 20};

    std::vector<Point2f> grids(8400);
    std::vector<float> expanded_strides(8400);
    int i, j, k, l = 0, h, w;
    for (i = 0; i < hsizes.size(); i++)
    {
        h = hsizes[i];
        w = wsizes[i];
        for (j = 0; j < h; j++)
        {
            for (k = 0; k < w; k++)
            {
                Point2f grid{float(k), float(j)};
                grids[l] = grid;
                expanded_strides[l] = float(strides[i]);
                l++;
            }
        }
    }

    const float* p_delta = (const float*)blob.data;

    Mat outs;
    Mat out(1, 6, CV_32FC1);
    for (i = 0; i < 8400; i++)
    {
        j = i * 85;
        Point2f grid = grids[i];
        float expanded_stride = expanded_strides[i];

        // retrieve objectness score
        float objectness = p_delta[j + 4];

        // retrieve class scores
        float max_score = -1.f;
        float max_idx = -1.f;
        float this_score;
        for (k = 5; k < 85; k++)
        {
            this_score = p_delta[j + k] * objectness;
            if (this_score > max_score)
            {
                max_score = this_score;
                max_idx = k - 5;
            }
        }
        if (max_score < 0.5)
            continue;
        out.at<float>(0, 4) = max_score;
        out.at<float>(0, 5) = max_idx;

        // retrieve bbox
        float cx = (p_delta[j] + grid.x) * expanded_stride;
        float cy = (p_delta[j + 1] + grid.y) * expanded_stride;
        float width = std::exp(p_delta[j + 2]) * expanded_stride;
        float height = std::exp(p_delta[j + 3]) * expanded_stride;
        out.at<float>(0, 0) = cx - width / 2;
        out.at<float>(0, 1) = cy - height / 2;
        out.at<float>(0, 2) = cx + width / 2;
        out.at<float>(0, 3) = cy + height / 2;

        outs.push_back(out);
    }

    Mat dets = outs;
    if (dets.rows > 1)
    {
        // batched nms
        float max_coord = -1;
        for (i = 0; i < dets.rows; i++)
            for (j = 0; j < 4; j++)
                if (max_coord < dets.at<float>(i, j))
                    max_coord = dets.at<float>(i, j);

        std::vector<Rect2i> boxes;
        std::vector<float> scores;
        float offsets;
        for (i = 0; i < dets.rows; i++)
        {
            offsets = dets.at<float>(i, 5) * (max_coord + 1);
            boxes.push_back(
                Rect2i(int(dets.at<float>(i, 0) + offsets),
                       int(dets.at<float>(i, 1) + offsets),
                       int(dets.at<float>(i, 2) + offsets),
                       int(dets.at<float>(i, 3) + offsets))
            );
            scores.push_back(dets.at<float>(i, 4));
        }
        std::vector<int> keep;
        dnn::NMSBoxes(boxes, scores, 0.5, 0.5, keep);

        Mat dets_after_nms;
        for (auto idx : keep)
            dets_after_nms.push_back(dets.row(idx));

        dets = dets_after_nms;
    }
    return dets;
}

int main(int argc, char** argv)
{
    Mat image = imread("/path/to/image"); // replace with the path to your image
    Mat input_blob = dnn::blobFromImage(image, 1.0f, cv::Size(640, 640), cv::Scalar(), true);

    dnn::Net net = dnn::readNet("/path/to/object_detection_yolox_2022nov.onnx"); // replace with the path to the model
    net.setPreferableBackend(dnn::DNN_BACKEND_CANN);
    net.setPreferableTarget(dnn::DNN_TARGET_NPU);

    net.setInput(input_blob);
    Mat out = net.forward();

    Mat dets = postprocess(out);
    for (int i = 0; i < dets.rows; i++)
    {
        int x1 = int(dets.at<float>(i, 0));
        int y1 = int(dets.at<float>(i, 1));
        int x2 = int(dets.at<float>(i, 2));
        int y2 = int(dets.at<float>(i, 3));
        float score = dets.at<float>(i, 4);
        int cls = int(dets.at<float>(i, 5));
        std::cout << cv::format("box [%d, %d, %d, %d], score %f, class %d\n", x1, y1, x2, y2, score, cls);
    }

    return 0;
}
```