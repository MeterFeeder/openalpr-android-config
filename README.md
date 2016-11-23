# openalpr-android-config

A bash script, loosely based on [this Wiki entry](https://github.com/openalpr/openalpr/wiki/Android-compilation) from the openalpr repo, and my experience trying to walk through it, to build openalpr and its dependencies for using in an Android Studio project.

## Environment

The script is updated to work under Ubuntu 16.10, but may work under Linux with some modifications.

Tested under:

- Ubuntu 16.10
- Android Studio 2.2.2
- Android SDK: 25.2.3
- Android NDK: version 13.1.3345770 and android-ndk-r12 (installed by this script)
- OpenCV: OpenCV-2.4.9-android-sdk
    - TODO: this script should allow testing build with different version of the OpenCV < 3.0
- Tess-two: 6.1.0
- OpenALPR: [commit 469c4fd6d782ac]
- android-cmake: Script installs taka-no-me's fork (which fixes [this issue](https://code.google.com/p/android-cmake/issues/detail?id=14#c6))

**Note**: As of the openalpr commit shown above, the script also runs a [patch](https://github.com/twelve17/openalpr-android-config/blob/master/etc/openalpr_android.patch) against the openalpr source to fix a compilation issue in `filesystem.cpp`.  The patch change is based [on this post](http://lxr.free-electrons.com/source/arch/arm/kernel/sys_oabi-compat.c).

## Prerequisites

The script assumes some convention with regards to the location of some of the packages, and expects some environment variables to be set to help it along.  

- `ANDROID_DEV_ROOT`: the top level directory which contains the [Android SDK](https://developer.android.com/sdk/installing/index.html), [Android NDK](https://developer.android.com/tools/sdk/ndk/index.html), and [OpenCV Android SDK](http://opencv.org/platforms/android.html) downloads.  The default value is `$HOME/Android`.  Here is what mine looks like:

```
  $HOME/Android/
      |- adt-bundle-mac-x86_64-20140702/
      |- android-ndk-r10d/
      |- OpenCV-2.4.10-android-sdk/
```  

You must install these prerequisites before running the script.

(Also, FYI: the script will install `android-cmake` to `$ANDROID_DEV_ROOT/android-cmake`.)

## Installation

- Clone this repo:

  ```
  git clone https://github.com/twelve17/openalpr-android-config
  ```
- Edit the `bin/build_dependencies.sh` file, and change the values of these variables to match your setup:
  - `ANDROID_DEV_HOME`
  - `ANDROID_PLATFORM`
  - `ANDROID_SDK_NAME`
  - `ANDROID_NDK_NAME`
  - `ANDROID_OPENCV_SDK_NAME`
  - `JAVA_SDK_DIR`
 
## Execution
 
Run the script from the root of the project:

  ```
   ./bin/build_dependencies.sh
  ```
 
The script will download `tess-two`, `android-cmake`, and `openalpr`.  It will then compile the tess-two and openalpr libraries for all platforms specified in the `BUILD_ARCHS` variable.
 
Once completed, you should have a directory `openalpr-android-config/work/output`, which contains a `libs` and `includes` directory, with the `tess-two` and `openalpr` libraries.  You can then copy these into your Android project by hand (along with the `openalpr_java.so` libraries from `$ANDROID_OPENCV_SDK/jni/libs`).  

Alternatively, if you have an Android Studio project, you can add the path to your project to the script, and it will copy the `tess-two`, `openalpr`, and `opencv` libraries and header files to your project.  Assuming the file hierarchy below, you will end up wih something like this:

  ```
/path/to/your/android/studio/project/app/src/main
      |- jni
      |    |- include
      |         |- openalpr
      |         |    |- ...
      |         |- tesseract
      |              |- ...
      |- jniLibs
           |- armeabi
           |    |- liblept.so
           |    |- libopenalpr-static.a
           |    |- libsimpleini.a
           |    |- libsupport.a
           |    |- libtess.so
           |- armeabi-v7a
           |    |- ...
           |- ...

  ```

Given the above path to the project, run the script like this:

  ```
   ./bin/build_dependencies.sh /path/to/your/android/studio/project
  ```
  
You will still need to build any of your own project's JNI classes per the [Wiki instructions](https://github.com/openalpr/openalpr/wiki/Android-compilation) (e.g. creating an `Android.mk` file, etc).

## Notes

- openalpr `cmake` command args partially based [on this thread](https://groups.google.com/d/msg/openalpr/IbcSnMqhTxs/Y-bh9qffYzwJ).
