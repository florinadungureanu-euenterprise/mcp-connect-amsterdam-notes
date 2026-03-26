# mcp-connect-amsterdam-notes
Notes from MCP Connect Amsterdam (March 2026) exploring the Model Context Protocol, agentic workflows, and real-world trade-offs in production. Focused on orchestration, onboarding, and scaling AI systems.
# Is MCP dead? Notes from MCP Connect Amsterdam - March 24, 2026

The week before this event, posts were circulating declaring MCP finished. A Y Combinator partner shared a hot take. A prominent open-source developer called it over-engineered middleware. The Fast MCP team announced they were holding a funeral in New York on April 1st.

Then ~210 people showed up to an evening event in Amsterdam to talk about it.

Make of that what you will.

---

This repo contains my notes, takeaways, and resources from [MCP Connect Amsterdam](https://lu.ma/mcp-connect-amsterdam), hosted at **AI House Amsterdam** (Zuidas) and co-organised by [GitHub](https://github.com), [Alpic](https://alpic.ai), [Orq.ai](https://orq.ai), and [Prosus](https://www.prosus.com).

> *I'm not an engineer; I'm an ecosystem operator embedded in the Amsterdam AI scene. I took notes, went down rabbit holes, and wrote up what stayed with me. If anything here is useful to you, great.*

---

## Should you still invest in MCP Apps, or go back to window.openai?

This question landed in the [OpenAI developer forum](https://community.openai.com/t/mcp-apps-in-chatgpt-are-fundamentally-broken-2-critical-bugs/1377697) the same week as the event. A developer building MCP Apps for ChatGPT hit two production bugs: the host strips `_meta` from tool results (breaking state persistence), and widgets don't survive a page refresh. Their conclusion: the platform is "essentially unusable for production apps." Their question: keep investing and wait for fixes, or move back to `window.openai`?

It's a fair question, and it points to a real tension. The MCP Apps spec leaves implementation details to each host. Claude and VS Code support it. ChatGPT is catching up but inconsistently. The same patterns that work in one client break in another.

This is the browser wars problem, playing out in agentic UI.

Meanwhile, [Alpic](https://alpic.ai), the team behind [Skybridge](https://github.com/alpic-ai/skybridge) (the main open-source framework for building MCP Apps), hit a related issue: AI coding agents like Cursor and Claude Code can't yet reliably generate Skybridge code, because the framework's conventions are too new to be in their training data. They opened an issue to address it: [Add coding agent harness to devtools](https://github.com/alpic-ai/skybridge/issues/587).

The bootstrapping problem, in concrete form: the tools being built *for* the agentic world aren't yet fully usable *by* the agentic tools themselves.

So: should you invest? The honest answer is yes, with eyes open. The spec is live, cross-vendor adoption is real, and the friction is known and being worked on. But if you're shipping to production users today, test across clients and don't assume the spec and the implementation match.

---

## What MCP actually is, and where it's headed

The [Model Context Protocol](https://modelcontextprotocol.io) is an open standard that lets AI agents connect to external tools, data sources, and services in a structured, vendor-neutral way. Think of it as USB-C for AI integrations: instead of every agent needing a custom connector for every tool, MCP defines one protocol that works across Claude, ChatGPT, VS Code, Cursor, and more.

Introduced by Anthropic in late 2024 and donated to the [Agentic AI Foundation](https://www.anthropic.com/news/donating-the-model-context-protocol-and-establishing-of-the-agentic-ai-foundation) (under the Linux Foundation) in December 2025, co-founded by Anthropic, OpenAI, and Block, with support from Google, Microsoft, AWS, Cloudflare, and Bloomberg.

**The three primitives:**
- **Tools:** callable functions that let agents take actions (query a DB, create a PR, send a message)
- **Resources:** readable data that gives agents context upfront (documentation, datasets, model lists)
- **Prompts:** reusable templates that guide agent behaviour within a specific product

**MCP Apps** is the first official MCP extension, launched January 2026. Tools can now return interactive React components that render directly inside the conversation. A flight search that returns a live, filterable widget. A deployment tool that surfaces a form with conditional fields. A contract review tool where you approve or flag clauses inline. The agent stays in the loop; the user gets an interface.

### The 2026 roadmap

The [MCP roadmap for 2026](https://modelcontextprotocol.io/development/roadmap) organises priorities into four areas:

| Priority | What it means |
|---|---|
| **Transport scalability** | Streamable HTTP at scale, stateless operation, load balancer behaviour, MCP Server Cards |
| **Agent-to-agent communication** | How agents delegate tasks to other agents |
| **Governance maturation** | Contributor ladder, Working Group delegation, community-led spec evolution |
| **Enterprise readiness** | Audit trails, SSO-integrated auth, gateway patterns, config portability |

The four talks at MCP Connect Amsterdam mapped almost perfectly onto these four areas, which is part of what made the evening feel coherent rather than scattered.

---

## The talks, mapped to the roadmap

### Transport scalability + agent communication - Sam Morrow (GitHub)
**Talk: MCP vs CLIs: Why Agents Need Purpose-Built Interfaces**

Sam's talk was the clearest articulation of where the protocol sits technically. His core argument: MCP vs CLI is the wrong question. The real question is what you're optimising for.

He framed the whole problem around five constraints he calls **The Five Context Engineering Constraints**: every token spent on tool calling competes across these forces, and optimising one often worsens another.

1. **Roundtrips** - each turn re-sends everything before it
2. **Cache Stability** - changing tools costs cache hits
3. **Context Budget** - schemas, results, and history all compete
4. **Trajectory Success** - failed runs waste ALL their tokens
5. **Context Rot** - quality degrades as context fills up

Most "MCP is dead" takes reduce to concerns about constraints 3 and 5, and those are real. But progressive discovery (Anthropic's Tool Search, now on by default; OpenAI's equivalent launched last week) means you no longer need to preload everything. The constraints interact with each other though: solving the context budget problem can worsen cache stability. There's no free optimisation.

The point that landed hardest: **MCP is the only interface that distinguishes between the user and the agent as a protocol-level contract.** CLIs don't do this. Tool annotations, elicitation flows, and MCP Apps all exist specifically to let humans stay in the loop in ways that are structurally enforced by the spec, not just informally hoped for.

Sam also mentioned he deliberately omitted the SSH public key API from the GitHub MCP Server: agents should never be able to set that. A small detail that says a lot about how he thinks about the protocol.

**Repo:** [github/github-mcp-server](https://github.com/github/github-mcp-server)

---

### Governance maturation - Anthony Diaz (Orq.ai)
**Talk: Improving Agentic Experience and Onboarding Using MCPs**

Anthony's talk was about a specific, underappreciated problem: agents feel fragile because they lack context about the products they're trying to use. His answer connects directly to governance: use all three MCP primitives together, not just tools, and invest heavily in the quality of what you expose.

What the agent gets from a well-built MCP: a communication layer with your product, a clear overview of your entities, and always access to the latest capabilities (unlike CLIs, which require reinstallation to update).

The onboarding comparison:

**Without MCP:** Read documentation (30 min) + Find API keys (20 min) + Configure integrations (45 min) + Debug connection issues (60 min) = first result after 3+ hours.

**With MCP:** Click "Connect Tools" (10 sec) + Agent discovers capabilities (5 sec) + Agent auto-configures (5 sec) + Personalised results delivered (30 sec) = value delivered in under 1 minute.

That's not a rounding error. That's a different category of experience.

Anthony's key takeaway: MCP is a strategic enabler for agentic products, and time-to-value is the ultimate onboarding metric. Agentic onboarding should be self-driving. MCP makes agents configure themselves from context. Act now for first-mover advantage.

Orq.ai's own MCP has ~25 tools, all optimised for context window efficiency. Pre-computed resources mean agents don't need database round trips for basic discovery.

**Orq.ai:** [orq.ai](https://orq.ai)

---

### Transport scalability (client side) - Sean Kenny (Prosus)
**Talk: Using or Not Using MCPs as a Client**

Sean's talk was the most practically grounded of the evening. Prosus runs an internal agentic platform with 4,000-5,000 active agents per day, many used by non-technical staff.

When you build agents and scale in production, you want to maximise reliability, maximise accuracy, minimise latency, and minimise cost. That means: **as few tools as possible, tools as tailored as possible.**

But when you build MCPs for others to use, you focus on maximum capability. That means: **as many capabilities as possible (wrap APIs).**

These goals are structurally opposed. No amount of goodwill from MCP providers resolves this. The enterprise client that wants a single tailored tool and the MCP provider that wants to showcase everything are not building for each other.

Sean's BigQuery example: the official BigQuery MCP has tools like `bigquery-analyze-contribution`, `bigquery-conversational-analytics`, `bigquery-execute-sql`, `bigquery-forecast`, and more. For his use case (account managers needing a daily restaurant report), he didn't need any of them. He built one custom tool: give me the restaurant ID, return the data. One tool. No ambiguity. Reliable at scale.

His framework for evaluating MCP sources: community MCP (unusable at enterprise scale, you can't monitor a live third-party project), official MCP (quality varies, structural conflict between provider and builder goals), and build it yourself (increasingly the right call for tailored experiences).

**Prosus AI House Amsterdam:** [prosus.com](https://www.prosus.com)

---

### MCP Apps (transport + governance intersection) - Charles Sonigo (Alpic)
**Talk: Building MCP & ChatGPT Apps with Modern DevX Using Skybridge**

Charles's talk covered MCP Apps in practice: how Skybridge handles the React component wiring, state sync between the UI and the model context, and the double-iframe security architecture the spec requires. The framework is platform-agnostic and works with both ChatGPT Apps SDK and MCP Apps clients.

The post-talk conversation turned to the bootstrapping issue linked above: building for the agentic world using tools that don't yet understand that world's conventions. It's an early-stage problem, and Alpic is actively working on it.

**Skybridge:** [github.com/alpic-ai/skybridge](https://github.com/alpic-ai/skybridge) | **Alpic:** [alpic.ai](https://alpic.ai)

---

## The gap: enterprise readiness

Of the four roadmap priorities, enterprise readiness is described by the maintainers themselves as "the least defined." There is no Enterprise Working Group yet. The roadmap lists the areas: audit trails, SSO-integrated auth, gateway patterns, config portability. But the team has explicitly said they want the people experiencing these challenges to help define the work.

Sean's talk scratched the surface of what enterprise deployment actually looks like at scale. The community MCP trust problem, the official MCP quality problem, the tool overexposure problem: all of these are enterprise readiness issues, and none of them have clean protocol-level answers yet.

I left the evening with more questions than when I arrived, which is usually a sign that the evening was worthwhile.

**One for anyone reading this:** if you were going to push one thing into the Enterprise Working Group charter first, what would it be? Audit trails, SSO-integrated auth, gateway behaviour, or something else entirely? I'd genuinely love to hear from practitioners who are hitting these walls. Open an issue or find me on [LinkedIn](https://www.linkedin.com/in/daniela-florinaungureanu/).

---

## Useful links

| Resource | Link |
|---|---|
| MCP official site | [modelcontextprotocol.io](https://modelcontextprotocol.io) |
| MCP 2026 roadmap | [modelcontextprotocol.io/development/roadmap](https://modelcontextprotocol.io/development/roadmap) |
| MCP Apps announcement | [blog.modelcontextprotocol.io](https://blog.modelcontextprotocol.io/posts/2026-01-26-mcp-apps/) |
| GitHub MCP Server | [github.com/github/github-mcp-server](https://github.com/github/github-mcp-server) |
| Skybridge (Alpic) | [github.com/alpic-ai/skybridge](https://github.com/alpic-ai/skybridge) |
| Skybridge coding agent harness issue | [alpic-ai/skybridge#587](https://github.com/alpic-ai/skybridge/issues/587) |
| MCP Apps broken in ChatGPT discussion | [OpenAI dev forum](https://community.openai.com/t/mcp-apps-in-chatgpt-are-fundamentally-broken-2-critical-bugs/1377697) |
| Orq.ai | [orq.ai](https://orq.ai) |
| Agentic AI Foundation | [Linux Foundation / AAIF](https://www.anthropic.com/news/donating-the-model-context-protocol-and-establishing-of-the-agentic-ai-foundation) |

---

## About me

I'm [Florina](https://www.linkedin.com/in/daniela-florinaungureanu/), an ecosystem operator and venture builder based in Amsterdam. I work at the intersection of developer tools, agentic workflows, and commercial growth. Not an engineer, but deeply curious about how this technology is being built and adopted, and what it means for the organisations trying to use it.

If you were at the event and want to compare notes, or if you have a view on the enterprise readiness question, feel free to open an issue or reach out on LinkedIn.
