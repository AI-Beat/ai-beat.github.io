---
title: "How OpenAI Ran WebRTC Through Kubernetes"
date: 2026-05-05T06:18:09Z
draft: false
slug: openai-voice-webrtc-kubernetes
categories: [infrastructure]
tags: [openai, voice, webrtc, kubernetes, infrastructure, realtime]
params:
  author: AI Beat Desk
  summary: >-
    OpenAI published a detailed engineering writeup on how they rebuilt their
    WebRTC stack for the Realtime API to run on Kubernetes at scale — separating
    a lightweight UDP relay from the stateful WebRTC transceiver and using the
    ICE ufrag as a routing hook embedded in standard protocol headers.
---

Most AI infrastructure posts are benchmarks or diagrams. OpenAI's [writeup on delivering low-latency voice AI at scale](https://openai.com/index/delivering-low-latency-voice-ai-at-scale/), published Monday and sitting at the top of Hacker News since morning, is something more useful: a detailed account of a real engineering problem and how they solved it.

The problem is WebRTC in Kubernetes. The two don't fit together naturally. Standard WebRTC requires each session to own one or more public UDP ports, which works fine when you have a fixed set of servers but becomes awkward when your infrastructure is Kubernetes pods that can be scheduled, rescheduled, scaled up, and torn down. Large public UDP port ranges are difficult to expose, difficult to secure, and create sticky routing requirements — a session can't move to a different pod without renegotiating ICE, which involves a round-trip through the client. When you're trying to keep voice latency in the 300–500ms range across 900 million weekly active users, you'd rather not renegotiate anything.

OpenAI's solution is a split between two services: a relay and a transceiver.

The relay is a lightweight UDP forwarding layer written in Go. It runs with a small public footprint — a fixed, manageable UDP surface — and its only job is to forward packets to the right transceiver. Critically, the relay never decrypts media, never runs an ICE state machine, and never participates in codec negotiation. It reads packet headers from a socket, updates a small amount of flow state, and forwards. The relay is essentially stateless relative to the WebRTC protocol itself.

The transceiver is the stateful WebRTC endpoint. It lives behind the relay, doesn't need public UDP exposure, and handles everything the relay deliberately avoids: ICE, DTLS, codec negotiation, the actual audio processing pipeline.

The clever part is routing. How does the relay know which transceiver to forward a given UDP packet to? It uses the ICE username fragment — the ufrag — which is already carried inside WebRTC packets as a protocol-native field. OpenAI generates ufrags that contain just enough routing metadata for the relay to extract the destination cluster and owning transceiver, without adding any fields to the packet or modifying client behavior. From the client's perspective, this is standard WebRTC. The routing logic is embedded in a field the client already has to set.

The result: relays can scale independently of transceivers, transceivers can run as ordinary Kubernetes pods without special UDP port allocation, and the overall public network surface stays small and fixed. First-hop latency stays low because relay pods can be placed globally with a small footprint, routing packets to transceivers wherever they're scheduled.

This is a clean solution to a real problem, and it's worth reading for the architecture alone. But there's a broader point worth noting. WebRTC was designed in an era when the server side was assumed to be a fixed fleet of machines, not a dynamically scheduled container environment. The assumptions baked into the protocol — especially around port ownership and ICE state machines — don't match what production Kubernetes deployments look like. OpenAI needed a protocol-compatible shim that separated the "where do UDP packets go" problem from the "who handles the WebRTC session" problem.

This kind of adapter pattern — a thin, dumb forwarding layer that makes an older protocol compatible with modern infrastructure — shows up constantly in production systems. The interesting engineering is usually in finding a routing key that's already in the packet. Here it was the ufrag. In other contexts it's the session ID in a cookie, a service token in a TLS SNI extension, or a routing tag in a Kafka partition key. The constraint that the relay must not modify client behavior is what forces the elegant solution; if you can change the client, the problem becomes trivial and the answer becomes uninteresting.

The post doesn't cover failure modes or what happens when a transceiver crashes mid-session — whether ICE renegotiation is needed, whether the relay can reroute to a new transceiver transparently, or whether the client notices. Those would be the interesting edge cases to know about. But for an explanation of the core architecture, it's unusually clear.
