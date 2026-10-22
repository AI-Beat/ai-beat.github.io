---
title: "The AI Toolchain as Attack Surface"
date: 2026-10-22T06:14:00+00:00
draft: false
slug: litellm-supply-chain-attack
categories: [security]
tags: [supply-chain, security, litellm, pypi]
params:
  author: AI Beat Desk
  summary: >-
    LiteLLM's PyPI release was backdoored on October 21 with a credential stealer
    that targets AI developer tokens specifically — Claude Code, Cursor, Copilot,
    LLM provider API keys — alongside the usual cloud credential haul. It's the
    second major supply chain hit on LiteLLM in 2026, and the .pth file technique
    used here is the one detail every Python developer should understand.
---

On October 21, 2026, between 11:39 and 11:40 UTC, three malicious wheels for LiteLLM 1.107.0 were uploaded to PyPI. Six minutes later, matching Docker images appeared on Docker Hub. Both were removed within two hours. Anyone who installed LiteLLM during that window should assume their host's credentials are compromised.

[LiteLLM](https://github.com/BerriAI/litellm) is a widely-used Python library that routes requests across LLM providers behind a unified OpenAI-compatible API. Its proxy server is common infrastructure in teams that run multiple models. That makes it an attractive supply chain target, and this is the second major attack on it in 2026 — the [first landed in March](https://www.herodevs.com/blog-posts/the-litellm-supply-chain-attack-what-happened-why-it-matters-and-what-to-do-next), using credentials that a backdoored Trivy scanner had harvested from LiteLLM's own CI weeks earlier.

**How it got in.** The attackers obtained GitHub CLI credentials from three LiteLLM developers, used those to redirect the release workflow to a lookalike repository, and slipped past the approval gate with a second developer's credential. The upstream GitHub repository was clean throughout; the malice lived only in the built artifacts.

**The .pth trick.** The payload consisted of three files dropped into `site-packages`: `cook.pth`, `cook.py`, and `sitecustomize.py`. The `.pth` file is the sharp edge here. Python processes every `.pth` file in `site-packages` at interpreter startup — not at import time, not when litellm is first used, but every time any Python process starts on that machine. `cook.pth` contains a single line of code that imports and runs `cook.py`. The result is that uninstalling litellm is not enough; anything that ran Python on a host where 1.107.0 was installed has already been affected.

`cook.py` is the stealer. It goes for the full catalog: SSH private keys, cloud provider credentials (AWS, Azure, GCP, Kubernetes, Docker, GitHub CLI), CI/CD secrets (GitLab, Jenkins, Vault, Terraform), browser saved passwords and shell history, environment variables, and — pointedly — AI developer tool tokens from Claude Code, Cursor, and Copilot. If the host was running a LiteLLM proxy, it also reads API keys out of the proxy's database. Everything is encrypted and sent to `www.litellm[.]dev`, a domain the attackers controlled.

`sitecustomize.py` is a daemon that starts on Python startup and runs again hourly. It reads a server address from a file on GitHub, re-runs the stealer, and downloads and executes arbitrary code from that server. The GitHub account was deleted at 13:40 UTC, so currently the daemon finds nothing. Hosts that ran before the deletion may have received and executed second-stage payloads that SafeDep could not recover.

**The AI-specific angle.** Supply chain attacks against Python packages are not new, and the credential categories here mostly overlap with standard developer compromise. What's notable is that the target list explicitly calls out AI developer tokens as a category. Claude Code, Cursor, and Copilot all authenticate with short-lived tokens stored in dotfiles or credential managers, and a compromised token typically grants full access to the user's billable API context. For teams running a self-hosted LiteLLM proxy, losing the proxy's database is worse — it may contain production API keys for every model the organization uses.

**What to do.** Versions 1.106.2 and the 1.107.0 release candidates are clean. If you installed 1.107.0 — or `pip install litellm` pulled it automatically on October 21 — rotate everything: SSH keys, cloud provider credentials, CI/CD tokens, AI tool credentials, and anything else the host had access to. Check for `cook.pth`, `cook.py`, and `sitecustomize.py` in your Python environment's `site-packages`. LiteLLM has published an incident report with detailed indicators of compromise.

The deeper issue is the `site-packages` injection pattern itself. A `.pth` file is processed without any import, which means standard dependency scanning and linting catch nothing. It also survives across virtual environments if the package was installed into a system Python. The only reliable defense at the CI/CD level is reproducible builds and provenance verification — checking that the artifact you downloaded matches the source you reviewed.
