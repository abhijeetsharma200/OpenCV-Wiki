Note: This page is currently brainstormed. The idea of this page is to collect ideas and issues to address to refine G-API for the OpenCV 5.0 release.

## How we see G-API in 5.0

- Dmitry - Python becomes interface #️⃣1️⃣
  - **pip-installable**, with dynamically loadable options when available (e.g., onnx runtime, gstreamer, openvino)
  - Well-documented (GSoC on tutorials in ongoing)
  - Tolik - Extensible in Python, including for Sources (e.g. to extend on audio/speech scenarios)
  - Tolik - Expose operation functions (API)
  - Tolik - Expose Operations as types - to make them implementable in Python.
    - Examples: use PIL resize as part of pipeline
  - Tolik - Refine API, how easy it is to use?
    - Example: GInferInputs versus a Python dict
- Tolik - High-order constructs (conditions and loops)
  - Can retire infer-list and infer-2 if implemented well. Also can work faster with more precise pipelining
- Tolik - TBB executor. Useful for latency case
- Dmitry - Streaming executor
  - Utilizes lots of threads, can be redesigned to use a limited thread pool

## What we miss

### Inference backends

- Tolik - Plugins for sources and inference backends
- Dmitry - Inference backend atop of OCV DNN module
  - Tolik - Any DNN prerequisites? E.g. understanding model shape at configuration/load time (w/no execution)?
- Tolik - ONNX Runtime updates
  - EP selection with options
- Tolik - Generic approach to model preprocessing?
  - Manual preprocessing pipeline configuration. Currently for ONNX (and, in theory, for OCV DNN), the configuration path is not very clean.
  - OpenVINO backend with imported blobs[?]
- Tolik - OpenVINO - Allow reading models from from memory (stream interface), not only file
- Dmitry - OpenVINO models as shared pool (in graph / for multiple graphs)
  - Dmitry - should be controllable (an option), as it may alter the "multi-context" cases.
  - Tolik - the input data precision may cause a new instance to be initiated.

### Operations

- Dmitry - Object tracking functionality still isn't there by default

### Documentation

- Doxygen documentation refined, redesigned (for new goals)
  - The original 4.0 documentation was written with image processing optimizations in mind, need to refocus on AI pipelines
- C++ Tutorial update
- Python Tutorial (GSoc)

## What we'd like to fix

- Tolik - Need to fix Python bindings for Python 3.7+ - fails on Windows. Applies to OpenCV in general, but we're affected most.