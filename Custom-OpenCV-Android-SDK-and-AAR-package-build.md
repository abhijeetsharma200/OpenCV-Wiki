In this article we’ll cover the following topics:

* How to cross-compile OpenCV for Android under Ubuntu
* How to use contrib modules in the Android build
* How to run sample applications on a device using Android Studio
* How to run tests on the connected Android device using ADB

## Prerequisites:
- Ubuntu 20.04+
- [Android Studio](https://developer.android.com/studio)
- OpenJDK 17: `sudo apt install openjdk-17-jdk openjdk-17-jre`
- CMake
- Ninja build tool: `sudo apt install ninja-build`
- OpenCV source code (https://github.com/opencv/opencv)
- (Optional) OpenCV contrib modules source code (https://github.com/opencv/opencv_contrib)
- (Optional) OpenCV test data (https://github.com/opencv/opencv_extra)

## OpenCV Build Steps:
1. Install SDK 21.1.2 and NDK 18.1 using SDK manager in Android Studio (use checkbox “Show package details” to choose a version)
2. Check out OpenCV source code at version 4.x to some folder, let’s call this folder `YOUR_OPENCV_SRC_FOLDER`
3. (Optional) Check out OpenCV contrib repo at version 4.x to some folder, let’s call it `YOUR_CONTRIB_SRC_FOLDER`
4. (Optional) Check out OpenCV test data repo at version 4.x to some folder, let’s call it `YOUR_TEST_DATA_FOLDER`
5. Create a folder to build OpenCV to, let’s call it `YOUR_OPENCV_BUILD_FOLDER`
6. Substitute your paths to the script below and run it in the console (pay attention to <marked> paths):
 - If you do not need contrib modules remove `--extra_modules_path` and its corresponding path to contrib repo from the arguments
 - If you need video I/O support, add `--use_media_ndk` to the list of arguments

```
export YOUR_OPENCV_SRC_FOLDER=<path to your OpenCV main repo folder>
export YOUR_CONTRIB_SRC_FOLDER=<path to your OpenCV contrib repo folder>
export YOUR_OPENCV_BUILD_FOLDER=<path to your OpenCV build folder>
export ANDROID_SDK=<path to your Android SDK, for example /home/user123/Android/Sdk>
export ANDROID_NDK_HOME=<path to NDK, for example /home/user123/Android/Sdk/ndk/18.1.5063045>

python3 $YOUR_OPENCV_SRC_FOLDER/platforms/android/build_sdk.py $YOUR_OPENCV_BUILD_FOLDER $YOUR_OPENCV_SRC_FOLDER --ndk_path $ANDROID_NDK_HOME --sdk_path $ANDROID_SDK --extra_modules_path $YOUR_CONTRIB_SRC_FOLDER/modules --config $YOUR_OPENCV_SRC_FOLDER/platforms/android/ndk-18-api-level-21.config.py
```

7. Change folder to some new one, let’s call it `YOUR_AAR_FOLDER`
8. Build AAR:
```
python3 $YOUR_OPENCV_SRC_FOLDER/platforms/android/build_java_shared_aar.py $YOUR_OPENCV_BUILD_FOLDER/OpenCV-android-sdk
```
9. Open file `<YOUR_OPENCV_BUILD_FOLDER>/OpenCV-android-sdk/samples/settings.gradle` in a text editor and change its beginning like this (pay attention to the <marked> place):

```
rootProject.name = 'opencv_samples'

gradle.ext {
	//opencv_source = 'maven_central'
	opencv_source = 'maven_local'
	//opencv_source = 'sdk_path'
}

if (gradle.opencv_source == 'maven_local') {
	gradle.ext {
    	opencv_maven_path = '<YOUR_AAR_FOLDER>/outputs/maven_repo'
	}
}
```

10. Open project in Android Studio in `<YOUR_OPENCV_BUILD_FOLDER>/OpenCV-android-sdk/samples`

## Advanced OpenCV Build options

OpenCV itself is configured and build with CMake and it means, that any valid CMake options may be applied to the build process. For example, you may need to build OpenCV without some modules, enable or disable some 3rdparty components, or tune performance optimizations. Android API level, plugin versions and CMake options are defined in `build_sdk.py` config provided with `--config` option. The exaple above uses `platforms/android/ndk-18-api-level-21.config.py` which corresponds to build with default NDK, Java API level 21 and default set of architectures. The config contains python list named "ABIs", each element of the list os ABI object defined in `build_sdk.py` as
```
class ABI:
    def __init__(self, platform_id, name, toolchain, ndk_api_level = None, cmake_vars = dict()):
        self.platform_id = platform_id # platform code to add to apk version (for cmake)
        self.name = name # general name (official Android ABI identifier)
        self.toolchain = toolchain # toolchain identifier (for cmake)
        self.cmake_vars = dict(
            ANDROID_STL="gnustl_static",
            ANDROID_ABI=self.name,
            ANDROID_PLATFORM_ID=platform_id,
        )
        if toolchain is not None:
            self.cmake_vars['ANDROID_TOOLCHAIN_NAME'] = toolchain
        else:
            self.cmake_vars['ANDROID_TOOLCHAIN'] = 'clang'
            self.cmake_vars['ANDROID_STL'] = 'c++_shared'
        if ndk_api_level:
            self.cmake_vars['ANDROID_NATIVE_API_LEVEL'] = ndk_api_level
        self.cmake_vars.update(cmake_vars)
    def __str__(self):
        return "%s (%s)" % (self.name, self.toolchain)
    def haveIPP(self):
        return self.name == "x86" or self.name == "x86_64"
```
`cmake_vars` parameter allows to substitute custom CMake arguments as a dictionary. Several examples:

- Disable G-API and DNN module for armv8-a architecture:
```
ABI("3", "arm64-v8a",   None, 21, cmake_vars=dict('BUILD_opencv_gapi': 'OFF', 'BUILD_opencv_dnn': 'OFF'))
```
- Force particular tools version for OpenCV build:
```
ABI("3", "arm64-v8a",   None, 21, cmake_vars=dict('ANDROID_GRADLE_PLUGIN_VERSION': '7.3.1', 'GRADLE_VERSION': '7.5.1', 'KOTLIN_PLUGIN_VERSION': '1.5.20'))
```
- Point to external libraries location for CMake find_package procedure:
```
ABI("3", "arm64-v8a",   None, 21, cmake_vars=dict('libavif_DIR': '<path to libavif library cross-compiled for Android arm-v8a>')
```
