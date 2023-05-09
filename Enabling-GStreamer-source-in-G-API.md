# Building G-API with GStreamer support on Ubuntu 20.04:

* As a first step, you need to install the following dependencies:
  ```bash
  $ sudo apt install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev gstreamer1.0-plugins-{base,good,bad}
  ```

* Secondly, you need to launch OpenCV cmake with an extra option specified:    
  ```bash
  $ cmake /path-to-opencv -DOPENCV_GAPI_GSTREAMER=ON
  ```


After completing these steps, you should see the following in the cmake output:
  ```bash
  -- Checking for module 'gstreamer-base-1.0'
  --   Found gstreamer-base-1.0, version 1.16.3
  -- Checking for module 'gstreamer-app-1.0'
  --   Found gstreamer-app-1.0, version 1.16.3
  -- Checking for module 'gstreamer-riff-1.0'
  --   Found gstreamer-riff-1.0, version 1.16.3
  -- Checking for module 'gstreamer-pbutils-1.0'
  --   Found gstreamer-pbutils-1.0, version 1.16.3
  -- Checking for module 'gstreamer-video-1.0'
  --   Found gstreamer-video-1.0, version 1.16.3
  -- Checking for module 'gstreamer-audio-1.0'
  --   Found gstreamer-audio-1.0, version 1.16.3
  ```

And, also the following:
```bash
--   Video I/O:
--   ...
--     GStreamer:                   YES (1.16.3)
--   ...
```

# Running G-API GStreamerSource Tests:

When you build G-API with GStreamer support, tests for streaming via GStreamerSource are enabled. However, some of them require test data from **_opencv_extra_** repository. So, to launch all the tests successfully, you need:

* Clone _opencv_extra_ repository:
  ```bash
  $ git clone https://github.com/opencv/opencv_extra.git
  ```
* Set _OPENCV_TEST_DATA_PATH_ environment variable:
  ```bash
  $ export OPENCV_TEST_DATA_PATH=/path-to-opencv_extra/testdata
  ```
* Run tests with _\*GStreamer\*_ filter:
  ```bash
  $ ./path-to-opencv-bin/opencv_test_gapi --gtest_filter=*GStreamer*
  ```

After completing these steps, you should see that all of tests pass.
For example:

![image](https://github.com/opencv/opencv/assets/15359579/016da457-1370-4297-ba15-3b3ee20ad67f)

You may, however, notice some warnings during execution, like these ones:

![image](https://github.com/opencv/opencv/assets/15359579/29b80e31-b560-47bd-9d49-1c384a52ceee)

or

![image](https://github.com/opencv/opencv/assets/15359579/37e0c27a-1239-4921-94c1-407e2cdd5acf)

Please don't worry, they are expected!