# llm-vision

[![CI](https://github.com/Mattbusel/llm-vision/actions/workflows/ci.yml/badge.svg)](https://github.com/Mattbusel/llm-vision/actions/workflows/ci.yml)
![C++17](https://img.shields.io/badge/C%2B%2B-17-blue.svg)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

**Ask OpenAI and Anthropic vision models about images, from C++.** One header, `llm_vision.hpp`. Needs libcurl (`apt install libcurl4-openssl-dev`, preinstalled on macOS, `vcpkg install curl` on Windows).

Screenshots, charts, receipts, photos from a camera: vision models can read them, but calling them means base64 encoding, multipart message formats and provider differences. llm-vision loads the image, builds the request for either provider and returns the answer, optionally streamed.

## Features

- `load_image()` reads a file; images are sent inline as base64 with their MIME type
- Or pass an image URL instead of bytes
- Multiple images per request
- OpenAI Chat Completions (`image_url` parts) and Anthropic Messages (`image` source blocks)
- `vision()` returns the full answer; `vision_stream()` delivers tokens through a callback

## Quick start

Copy the header into your project:

```bash
curl -fsSLO https://raw.githubusercontent.com/Mattbusel/llm-vision/main/include/llm_vision.hpp
```

Define `LLM_VISION_IMPLEMENTATION` in exactly one `.cpp` file before including it; every other file just includes the header. Save this as `main.cpp` next to the header:

```cpp
#define LLM_VISION_IMPLEMENTATION
#include "llm_vision.hpp"
#include <cstdlib>
#include <iostream>

int main() {
    const char* key = std::getenv("OPENAI_API_KEY");
    if (!key) { std::cerr << "set OPENAI_API_KEY\n"; return 1; }

    llm::VisionConfig cfg;
    cfg.api_key  = key;
    cfg.model    = "gpt-4o";
    cfg.provider = llm::VisionProvider::OpenAI;   // or VisionProvider::Anthropic

    // Local file: sent inline as base64.
    auto img = llm::load_image("screenshot.png", "image/png");
    std::cout << llm::vision("What error is shown in this screenshot?", {img}, cfg) << "\n";

    // Remote image by URL, streamed token by token.
    llm::VisionImage remote;
    remote.url = "https://example.com/chart.png";
    llm::vision_stream("Describe this chart in one sentence.", {remote}, cfg,
                       [](const std::string& d) { std::cout << d << std::flush; });
}
```

```bash
g++ -std=c++17 -O2 main.cpp -lcurl -o demo
```

## API at a glance

| Call | Purpose |
|---|---|
| `load_image(path, mime_type)` | Read a file into a `VisionImage` |
| `vision(prompt, images, cfg)` | Send and wait for the answer |
| `vision_stream(prompt, images, cfg, on_delta)` | Stream the answer |
| `VisionConfig{api_key, model, provider, max_tokens, temperature}` | Defaults: `gpt-4o`, OpenAI |

## Notes and limitations

- `provider` is set explicitly in `VisionConfig`; it is not inferred from the model name.
- The header defines `NOMINMAX` so it is safe to include alongside `<windows.h>`.

## Build the examples

The repo builds `examples/describe_image.cpp`, `examples/compare_images.cpp`, `examples/url_image.cpp`, `examples/stream_vision.cpp` with CMake (requires libcurl):

```bash
cmake -B build
cmake --build build
```

## Part of llm-cpp

llm-vision is one of 26 single-header C++ libraries in [llm-cpp](https://github.com/Mattbusel/llm-cpp), a toolkit for building LLM features into native code. Each library stands alone; combine them by giving each `*_IMPLEMENTATION` define its own `.cpp` file. See the [llm-cpp README](https://github.com/Mattbusel/llm-cpp#using-several-together) for the full list and examples of using several together.

## License

MIT. See [LICENSE](LICENSE).
