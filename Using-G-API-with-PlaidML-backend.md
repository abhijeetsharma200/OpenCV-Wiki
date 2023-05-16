## Building with PlaidML support

* Follow instruction to build PlaidML2: https://plaidml.github.io/plaidml/docs/building
```bash
bazelisk build //plaidml2:wheel
pip install -U bazel-bin/plaidml2/wheel.pkg/tmp/dist/*
```
* Setup PLAIDML_DEVICE
```bash
python3 /path-to-plaidml/plaidml/plaidml2/plaidml_setup.py
export `cat ~/.plaidml2`
```
* Build G-API with PlaidML2 support:
```bash
cmake /path-to-opencv -DPlaidML2_DIR=path-to-miniconda3/share/plaidml2 -DWITH_PLAIDML=ON
```
* Run tests:
```bash
/path-to-opencv-build/bin/opencv_test_gapi --gtest_filter=*GAPI_PlaidML_Pipelines*
```