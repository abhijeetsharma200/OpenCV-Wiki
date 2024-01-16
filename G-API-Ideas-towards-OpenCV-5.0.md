Note: This page is currently brainstormed. The idea of this page is to collect ideas and issues to address to refine G-API for the OpenCV 5.0 release.

Note(2): This page is now a list of linked features or discussions. Please follow the link for details.

## How we see G-API in 5.0

### Core functionality

- Tolik - High-order constructs (conditions and loops)
  - Can retire infer-list and infer-2 if implemented well. Also can work faster with more precise pipelining
- Tolik - 🔲[~~TBB~~Threaded executor](https://github.com/opencv/opencv/pull/24845). Useful for latency case
- Dmitry - Streaming executor
  - Utilizes lots of threads, can be redesigned to use a limited thread pool

### Python

Python should become a first-class interface to G-API:
- 🔲 Full-featured [**pip package**](https://github.com/orgs/g-api-org/discussions/4), with dynamically loadable options when available (e.g., onnx runtime, gstreamer, openvino).
- 🔲 Well-documented:
  - ✅ [GSoC'23](https://github.com/opencv/opencv/wiki/GSoC_2023#idea-g-api-a-complete-python-tutorial) project on G-API Jupyter Notebooks is [now complete](https://github.com/g-api-org/tutorials-py/pull/1).
  - 🔲 Need to wrap up the repo in 5.0 timeframe and import notebooks into OpenCV Documentation.
- 🔲 Extensible:
  - 🔲 [Expose Operations as types](https://github.com/orgs/g-api-org/discussions/5).
  - 🔲 [Expose Operation wrappers](https://github.com/orgs/g-api-org/discussions/6).
  - ☑️ Allow new input sources in Python, for a better Python inter-operability:
    - ❌ [GSoC'23](https://github.com/opencv/opencv/wiki/GSoC_2023#idea-g-api-implement-custom-stream-sources-in-python) wasn't success;
    - ☑️ Can be substituted with [`QueueInput`](https://github.com/opencv/opencv/pull/24178) interface for data and use-case of any type, if wrapped in Python.
- 🔲 Refined:
  - 🔲 [Inference API](https://github.com/orgs/g-api-org/discussions/7)

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
- Tolik - OpenVINO - Use OpenVINO resize (ov::PrePostProcessor) for InferList, InferList2, InferROI
- Dmitry - OpenVINO models as shared pool (in graph / for multiple graphs)
  - Dmitry - should be controllable (an option), as it may alter the "multi-context" cases.
  - Tolik - the input data precision may cause a new instance to be initiated.
- Tolik - Support modern iterative topologies (infer using its own results from past iteration). Like Stable Diffusion. May be implemented as "high order constructs" (see above).
- Dmitry - DL Model introspection https://github.com/orgs/g-api-org/discussions/3

### Operations

- Dmitry - ✅Object tracking functionality still is ~~n't~~ [there by default now](https://github.com/opencv/opencv/pull/24224)! 

### Debugging and profiling

- Dmitry - Debugging accuracy in-pipeline. People often ask what's the way to debug accuracy issues with G-API, esp. if it happens inside the pipeline. As we gained more experience with it recently, 5.0 should be the right time to summarize the debugging story.

  - Tolik - practical example. What we did is we dumped binary data from operation inputs/outputs. Then visualize with Python & compare with reference. What did is actually added extra outputs to graph. Thinks of assigning names to data objects & querying data afterwards. Could be done atop of the existing API. More details:
    - We've had a big pipeline, tried to match its result against a reference. Hypotheses:
      - What if our source produces slightly different data? (a different decode path, etc).
        - Fed pipeline with a custom source producing reference raw data (taken from the original pipeline).
      - What if our infers are configured differently?
        - What if we pass "2nd" infer with the reference data, not with ours? Output mismatched with the reference input, found that there was an issue with infer configuration. E.g., a wrong preprocessing combination?

  - Ruslan - we should separate island-level data from backend-level data.
  - Ruslan - dump data required to produce a certain output (G-API can figure dependencies automatically).
  - Anatoliy - can we write a pass with readable names? Can run this pass & assign names which will be persistent for further queries. 

- Dmitry - Pipeline performance. Same as above, a frequently asked feature. What we have right now is only ITT which may be not the most user-friendly option (in terms of its UX as well as the data provided in there).

### Documentation

- Doxygen documentation refined, redesigned (for new goals)
  - The original 4.0 documentation was written with image processing optimizations in mind, need to refocus on AI pipelines
- C++ Tutorial update
- Python Tutorial (GSoc)

## What we'd like to fix

- Tolik - Need to fix Python bindings for Python 3.7+ - fails on Windows. Applies to OpenCV in general, but we're affected most.
- Dmitry - Test time. Unit tests for G-API run longer than for all other modules (e.g. 5m against 1-2m on the Github Actions Ubuntu machines). Need to revise test scope and tested configurations.
- Dmitry - Performance tests. Need to be revised and restructured for a better reporting.
- Dmitry - exception handling: the non-template `util::throw_error(std::runtime_error(..))` should go!

## What we'd like to remove

- Dmitry - async API, as it is just a wrapper over regular synchronous API, not actively used, but causes issues time to time. Instead, Streaming mode can be extended to support queue-like push-pop interface which will be asynchronous in its sense.
- Dmitry - PlaidML backend - no longer needed.

## Beyond 5.0

- Dmitry - Revive S11N (serialization) as a deployment model for API-reduced environments
- Dmitry - Portable S11N with first-class kernels (written in G-API opset)
- Dmitry - JS API (maybe in a reduced mode to load & run computations via S11N)