---
title: "The Agent Without a Toolkit"
date: 2026-07-12T10:48:00+00:00
draft: false
slug: lisp-agent-eval-tool
categories: [agents]
tags: [agents, lisp, eval, open-source, minimal]
params:
  author: AI Beat Desk
  summary: >-
    A post from July 7 builds an AI agent in ~100 lines of Common Lisp
    with exactly one tool: eval. The model writes Lisp code that gets
    executed directly; capabilities persist across sessions by
    re-evaluating function definitions stored in the JSON transcript.
    The model spontaneously built a web search client from scratch when
    given API credentials.
---

A [post from last week](https://thebeach.dev/posts/lisp-agent/) builds an AI agent in about 100 lines of Common Lisp. The line count isn't the interesting part. The interesting part is that the agent gets exactly one tool: `eval`.

The conventional approach to agent construction is to enumerate capabilities ahead of time. You decide the agent will need web search, so you write a search function and expose it as a JSON schema. You decide it will need file access, so you write that too. Every action the agent can take exists as a discrete function you defined before deployment. If you didn't think to include it, the agent can't do it. The toolkit is closed.

This post throws that assumption out entirely. Instead of a predefined set of tools, the author provides a single function that evaluates arbitrary Common Lisp:

```lisp
(defun lisp-eval (form-string)
  (handler-case
      (format nil "~s" (eval (read-from-string form-string)))
    (error (e) (format nil "ERROR: ~a" e))))
```

The model writes Lisp code as a string. The agent reads it, evaluates it, and returns the result. That's the entire tool surface. In Lisp this works with unusual cleanliness because of homoiconicity: Lisp programs are represented as Lisp data structures (lists of lists), so the boundary between "code the model generates" and "data the agent processes" is structurally nonexistent. When the model produces a function definition, the agent evaluates it into the running image immediately. The model can then call that function in subsequent tool invocations.

The agent loop itself is recursive — eight lines:

```lisp
(defun agent-loop (messages)
  (let* ((message (ref (call-model messages) "choices" 0 "message"))
         (tool-calls (gethash "tool_calls" message)))
    (if (and tool-calls (plusp (length tool-calls)))
        (agent-loop (append messages
                            (list message)
                            (map 'list #'execute tool-calls)))
        (append messages (list message)))))
```

Either the model returns a response (base case, return the enriched message list) or it requests tool calls (recursive case, execute them and continue with the results appended). There's no state machine, no session object, no framework.

## Memory as transcript

The more compelling part is the memory system. The agent serializes its entire message history to JSON after each exchange. On a fresh session, it loads this transcript and re-evaluates the Lisp forms embedded in it — function definitions included. The author describes this as: "skills are stored as memories of having built them, re-hydrated on demand."

This means a function the model wrote in a previous session — perhaps a specific parser, or a wrapper for some API — comes back into the running image in subsequent sessions by being literally re-executed from the transcript. There's no separate "skill store" or "tool registry." The JSON message history *is* the tool registry. The serialization format is also the runtime format.

The most striking demonstration: the author gave the agent API credentials for a web search service, with no predefined search tool. The model inspected what Lisp libraries were available (dexador for HTTP, shasht for JSON), wrote a function to construct the API request, handled the response format, and integrated results into its reasoning. By the end of the session, there was a working `brave-search` function in the image that the agent had built for itself from nothing:

```lisp
(defun brave-search (api-key query &key (count 10))
  (let* ((encoded-query (quri:url-encode query))
         (url (format nil "https://api.search.brave.com/res/v1/web/search?q=~a&count=~d"
                      encoded-query count))
         (response (dexador:get url
                                :headers `(("Accept" . "application/json")
                                          ("X-Subscription-Token" . ,api-key)))))
    (shasht:read-json response)))
```

## The obvious objection

`eval` as a tool means the model runs arbitrary code on your machine. The author is explicit: "This is a toy for a sandbox. I haven't run this outside of a local docker container." That's the right framing. There's also a practical limit: the transcript grows unbounded, eventually exceeding context windows in long sessions.

These are genuine limitations, and they're probably why most agent frameworks go the other direction: enumerate tools carefully, constrain the model's action surface, expand the toolkit deliberately. That approach has real value — it makes agents auditable, their action space bounded and inspectable.

But the Lisp agent points at something worth sitting with: what the conventional approach gives up is emergent capability. An agent with a fixed toolkit can only do what you thought to give it. An agent with eval can, in principle, build any capability that can be expressed in code — given a runtime, a package manager, and the right initial libraries. The question of whether that's useful outside a sandbox isn't really the point. That it works at all — cleanly, in a hundred lines — is the interesting data point.
