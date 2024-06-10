[Hardware Address Sanitizer](https://developer.android.com/ndk/guides/hwasan) or HWASan is an ARM64 processor feature that simplifies detection of potential memory problems such as stack/buffer overflow, double free and so on.
To use it with OpenCV, the library should be build with HWASan support enabled. In this instruction we'll describe how to build it, run tests on a device and get actual stack traces when memory error is detected.

1. Build OpenCV for Android:
   * Clone the code from this branch: [#25718](https://github.com/opencv/opencv/pull/25718) (or from 4.x branch if this PR is already merged)
   * Open [this instruction](https://github.com/opencv/opencv/wiki/Custom-OpenCV-Android-SDK-and-AAR-package-build)
   * Add `--hwasan` and `--debug` flags to `build_sdk.py` script call
   * Use NDK with version not less than 25
   * Optionally you can speed up build process by commenting out all configurations except `arm64-v8a` in a file `$YOUR_OPENCV_REPO/platforms/android/ndk-18-api-level-21.config.py`
   * Do first 6 steps of the instruction (before building AAR archive)
   * From here on we use variable names given in the script from the instruction
2. Check if your Android device is ready to run tests:
   * Connect the device
   * Go to a directory containing ADB tool in terminal: `cd $ANDROID_SDK/platform-tools`
   * Run ADB: `./adb devices` You should see your connected device
   * On Linux you may get an error telling that you do not have enough permissions. It can be fixed by:
     - adding your user to `plugdev` group: `sudo usermod -aG plugdev $USER`
     - logging out then back in
     - plugin out and then back in your Android device
3. Upload test application, test data and C++ STL shared library onto your device:
   * `./adb push $YOUR_OPENCV_BUILD_FOLDER/o4a/bin/opencv_test_core /data/local/tmp`
   * `./adb push $YOUR_OPENCV_EXTRA_REPO/testdata /sdcard`
   * `./adb push $YOUR_OPENCV_BUILD_FOLDER/o4a/opencv_android/opencv/build/intermediates/stripped_native_libs/debug/out/lib/arm64-v8a/libc++_shared.so /data/local/tmp`
4. Open ADB shell `./adb shell` and run tests on a device:
   ```
   export OPENCV_TEST_DATA_PATH=/sdcard
   cd /data/local/tmp
   LD_HWASAN=1 LD_PRELOAD=/data/local/tmp/libc++_shared.so ./opencv_test_core 2> out.txt
   ```
5. Extract logs from device: `./adb pull /data/local/tmp/out.txt $YOUR_LOGS_FILE` (do not forget to set `YOUR_LOGS_FILE` variable)
6. HWAsan logs contain stack traces with return addresses expressed in hexadecimal. We’d like to get source file and line numbers instead as we expect that from typical stack trace for a debug build. To get them, we post process these logs by `hwasan_symbolize` script:
```
$ANDROID_NDK_HOME/toolchains/llvm/prebuilt/linux-x86_64/lib64/clang/14.0.6/bin/hwasan_symbolize  --symbols $YOUR_OPENCV_BUILD_FOLDER/o4a/bin/  < $YOUR_LOGS_FILE
```
