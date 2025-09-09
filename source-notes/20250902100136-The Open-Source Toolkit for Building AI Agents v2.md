---
type: source-note
title: The Open-Source Toolkit for Building AI Agents v2
id: 20250902100936
created: 2025-09-02T10:01:36
source:
  - web
url: https://www.aitidbits.ai/p/open-source-agents-updated
tags:
  - source-note
  - ai/agent
  - llm
  - tool
  - open-source
processed: false
archived: false
---
### An opinionated, developer-first guide to building AI agents with real-world impact

***Welcome to a new post in the AI Agents Series - helping AI developers and researchers deploy and make sense of the next step in AI.***

This one is an updated version of one of my recent popular posts, which outlined the most popular and useful open-source libraries for AI agent builders:

  
The landscape of AI agent tools is evolving rapidly. After publishing my previous post, I received suggestions for additional packages I hadn't encountered. Over the past few months, I've tested these tools and documented new, valuable agent-related libraries. This post shares my updated insights.

If you often wonder, "What tools are people actually using to build voice agents or understand documents?"—this post is for you. With new packages emerging almost daily, it can be challenging to determine what's state-of-the-art and truly usable. This list is deliberately selective, focusing on the libraries I've personally found most effective, or those recommended by colleagues I trust.

In this post, I'll provide a curated and updated overview of the open-source ecosystem for developers building AI agents. While there’s no shortage of AI agent market maps, most are geared toward non-builders who need actionable tools and frameworks to launch functional AI agents today.

Every package listed in this post allows commercial use and has a permissive open-source license.

Categories covered in this piece:  
→ Building and Orchestrating Agents (10)  
→ Computer Use (5)  
→ Browser Automation (5)  
→ Voice (12)  
→ Document Processing (7)  
→ Memory (3)  
→ Testing, Evaluation, and Observability (6)  
→ Vertical Agents (7)  
  
Plus:

- Real-world agent stacks: Voice agent that answers phone calls + Browser agent that crawls LinkedIn URLs
- Curated guides and tutorials to get started building agents

