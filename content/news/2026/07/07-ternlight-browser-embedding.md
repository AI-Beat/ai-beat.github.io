---
title: "Seven Megabytes of Semantic Search"
date: 2026-07-07T06:09:27+00:00
draft: false
slug: seven-megabytes-of-semantic-search
categories: [inference]
tags: [embeddings, wasm, browser, edge-inference, ternary-weights]
params:
  author: AI Beat Desk
  summary: >-
    Ternlight ships a sentence embedding model as a 7MB WASM bundle that runs
    on CPU in the browser — no API, no model download, no GPU required. Ternary
    weights are the key to the footprint; the result is semantic search you can
    include in an npm install.
---

Seven megabytes is small enough to ship with a JavaScript bundle. That's the design premise behind [Ternlight](https://ternlight-demo.vercel.app/), which landed on npm today: a sentence embedding model that fits in a single `npm install @ternlight/base`, runs on CPU via WebAssembly, and produces 384-dimensional vectors in roughly 5 milliseconds — entirely in the browser, no API call required, no separate model download step.

The size comes from ternary weights. Instead of 16-bit floats, Ternlight's weights are stored as -1, 0, or 1 using BitLinear layers, the same quantization family popularized by BitNet research. This allows efficient bit-packing and makes the WASM module self-contained: the bundle includes the inference engine, the quantized weights, and the tokenizer together. A `@ternlight/mini` variant cuts the footprint to 5MB for tighter budgets.

The practical surface area is narrow but genuine. Semantic search over a local dataset, FAQ and intent matching, client-side similarity filtering — anything that needs embeddings but where sending text to a remote server is either a privacy concern, a latency problem, or a cost you'd rather not pay. The demo runs a semantic search over the React documentation entirely client-side, which gives a reasonable sense of the quality ceiling.

What Ternlight isn't: a substitute for cloud embedding APIs when retrieval quality is critical. Models like OpenAI's text-embedding-3-small produce 1536-dimensional vectors trained on orders of magnitude more data; the gap in recall on a large, nuanced corpus will be real. For a documentation search, a local FAQ matcher, or a semantic filter that runs before a more expensive cloud call, 384 dimensions at 5ms is probably exactly right. For precise retrieval over a billion documents, probably not.

The timing is reasonable. Privacy regulations and offline-first architectures have been pushing more processing to the edge, and embedding models are often the last component to move there — they've historically been too large to bundle without complaint. A 7MB WASM module crosses the threshold where including it in a web app is uncontroversial; it's smaller than many web font files. The npm packaging means it integrates into standard JavaScript build tooling without ceremony.

The ternary weight approach also has an underappreciated advantage beyond size: integer arithmetic on ternary values maps well onto CPU SIMD instructions, which is why the 5ms latency figure holds on ordinary laptop CPUs without any GPU or dedicated hardware. For most embedding use cases, inference latency isn't the bottleneck, but it's useful to know there's no hidden cost from running in WASM vs. native.

Whether Ternlight's embedding quality is competitive with what a hosted API returns at the 384-dimension tier is a question worth benchmarking against your actual retrieval task before committing. The demo is convincing for short-form semantic search; performance on longer documents or specialized domains will vary.
