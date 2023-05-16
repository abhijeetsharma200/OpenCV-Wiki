Submitting G-API PRs
====================

G-API supports so-called STANDALONE mode build which is not validated by default.
Please put the below lines to the PR description to validate that this mode is not broken by the PR:

```
force_builders=Custom,Custom Win,Custom Mac
build_gapi_standalone:Linux x64=ade-0.1.1f
build_gapi_standalone:Win64=ade-0.1.1f
build_gapi_standalone:Mac=ade-0.1.1f
build_gapi_standalone:Linux x64 Debug=ade-0.1.1f

Xbuild_image:Custom=centos:7
Xbuildworker:Custom=linux-1
build_gapi_standalone:Custom=ade-0.1.1f

build_image:Custom=ubuntu-openvino-2021.4.1:20.04
build_image:Custom Win=openvino-2021.4.1
build_image:Custom Mac=openvino-2021.4.1

test_modules:Custom=gapi,python2,python3,java
test_modules:Custom Win=gapi,python2,python3,java
test_modules:Custom Mac=gapi,python2,python3,java

buildworker:Custom=linux-1
# disabled due high memory usage: test_opencl:Custom=ON
test_opencl:Custom=OFF
test_bigdata:Custom=1
test_filter:Custom=*
```

In order to enable `oneVPL` related builders please use `Custom Win` with `build_image:Custom Win=gapi-onevpl-2021.6.0`

Notes:
- ADE version may change, refer to the latest correct one (see `DownloadADE.cmake`).

G-API build depends on a number of external components which may not be built by default, e.g. [PlaidML](https://github.com/plaidml/plaidml):

```
force_builders=Custom
buildworker:Custom=linux-1
build_image:Custom=plaidml2
test_modules:Custom=gapi
test_filter:Custom=*ML*
```

or [ONNX Runtime](https://github.com/microsoft/onnxruntime):

```
force_builders=Custom
buildworker:Custom=linux-1
build_image:Custom=ubuntu-onnx:20.04
test_modules:Custom=gapi
test_filter:Custom=*ONNX*
```

or [FreeType](https://www.freetype.org/):

```
force_builders=Custom
buildworker:Custom=linux-1
build_image:Custom=ubuntu-gapi-freetype:16.04
```

It is also worth testing if any internal API changes are made.