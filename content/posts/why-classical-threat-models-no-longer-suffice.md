---
title: "Why classical threat models no longer suffice for AI systems"
date: 2026-05-31
draft: false
tags: ["security", "ai", "threat-modeling"]
series: ["Foundation"]
summary: "Classical threat modelling didn't fail. It met a system it wasn't designed for."
---

A security engineer sits down to threat-model a legal document analysis pipeline that runs on a large language model. They open their STRIDE template and stall at *Tampering*. The binaries are signed. Transit is TLS. Database writes are authenticated. And yet, the runtime has been hijacked: someone uploaded a benign-looking PDF with crafted instructions hidden in invisible text, and the model dutifully exfiltrated private user sessions. From the perspective of classical threat modelling, every data flow is authorised, every file is intact, no exploit code ran. But the control flow of the application is gone.

This is the friction point I want to talk about. The issue isn't that classical threat modelling has become obsolete — it's that we're asking it to evaluate a different kind of system. STRIDE, DREAD, PASTA didn't fail. They met a probabilistic system, and their categories don't quite fit.

## The foundations of classical threat modelling

Before getting to where the gap opens, it's worth being clear about what classical threat modelling does well.

Decades of application security practice have produced structured methods for catching design flaws early. STRIDE remains the most widely used. It categorises threats as Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, or Elevation of Privilege, surfaced by walking the data flows and trust boundaries of a system. DREAD is the older risk-scoring companion: Damage, Reproducibility, Exploitability, Affected Users, Discoverability. PASTA (Process for Attack Simulation and Threat Analysis) is the more business-aligned, attacker-centric alternative, structured around seven stages.

These methods remain genuinely useful. Most corporate AI deployments are not autonomous neural systems. They are conventional web applications with an API call to a hosted language model in the middle. STRIDE handles those well and it still catches SQL injection in adjacent databases, XSS in presentation layers, and authentication failures at the integration gateway.

The breakdown happens at a single assumption, that a system is made of discrete, deterministic components with well-defined trust boundaries. When one of those components is a language model, that assumption stops being true.

## The paradigm shift in system architecture

Three architectural shifts make LLM applications behave differently from the systems classical threat modelling was designed for.

### The input channel as an execution interface

In classical systems, parsers, schemas, and validators keep user input separate from execution instructions. With language models, that separation collapses. Instructions and data live in the same natural-language context. Prompt injection isn't a parsing error or a syntax bug. It's the model behaving exactly as designed. Interpreting natural-language instructions, regardless of who wrote them.

The pattern comes in two flavours. *Direct* prompt injection is the user overriding system instructions to force unintended behaviour. *Indirect* prompt injection is more interesting: the model processes external data (a summarised web page, an email, an uploaded PDF) and the data itself contains the instructions. Multimodal inputs and adversarial suffixes extend this further. The input layer is no longer just user-controlled. It is anyone-controlled.

### The model as an asset class of its own

In a conventional application, your assets are source code, binaries, configuration files, databases. In an AI system, the model itself is an asset class and a larger one than it first appears. Pre-training data, fine-tuning datasets, weights, system prompts, RAG indexes, agent tool definitions. All of these can be exfiltrated, modified, or poisoned. STRIDE has no column for "the model leaks its own system prompt through legitimate output."

### Emergent, non-deterministic execution

Traditional software is deterministic. Trace the code, follow the control flow, and you can predict the output. Language models don't work that way. Identical inputs can produce different outputs depending on sampling parameters, model state, and context. Behaviour emerges statistically from training data, not procedurally from code. That means threat models can't assume there is a specification to test against.

## The modern security vocabulary

Three specialised frameworks have emerged to fill these gaps. They aren't competing — they're complementary, and each addresses a different layer.

### OWASP Top 10 for LLM Applications

