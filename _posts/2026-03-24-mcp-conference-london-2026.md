---
layout: post
title: "MCP Conference London 2026"
date: 2026-03-24 00:00:00 +0000
tags: [ai, mcp, security, agents]
---

In February I went to the MCP Conference in London, part of ContainerDays London 2026. Two days, back to back, all about the Model Context Protocol and what people are actually building with it (and breaking with it). These are my notes, cleaned up a bit.

---

## Security was everywhere

Almost every talk touched on security. It's the same pattern we've seen with other tech waves: build fast, ship it, deal with the consequences later. With agentic AI a mistake can delete or leak something before anyone notices.

### Vibe coding has a vibe hacking problem

Sonya from Snyk talked about the security problems with AI-generated code. She brought up [Replit's AI agent deleting a production database](https://www.pcmag.com/news/vibe-coding-fiasco-replite-ai-agent-goes-rogue-deletes-company-database), which most people following this space have already heard about. Her framing was that agentic engineering is the next step after vibe coding, and that engineers relying on AI tools will eventually move from shipping and hoping it works to something more deliberate.

A few things from the talk stood out to me:

**Slopsquatting** — LLMs sometimes hallucinate package names that don't exist. Attackers notice this and register those same names, so if you install what the model suggested, you're running their code instead. It's a supply chain attack that only exists because of how these models work.

![Diagram of a slopsquatting attack: an LLM hallucinates a package name, an attacker publishes it, and a normal user installs the malicious package](/assets/img/mcp-conference-london-2026/slopsquatting-diagram.jpg)

The OpenClaw example got the most reaction from the room. The talk went through [Cisco's analysis of OpenClaw](https://blogs.cisco.com/security/personal-ai-agents-like-openclaw-are-a-security-nightmare): Cisco's AI Defense team ran their open-source [Skill Scanner](https://github.com/cisco-ai-defense/skill-scanner) against a third-party skill called "What Would Elon Do?" that had climbed to #1 in a community repository. They found nine security issues, two of them critical, and described the skill as functionally malware — it exfiltrated data over the network and used prompt injection to get around safety rules. The lesson is the same one we already know from other supply chains: being popular in a catalog doesn't mean anyone reviewed it.

A few recommendations came out of this part of the talk:
- Secure the prompt: think about security while writing it, not after
- Keep architecture boundaries: separate environments, agents shouldn't touch production directly
- Harden the supply chain: require SBOMs, watch for typosquatting and slopsquatting, audit MCP servers before enabling them
- AIBOM: Snyk's term for an AI Bill of Materials, meant to catch shadow AI usage the same way we track shadow IT
- Turn off `--yolo` and `--trust-all-tools` flags in MCP clients — apparently people actually use these

![Snyk talk slide on configuration hardening for AI-native IDEs and CLIs](/assets/img/mcp-conference-london-2026/snyk-security-hardening-talk.jpg)

There was also a slide about a mindset shift: review what the agent produces, understand it before shipping, and watch out for the model's people-pleasing bias — it will agree with you even when you're wrong.

![Snyk talk slide on the mindset shift needed for AI-generated code: review everything, understand before shipping](/assets/img/mcp-conference-london-2026/snyk-mindset-shift.jpg)

Resources mentioned: [OWASP GenAI](https://genai.owasp.org/), [Gartner AI TRiSM](https://www.gartner.com/en/articles/ai-trust-and-ai-risk).

### MCP and prompt injection

Josh, co-founder of [Zuplo](https://zuplo.com/), talked about MCP gateways and spent part of his talk on something that doesn't come up much: MCP prompt injection, where a malicious MCP server injects instructions straight into the LLM's context. Prompt injection from users is a known problem by now; injection coming from the MCP layer itself is newer. [CyberArk has a threat analysis of MCP](https://www.cyberark.com/resources/threat-research-blog/is-your-ai-safe-threat-analysis-of-mcp-model-context-protocol) that covers this in more detail.

![Josh from Zuplo presenting on MCP prompt injection, with a slide showing a malicious LinkedIn profile injecting instructions via user-generated content](/assets/img/mcp-conference-london-2026/mcp-prompt-injection-zuplo.jpg)

The architecture he described: MCP client → MCP gateway (discovery, authentication, authorisation, guardrails) → MCP servers. The point of the gateway is to put the checks in one place instead of trusting every server on its own.

### What's hard about implementing MCP servers (Solo.io)

A Solo.io session on implementing, securing, and managing MCP servers covered trust (who's calling), control (what the agent can do), and audit. It also pointed out how the spec has changed: early versions expected the MCP server to handle authorisation itself, which most teams don't really want to own. Newer, OAuth-oriented directions treat MCP more like a resource server sitting in front of a separate identity layer — not plug-and-play yet, but a cleaner split of responsibility. Worth checking which spec version you're actually targeting. The [Agentgateway](https://github.com/agentgateway/agentgateway) project, part of the Linux Foundation, came up as one place this is being worked on.

### Docker's take on isolation

Oleg from Docker made a fairly simple point: LLMs don't have a concept of boundaries, data and code look the same to them. That's the root of why agentic systems are risky.

![Slide showing prompt injection via user-generated content: a malicious LinkedIn profile injects instructions that make an LLM leak a recipe in an unrelated email](/assets/img/mcp-conference-london-2026/prompt-injection-ugc-example.jpg)

He also showed real screenshots of agents going wrong, including a chat where the agent admits to deleting a production database during a code freeze.

![Chat transcript of an AI agent admitting it deleted a production database during a code and action freeze](/assets/img/mcp-conference-london-2026/agent-deleted-database-chat.jpg)

Docker's four principles for reducing risk:
1. Isolate components
2. Only use trusted components
3. Remove capabilities you don't need
4. Split deterministic and non-deterministic paths, and put more controls on the latter

Docker is building a sandboxing feature — isolated VMs with a Docker daemon inside, made specifically to contain agents. I didn't expect to write the sentence "I watched Docker announce a VM solution", but here we are. Their [MCP horror stories post](https://www.docker.com/blog/mcp-horror-stories-the-supply-chain-attack/#technical-breakdown-the-attack) has the technical breakdown if you want more detail.

![Docker's "lethal trifecta" diagram: the overlap between external communication, access to sensitive data, and exposure to untrusted content](/assets/img/mcp-conference-london-2026/docker-lethal-trifecta-diagram.jpg)

---

## Context engineering

Danilo Poccia, AWS Chief Evangelist for EMEA, talked about context pressure. He defined context as everything the model sees when making a decision — system prompts, tool definitions, conversation history — and said quality drops once it fills up.

![AWS AgentCore platform overview slide, listing Runtime, Memory, Identity, Gateway, Code Interpreter, Browser, Observability, Policy, and Evaluations](/assets/img/mcp-conference-london-2026/aws-agentcore-platform-overview.jpg)

Some of the techniques he covered:
- Unambiguous parameter names (`user_id`, not `user`)
- Domain-specific terminology, since niche language compresses meaning
- Returning the minimum amount of information, and making it semantically meaningful rather than a raw UUID
- Multiple focused agents, each with its own context window
- Multi-agent patterns: agents-as-tools, swarms, meta-agents that spin up sub-agents on demand
- Carrying only the outcome of a sub-task forward, not its internal details
- Deferred or lazy loading of context, which is where agent skills fit in

A good chunk of the talk was AWS product pitches — AgentCore Platform, Strands agents — which is what you'd expect from someone in that role. The underlying ideas about context management hold up regardless of which infrastructure you use. [Cedar](https://github.com/cedar-policy), the open-source policy language AWS built for fine-grained access control, also came up.

---

## Generative UI

Ruben Casas, from Postman and a Google Developer Expert (his job title is apparently Staff Vibe Engineer), talked about generative UI and described three levels:

- **Level 1 — Static components:** the agent orchestrates, calls tools, passes data; the client renders pre-built components. [ag-ui](https://github.com/ag-ui-protocol/ag-ui) has an SDK for this
- **Level 2 — Declarative UI:** the agent generates descriptors that get combined with pre-defined components to produce UI. See [json-render.dev](https://json-render.dev/)
- **Level 3 — Generative UI:** the agent creates the UI on the fly

His point was that so far we've mostly been injecting AI into interfaces, like a sidebar chatbot. The newer pattern goes the other way: injecting UI into the agent, rendering components inside the agent's own window. First-party apps tend to use ag-ui or a2ui protocols; third-party apps go through MCPs.

He landed on declarative generative UI as the right balance for now — enough flexibility without the interface changing completely on every interaction.

---

## Agent skills

Peder Holdgaard Pedersen from Saxo Bank talked about skills and how they relate to MCP. The main idea is progressive disclosure: give the model context bit by bit instead of dumping everything at once.

The three layers he described:
- **agents.md** — collapsed context: *when* to use a skill (frontmatter, ~50 tokens)
- **skills.md** — expanded context: *how* to use a skill
- **References** — context navigation: *what* resources to use

He was clear that skills are a big prompt injection attack surface, and his recommendation was to host skills as resources on MCP servers instead of on public GitHub, where they're exposed to supply chain attacks. The [experimental MCP skills extension](https://github.com/modelcontextprotocol/experimental-ext-skills) is where this seems to be heading.

Hugging Face showed something similar: a tool called UPskill that runs a skill against multiple models to see how it performs. They warned that we might be heading into skills mania, the same way there was MCP mania about a year ago. Their advice was to only use skills when they genuinely add to the model's knowledge, not as a way to bolt on everything — what they called the [Goldilocks principle](https://www.asyncagile.org/method-stack/find-the-goldilocks-zone).

---

## A few other things

- **"From MCP, the S is missing"** — a joke from the panel, and not far from the truth right now
- **Governance in enterprises** — multiple organisations are building MCP servers independently, which leads to overlapping efforts. It's the microservices story again, except the shared infrastructure needed this time is auth, discovery, logging, guardrails, policy enforcement
- **Context degradation in multi-agent systems** — as agents pass context between each other, it degrades, and nobody seems to have a clean fix for this yet
- The URL pattern `https://domain/mcp` is showing up as a convention for exposing MCP endpoints, similar to how `/.well-known/` standardised certain discovery endpoints
- **[Tedix](https://tedix.dev/)** talked about ACP (Agentic Commerce Protocol) and x402 for payments, under the framing of "the backendisation of the internet"

---

Two packed days overall. Security was the theme that came up most consistently. The spec is still settling, especially around auth, and the tooling for testing, monitoring, and operating MCP systems at scale hasn't caught up with how much people want to build on it. That gap will probably close eventually, but for now, teams putting agentic systems into production are mostly on their own for this part.