![](https://substackcdn.com/image/fetch/$s_!91rM!,w_424,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F190a5f5b-203f-447d-8ad6-a07d9ab3c874_2600x1456.png)

---

```markup
Become a premium AI Tidbits subscriber and get over $1k in free credits to build AI agents with Vapi, Claude, and other leading AI tools (Hugging Face, Deepgram, etc.), along with exclusive access to the LLM Builders series and in-depth explorations of crucial topics, such as the future of the internet in an era driven by AI agents.

Many readers expense the paid membership from their learning and development education stipend.
```

---

## Building and Orchestrating Agents

To build agents that go beyond simple prompting, you need infrastructure for planning, memory, and tool use, and a way to hold it all together.

As more developers started shipping real-world agents, new frameworks popped up and older ones evolved to meet the actual challenges of agentic AI. This section covers the tools I’ve found most effective for building agents that can think, remember, and act with minimal hand-holding.

<iframe src="https://datawrapper.dwcdn.net/oYI5j/13/" width="730" height="400" frameborder="0"></iframe>

**  
How to choose?**

- For beginners or rapid prototyping, consider **Langflow** for its intuitive visual interface and **OpenAI’s Agents SDK**, or **LangChain** for their simplicity and flexibility.
- For enterprise applications, **Portia** and **CrewAI** offer robust features suitable for production environments requiring control and scalability.
- For multimodal or memory-intensive agents, **Agno** provides lightweight support for agents needing persistent memory and multimodal inputs.
- For complex simulations or data generation, **Camel** excels in creating customizable multi-agent systems for simulating real-world interactions.
- For autonomous task execution, **AutoGPT** is designed for agents that need to operate without continuous human input.

![Welcome to Langflow | Langflow Documentation](https://substackcdn.com/image/fetch/$s_!gpJj!,w_424,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fc904ca6c-ab30-454e-b358-c45f8cf2a1e3_5760x3240.png)

Langflow simplifies the creation of agents and workflows that integrate with any API, model, or database

---

## Computer Use

AI agents become far more useful when they can operate computers like humans: clicking, typing, browsing, and running programs. The libraries below make that possible, letting agents bridge the gap between language output and real-world action.

<iframe src="https://datawrapper.dwcdn.net/jKBVM/5/" width="730" height="400" frameborder="0"></iframe>

**  
How to choose?**

- For local code execution via natural language, go with **Open Interpreter** – it’s fast to set up and great for command-driven agents.
- For agents that need to see and control a computer screen like a human, **Self-Operating Computer** is your best bet.
- If your agent needs to run in a secure, fast, sandboxed environment, use **CUA**.
- For dynamic multi-step tasks on irregular interfaces, **Agent-S** offers the most flexibility with its planning and learning capabilities.
- If your agent relies on interpreting UIs from screenshots (e.g., grounding actions in visual layouts), **OmniParser** adds critical visual parsing capabilities.

![temp.mov [optimize output image]](https://substackcdn.com/image/fetch/$s_!1c1O!,w_424,c_limit,f_webp,q_auto:good,fl_lossy/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F8331c21f-e1cd-41fb-8db0-8409afc47ac5_800x502.gif)

Using CUA to edit a photo in Photoshop. All actions in this video are automated from a given natural language prompt.

---

## Browser Automation

As AI agents move from passive reasoning to active execution, the browser becomes their gateway to the internet. Whether scraping data, submitting forms, or navigating complex workflows, browser automation tools let agents interact with web pages just like a human would—with clicks, scrolls, and typed input. These libraries differ in abstraction level, performance, and agent integration, so choosing the right one depends on your goals.

<iframe src="https://datawrapper.dwcdn.net/1CVsC/11/" width="730" height="400" frameborder="0"></iframe>

  
**How to choose?**

- For a low-code, declarative approach where the LLM plans the steps, try **Stagehand**.
- If you're building agents that need to deeply understand and extract content from websites, **Firecrawl** offers the cleanest pipeline.
- For LLM-friendly control over browser actions with integration hooks, I’d recommend the popular **browser-use**.
- Choose **Playwright** if you need more low-level control over browser actions across browsers.
- Use **Puppeteer** if you need fast, scriptable Chrome automation in a Node.js environment.

![442961777-a0ffd23d-9a11-4368-8893-b092703abc14.gif [optimize output image]](https://substackcdn.com/image/fetch/$s_!dX7B!,w_424,c_limit,f_webp,q_auto:good,fl_lossy/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F8ef20a49-850a-47e7-b498-7ff6e7feffb7_1809x851.gif)

Adding grocery items to a cart and checking out using browser-use

---

## Voice

Voice is still the most intuitive interface for humans, and increasingly, for agents too. These tools let agents handle speech in and out: understanding spoken language, keeping track of conversations, and responding naturally.

<iframe src="https://datawrapper.dwcdn.net/dZHX8/7/" width="730" height="400" frameborder="0"></iframe>

**  
How to choose?**

I wrote a whole post covering open and commercial packages and APIs for voice agents, including my guide for choosing the right ones:

<video src="blob:https://www.aitidbits.ai/dd1bd143-8ccd-4efc-97b5-63fb88592f67"></video>

Dia [compared](https://yummy-fir-7a4.notion.site/dia) to ElevenLabs and Sesame 👆

---

## Document Processing

Modern AI agents must process and comprehend documents in various formats, from PDFs to images containing text. The following open-source tools empower agents to extract, interpret, and act upon information from unstructured documents, facilitating real-world business processes.

<iframe src="https://datawrapper.dwcdn.net/LW5mf/4/" width="730" height="400" frameborder="0"></iframe>

**  
How to choose?**

- Long-form PDFs such as contracts, research papers - use **Qwen2.5-VL** or **mPLUG-DocOwl2** for efficient multi-page understanding without relying on OCR. And, as of a few months ago, you can also easily fine-tune a DocOwl2 model on your own data with [ms-swift](https://github.com/modelscope/ms-swift).
- Text + image docs such as medical reports, annotated diagrams - try **Molmo** for high-resolution multimodal inputs, visual QA, and GUI parsing.
- Layout analysis & table extraction - use **Docling** for JSON/Markdown conversion, or **LayoutLMv3** for form understanding and layout-aware modeling.
- Lightweight multimodal with speech - **Phi-4** handles text, vision, and speech in a compact model—great for on-device agents.
<video src="blob:https://www.aitidbits.ai/f477049c-f395-4a52-a011-1521952f091c"></video>

---

## Memory

To feel truly intelligent, AI agents need memory. Without it, they’re stuck in single-turn loops, forgetting what just happened, what the user wants, or what they already did. The libraries below help agents remember, adapt, and personalize, enabling everything from contextual conversations to long-term planning.

<iframe src="https://datawrapper.dwcdn.net/Lodm4/4/" width="730" height="400" frameborder="0"></iframe>

**  
How to choose?**

- For long-term, personalized memory that improves with use, **Mem0** is a solid choice—especially if you want minimal overhead.
- Use **Letta** when building agents that need persistent memory across sessions and integration with tools or APIs.
- To enable active memory management and knowledge sharing among agents, **LangMem** facilitates dynamic memory operations and shared knowledge bases.

![temp.mov [optimize output image]](https://substackcdn.com/image/fetch/$s_!XwTg!,w_424,c_limit,f_webp,q_auto:good,fl_lossy/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F736173d0-250a-453b-bdec-0c295b18c2c1_800x448.gif)

Creating stateful agents with Letta

---

```markup
Become a premium AI Tidbits subscriber and get over $1k in free credits to build AI agents with Vapi, Claude, and other leading AI tools (Hugging Face, Deepgram, etc.), along with exclusive access to the LLM Builders series and in-depth explorations of crucial topics, such as the future of the internet in an era driven by AI agents.

Many readers expense the paid membership from their learning and development education stipend.
```

---

## Testing, Evaluation, and Observability

As agents grow more complex, they need to be tested, measured, and monitored like any serious software system. These tools help you catch edge cases, debug behavior, and track performance, both during development and in production.

<iframe src="https://datawrapper.dwcdn.net/Zyh1j/2/" width="730" height="400" frameborder="0"></iframe>

**  
How to choose?**

- To monitor and benchmark agent performance in production environments, **AgentOps** provides robust tracking and analysis tools.
- When comparing various agent configurations or conducting A/B tests, **Agenta** facilitates structured evaluations.
- To integrate observability into LLM applications, **OpenLLMetry** leverages OpenTelemetry for seamless monitoring.
- If detecting and addressing performance, bias, or security issues is a priority, **Giskard** offers automated scanning capabilities.
- For comprehensive LLM observability and debugging, **Langfuse** provides an open-source platform tailored for LLM applications.
- For voice agent evaluation across different models and prompts, **VoiceLab** offers a comprehensive testing framework.

![Demo usage](https://substackcdn.com/image/fetch/$s_!VEsF!,w_424,c_limit,f_webp,q_auto:good,fl_lossy/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F2156ad29-8bdc-4efa-abb2-209ec74ed81f_800x400.gif)

Test and refine your voice agents with Voice Lab

---

## Vertical Agents

The open-source world is full of vertical agents: purpose-built tools for coding, research, data analysis, and more. I’ve tested a bunch. These are the ones I’d actually reach for when building something real.

<iframe src="https://datawrapper.dwcdn.net/8yunC/1/" width="730" height="400" frameborder="0"></iframe>

**  
How to choose?**

- **Goose** allows custom workflow integration to build extensible AI coding assistants.
- For comprehensive coding agents with GUI capabilities, **OpenHands** offers a full-stack solution inspired by Devin.
- If you prefer a Claude Code-like terminal-based pair programming, **aider** provides Git integration and multi-file editing.
- To convert UI designs from images to code, **screenshot-to-code** automates the prototyping process.
- For autonomous research tasks, **GPT Researcher** can scrape, summarize, and export findings efficiently.
- For conducting in-depth, privacy-focused research using local LLMs, **Local Deep Research** offers iterative analysis and comprehensive, cited reports.
- If your focus is on generating SQL queries from text, **Vanna** offers customizable and database-integrated solutions.
	![430601405-8fcaaa4c-31e5-4814-89b4-94f1433d139d.mp4 [optimize output image]](https://substackcdn.com/image/fetch/$s_!NZzC!,w_424,c_limit,f_webp,q_auto:good,fl_lossy/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fc2ec8571-c1a7-4945-9fb4-aad5cda51ebc_800x450.gif)
	GPT Researcher can conduct research using local and web

---

Lastly, here are real-world stacks and beginner-friendly tutorials to help you launch your first AI agent.

## Real-world agent stacks

All the tools above are powerful on their own, but how do they actually fit together in practice? What does a real architecture look like when you're stitching these components into something usable, testable, and shippable? I’ve compiled a few concrete examples from recent open-source projects and builders in the space. If you're trying to move from “exploring tools” to “building real systems”, these will give you a head start.