The **[OWASP Top 10 for LLM Applications (2025)](https://genai.owasp.org/llm-top-10/)** is the direct analogue to the classic OWASP Top 10 for web apps. It catalogues the ten most critical risks for deployed LLM systems and is aimed squarely at application security teams. Its categories include Prompt Injection (LLM01), Sensitive Information Disclosure (LLM02), Supply Chain Vulnerabilities (LLM03), Data and Model Poisoning (LLM04), Improper Output Handling (LLM05), Excessive Agency (LLM06), System Prompt Leakage (LLM07), Vector and Embedding Weaknesses (LLM08), Misinformation (LLM09), and Unbounded Consumption (LLM10).

### MITRE ATLAS

Where OWASP asks *"what can go wrong,"* **[MITRE ATLAS](https://atlas.mitre.org/)** (Adversarial Threat Landscape for Artificial-Intelligence Systems) asks *"how would an adversary make it happen."* Modelled after MITRE ATT&CK, ATLAS catalogues tactics, techniques, and procedures observed in real attacks and red-team work. As of the November 2025 update, the framework covers 16 tactics and 84 techniques.

More relevant for what's coming: the February 2026 update extended ATLAS into agentic systems. New techniques around API exploitation patterns, tool credential harvesting, tool-mediated data poisoning, and data destruction via agent tool invocation. The recurring theme is that the agent's tool surface becomes the attack surface.

### NIST AI Risk Management Framework

The **[NIST AI Risk Management Framework](https://airc.nist.gov/AI_RMF_Knowledge_Base/Playbook)** sits one level higher. It isn't a threat list. It's a governance scaffold structured around four functions Govern, Map, Measure, Manage and covering the full system lifecycle. The Generative AI Profile (NIST AI 600-1) translates this into around 200 suggested risk-management actions specific to generative systems, including dual-use risks, confabulation, and systemic bias.

| Framework | Primary audience | Core function | Analytical contribution |
|---|---|---|---|
| STRIDE / PASTA | Security architects, software engineers | Trust boundaries, component decomposition, traditional data flows | Maps the conventional web infrastructure surrounding the model |
| OWASP Top 10 for LLM Applications | AppSec teams, developers | Application-layer risks | Catalogues which integration-layer risks need mitigating |
| MITRE ATLAS | Red teams, threat intelligence | Adversary tactics and techniques | Details how attackers actually exploit model architectures and agent tools |
| NIST AI RMF | CISOs, auditors, governance practitioners | Socio-technical risk, lifecycle, policy | Frames corporate risk tolerance, compliance, and measurement |

These frameworks operate alongside classical methods rather than replacing them. STRIDE still maps the data flows of the surrounding ecosystem. OWASP, ATLAS, and the NIST AI RMF extend the practitioner's vocabulary at the precise interface points where classical assumptions break down.

## Operational implications for security teams

Adopting this expanded vocabulary changes three things in day-to-day security work.

### A new participant in threat-modelling sessions

Threat-modelling sessions need a new participant. Someone who understands how the model actually behaves: token generation, fine-tuning effects, context window limits, model state. In a larger organisation, that's a dedicated AI security engineer. In a smaller one, it's an AppSec engineer who has to grow into the role. Either way, the conversation doesn't work without that voice in the room.

### An expanded asset inventory

Asset management has to evolve beyond cataloguing virtual machines, containers, and databases. Model-side assets like system prompts, retrieval indexes, fine-tuning datasets, agent tool definitions or orchestration configurations need to be inventoried, classified, and protected the same way every other asset is. If they aren't catalogued, they can't be classified. If they can't be classified, they can't be protected proportionally. This is the foundation everything else rests on.

### The triage question

Historically, any project with an AI component was routed to the AppSec queue. That now needs a triage step: is this a web app with an LLM call inside it, or an LLM-native application with direct tool access? The two need different reviews. The first largely fits existing AppSec pipelines, the second needs a process focused on privilege boundaries, tool permissions, and runtime monitoring.

## A note before the rest of the series

None of these adjustments are particularly novel. Most security organisations already have variants of them. What's new is the deliberate work of mapping them onto a different kind of system — one without a deterministic specification, with input channels that double as execution channels, and assets that don't show up in any existing inventory. The vocabulary exists. The frameworks are public. What's missing is the habit of looking at AI systems through them.

That's what the rest of this series is about.

## The Foundation series roadmap

The next posts translate these shifts into something more concrete:

- **Post 2 — Mapping STRIDE to AI-specific threats.** A translation matrix that bridges legacy vulnerability categories with the new adversarial frameworks.
- **Post 3 — Model security as a protection object.** A look at modern asset inventories: what changes when models, weights, and embeddings become protection objects in their own right.
- **Post 4 — Pragmatic risk profiling.** A one-page method that scales from internal search tools to fully autonomous agents.
- **Post 5 — Ten questions for a fast AI security assessment.** A short triage checklist designed to classify any AI use case.
- **Post 6 — Enterprise realities of AI risk.** Which AI risks actually matter at enterprise scale, and which are mostly literature.

---

*Sources:*

- *[OWASP Top 10 for LLM Applications (2025)](https://genai.owasp.org/llm-top-10/)*
- *[MITRE ATLAS](https://atlas.mitre.org/)*
- *[NIST AI Risk Management Framework + GenAI Profile](https://airc.nist.gov/AI_RMF_Knowledge_Base/Playbook)*
