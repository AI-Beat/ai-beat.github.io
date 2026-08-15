---
title: "ML on Ciphertexts"
date: 2026-08-15T07:00:00Z
draft: false
slug: heir-private-ai-inference
categories: [research]
tags: [privacy, homomorphic-encryption, google, inference]
params:
  author: AI Beat Desk
  summary: >-
    Google's August 14 post on HEIR describes four production deployments of
    homomorphic encryption for ML inference — fraud detection, recommendations,
    network intrusion detection, hotword detection — with hardware accelerator
    partnerships starting to close the latency gap that has kept HE in the lab
    for the past decade.
---

Homomorphic encryption has an unusual reputation in cryptography circles: it's the idea everyone finds beautiful, few find usable. The basic premise is elegant — computations performed on ciphertext produce an encrypted result that, when decrypted, equals what you'd have gotten by computing on the plaintext directly. You can run a neural net on data you've never seen. The catch has always been performance: operations on ciphertexts are orders of magnitude slower than on plaintext, and the gap has historically made HE a lab curiosity rather than a production tool for ML inference.

Google's [August 14 blog post](https://blog.google/security/how-google-is-making-private-ai-practical-with-homomorphic-encryption/) describes a different situation. [HEIR](https://github.com/google/heir) (Homomorphic Encryption Intermediate Representation) is an open-source, MLIR-based compiler that converts pre-trained neural networks — models that already work on unencrypted data — into versions that operate on encrypted inputs. The compiler targets multiple HE schemes (BGV, BFV, CKKS) and multiple backend libraries (OpenFHE, Lattigo, tfhe-rs). The approach is significant because it removes the need to hand-implement HE operations from scratch: you train a model normally, then compile it to a cipher-native variant.

Four production applications are now running on HEIR:

**Recommendation systems** — a deep learning recommender that selects content without the server ever observing the user's feature vector. Developed with Belfort Labs, LG, and NYU.

**Credit card fraud detection** — transaction screening run in partnership with Niobium and hardshell.ai, where the card network can flag suspicious patterns without reconstructing the full transaction history in plaintext.

**Network intrusion detection** — the [Kitsune](https://arxiv.org/abs/1802.09089) anomaly detection system adapted to run over encrypted traffic features.

**Hotword detection** — audio processed on-device in encrypted form, so the wake-word model never receives plaintext audio.

These are all inference tasks, which is the easier case for HE: the weights remain on the server (or device), and only the input is encrypted. Training with HE is a much harder problem and isn't on the table here. But inference is where the privacy concern is most acute in production systems — you don't want to send plaintext audio to a cloud service just to check if someone said a wake word, or expose a patient's lab values to a cloud diagnostic model.

The latency gap is real and the post is honest about it, if vague on specifics. That's where the hardware partnerships come in. Belfort, Niobium, Cornami, and Optalysys are all building accelerators targeting HE workloads — ASICs and FPGAs designed around the specific arithmetic that HE requires (polynomial multiplication in large rings). Software-only HE inference is still slow enough to be painful for low-latency use cases. With dedicated silicon, the story changes: what's currently measured in seconds per query can, on specialized hardware, drop to millisecond range.

The MLIR foundation matters for long-term viability. MLIR is a compiler framework designed precisely for this kind of multi-backend situation: you define your computation once, and the framework handles lowering to whatever target you need. HEIR sits in that ecosystem, which means hardware vendors can add HEIR backends as they ship new silicon without requiring Google to write new compiler backends from scratch. That's a reasonable bet on longevity.

Whether this constitutes HE "going practical" depends on your definition of practical. Four deployed systems is not zero, and the hardware investment from multiple silicon companies suggests the bet is real. But "practical for hotword detection on specialized hardware at Google scale" and "practical for your ML startup's API" are very different thresholds. The honest summary is that HE for inference has crossed from "pure research" to "works in production under specific conditions with hardware support." The second half of that sentence still involves significant infrastructure investment.

HEIR is [on GitHub](https://github.com/google/heir) under an open license. The heir_py Python package is the entry point for getting a pre-trained model through the compiler.
