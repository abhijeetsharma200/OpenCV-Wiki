## Building G-API with oneVPL Toolkit support

* Build (https://github.com/oneapi-src/oneVPL) or install `oneVPL` from installation manager (https://www.intel.com/content/www/us/en/developer/tools/oneapi/onevpl.html)

* Source an oneVPL environment variables or execute shell script

```bash
<installation path>/share/oneVPL/env/vars.bat"
```

* Then specify extra options to OpenCV CMake:

```bash
 $ cmake /path-to-opencv -DWITH_GAPI_ONEVPL=ON
```

* Run tests:
```bash
/path-to-opencv-build/bin/opencv_test_gapi --gtest_filter=*OneVPL_Source*
```

* How to run example and configure `oneVPL` file-based source and launch `OpenVINO` inference please find out examples in `/path-to-opencv-build/bin/example_gapi_onevpl_infer_single_roi`.
Also see `Building with OpenVINO Toolkit support` section to how to configure G-API with `OpenVINO`

### VPL Source capabilities & limitations
* G-API oneVPL Source implements string-based parameters configuration mechanism through `CfgParam` objects packed into array or initialization list. These parameters has reflection of `oneVPL` configuration parameters which can be found by the link https://spec.oneapi.io/versions/latest/elements/oneVPL/source/programming_guide/VPL_prg_session.html#dsp-conf-prop-table.
Some of these parameters are MAJOR and some others are OPTIONAL. Using MAJOR params is necessary to make VPL source work, while OPTIONAL params provide extra optimization tricks or advice VPL dispatcher to select the preferable oneVPL library implementation (like a version index) and so on.
All params have `name` and `value` fields which should be mapped to VPL-related configuration parameter by G-API oneVPL Source by itself.

Lets consider example of choosing Hardware Acceleration type for VPL Source:

As described in https://spec.oneapi.io/versions/latest/elements/oneVPL/source/programming_guide/VPL_prg_session.html#dsp-conf-prop-table it has name `mfxImplDescription.AccelerationMode` and with type `MFX_VARIANT_TYPE_U32` then we just use 

```
std::vector<CfgParam> cfg_params;
cfg_params.push_back(CfgParam::create_acceleration_mode(MFX_ACCEL_MODE_VIA_D3D11));
```
or
```
std::vector<CfgParam> cfg_params;
cfg_params.push_back(CfgParam::create_acceleration_mode("MFX_ACCEL_MODE_VIA_D3D11"));
```

G-API oneVPL Source interface must parse either `int` or `string` like parameter value representation. To find out which VPL parameters are supported please proceed by https://github.com/opencv/opencv/blob/4.x/modules/gapi/include/opencv2/gapi/streaming/onevpl/cfg_params.hpp#L63
(List of parameters is updated regularly)

* Only Windows platform is tested and supported with Hardware Acceleration DX11

According to `oneVPL` dispatcher the user is free to choose preferred acceleration type. But default deployment of VPL implementation driver supply hardware acceleration only which means that ALL decoding operations use hardware acceleration. It is possible only to clarify to G-API oneVPL Source what types of memory should produce oneVPL Source in its own `cv::MediaFrame` as result. If no parameters described `mfxImplDescription.AccelerationMode` have passed during G-API oneVPL source construction then `cv::MediaFrame` will carry CPU memory as frame data (it means copy from CPU to acceleration during decode operation and back again). In otherwise, let's assume we passed `CfgParam::create_acceleration_mode("MFX_ACCEL_MODE_VIA_D3D11"))`, a `cv::MediaFrame` would carry GPU memory as data in DX11Texture2D inside and would require using `cv::MediaFrame::access` to get it's value.

* G-API oneVPL Source support video decoding either using RAW video stream formats (see onVPL support codes) and inner demultiplexing using `Microsoft Foundation Primitives`. 

Implementation doesn't rely on file extension to choose format because usually it might be wrong. Instead the following interface is provided: If no parameters described `mfxImplDescription.mfxDecoderDescription.decoder.CodecID` have passed during G-API oneVPL source construction then implementation try out demultiplexing schema; if specific codecId is set ( for example `CfgParam::create_decoder_id(MFX_CODEC_HEVC)`) then implementation assume RAW stream unconditionally.

Please use environment variable `OPENCV_LOG_LEVEL=Info` at least to consider full description in case of any source file errors but usually default level `OPENCV_LOG_LEVEL=Warn` is enough

### How-to-launch OpenVINO Inference using VPL Source practical guide

First of all make sure that conditions are met: https://github.com/opencv/opencv/wiki/Graph-API#building-with-openvino-toolkit-support and https://github.com/opencv/opencv/wiki/Graph-API#building-with-onevpl-toolkit-support.
The sample with oneVPL & OpenVINO Inference Engine can be found in `gapi/samples` directory under openCV project directories tree. Only infer single ROI is supported at now.

(Current configuration parameters are obsolete and new will be introduced in https://github.com/opencv/opencv/pull/21716. Description written down in assumption that this PR was merged)

* Source video file is a RAW encoded video stream (h265 for example)
```bash
example_gapi_onevpl_infer_single_roi --facem=<model path> --cfg_params="mfxImplDescription.mfxDecoderDescription.decoder.CodecID:MFX_CODEC_HEVC;" --input=<full RAW file path>
```
Please explore the full list of supported codec constants here: https://github.com/opencv/opencv/blob/4.x/modules/gapi/include/opencv2/gapi/streaming/onevpl/cfg_params.hpp#L95

* Source file is a containerized media file: *.mkv, *.mp4, etc. (Applicable for WINDOWS only)
```bash
example_gapi_onevpl_infer_single_roi --facem=<model path> --cfg_params="" --input=<full RAW file path>
```
or
```bash
example_gapi_onevpl_infer_single_roi --facem=<model path> --input=<full file path>
```

Please pay attention that examples launch non-optimized pipeline for default acceleration types:
- VPL Source uses GPU device for decoding with copying media frame into CPU RAM
- VPL preprocessing used GPU device for decoding with copying media frame from/into CPU RAM
- Inference uses CPU device too

Also it is possible to configure such pipeline stages in fine-grained way and seize heterogenous computation advantages. Thus, three acceleration parameters exposed: `source_device`, `preproc_device` and `faced`. Variety combinations of either `CPU` & `GPU` values are supported. 
Full list of supported configuration enumerated in sample support device matrix and is constantly growing. 

The most interesting cases are:

* Default GPU-accelerated decoding & copy-based CPU use-case is similar with (synonym for empty parameters): 
```bash
example_gapi_onevpl_infer_single_roi  <...> --source_device=CPU --preproc_device=CPU --facem=CPU
```

*  GPU decode/preprocessing pipeline with CPU-based inference: 
```bash
example_gapi_onevpl_infer_single_roi  <...> --source_device=GPU --preproc_device=GPU --facem=CPU
```

*  Full copy-free GPU pipeline can be configured as:
```bash
example_gapi_onevpl_infer_single_roi  <...> --source_device=GPU --preproc_device=GPU --facem=GPU
```