## Building with OpenCV AI Kit support
* Get `Luxonis DepthAI library` from https://github.com/luxonis/depthai-core
* Build it according to instructions on their repository's readme
* Build G-API with OpenCV AI Kit support:
```bash
cmake /path-to-opencv -DWITH_OAK=ON -Ddepthai_DIR=/path/to/depthai/library
```

*Note*: currently OpenCV AI backend is rather limited in terms of covering DepthAI's functionality. See G-API's samples for usage examples (e.g. [OpenCV AI Kit inference](https://github.com/opencv/opencv/blob/4.x/modules/gapi/samples/oak_basic_infer.cpp) and [Getting video from OpenCV AI Kit device](https://github.com/opencv/opencv/blob/4.x/modules/gapi/samples/oak_copy.cpp)).