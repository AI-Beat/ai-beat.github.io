# AI Headlines — 2026-04-19

- [Zero-Copy GPU Inference from WebAssembly on Apple Silicon](https://abacusnoir.com/2026/04/18/zero-copy-gpu-inference-from-webassembly-on-apple-silicon/) — Blog post on building a Wasmtime-based runtime that shares its linear memory directly with Metal GPU buffers by exploiting Apple Silicon's unified memory architecture; eliminates copy overhead entirely (0.03 MB vs 16.78 MB), achieves ~9 ms/token for Llama 3.2 1B on an M1 MacBook, and enables KV cache snapshots that restore 5.45× faster than recomputing prefill. *(April 18, 2026)*
