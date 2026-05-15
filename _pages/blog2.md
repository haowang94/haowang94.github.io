---
permalink: /blog/specbench/
title: "The Next Bottleneck for Coding Agents Isn't Code. It's Knowing What to Ask For."
author: Hao Wang
author_profile: true
layout: single
---

<p class="blog-date">May 2026 · 4 min read</p>

For decades, software engineering had a high barrier to entry. Building production software required years of training across programming languages, frameworks, databases, architecture, testing, and deployment.

Then AI coding tools arrived.

Coding agents made it possible to generate large amounts of working code from natural language. For a moment, it seemed like the barrier had collapsed. Maybe anyone could describe what they wanted and get finished software.

But early evidence suggests otherwise.

[Fastly's July 2025 survey](https://www.fastly.com/blog/senior-developers-ship-more-ai-code?utm_source=chatgpt.com) of 791 professional developers found that senior developers get more value from AI coding tools than junior developers. 32% of senior developers said more than half of their shipped code is AI-generated, compared with only 13% of junior developers. Seniors were also more likely to say AI helps them ship faster: 59% versus 49%. And 26% of seniors said AI makes them "a lot faster," compared with 13% of juniors.

This points to an uncomfortable truth: AI coding tools do not automatically make everyone the same level of engineer. They amplify differences in a user's ability to specify, evaluate, and steer.

<img src="/images/intent_to_spec.png" alt="From intent to specification" style="max-width: 600px; width: 100%;">

---

## AI Has Lowered the Cost of Implementation, but Not the Cost of Specification

Traditional software development has a well-established pipeline.

A client starts with a business need: "We need an AI toolkit for auditing bias in ML models." A product manager then works with the client to clarify goals, constraints, success metrics, and requirements. Engineers review the spec, assess feasibility, identify risks, and propose an implementation plan.

This process is slow, but it exists because software is not just code. Software is a set of decisions.

AI coding agents shorten the implementation phase. They can create files, write functions, fix bugs, run tests, and even deploy applications. But they still need direction.

Experienced engineers are not just better at coding. They are better at shaping ambiguity. They know which questions matter before implementation begins. They can anticipate conflicting requirements, reason through trade-offs, define success metrics, and distinguish core features from nice-to-haves.

Non-technical users often have not thought through these issues. When they give an underspecified request, the agent fills in the gaps to produce something that works, but the result may not reflect what the user actually wanted.

---

## The Missing Layer: Spec Collaboration

To make AI coding tools useful to more people, we need better spec collaborators.

Before an agent writes code, it should help the user discover what they actually want to build. It should ask clarification questions, identify conflicts, surface assumptions, and turn a vague request into a comprehensive spec sheet.

Today, many agents behave like eager programmers. The user says, "Build me an app," and the agent starts building.

But good engineers do not immediately code. They ask questions:

- What is the core workflow?
- What are the privacy, security, and reliability constraints?
- What should happen when something fails?
- How will we know the product is successful?

How well do existing coding agents help users turn vague ideas into executable specifications?

---

## Benchmarking the Spec-Writing Ability of Coding Agents

<img src="/images/specbench.jpg" alt="Pipeline of SpecBench" style="max-width: 800px; width: 100%;">
<p><em>Figure 1. Pipeline of SpecBench.</em></p>

We introduce **SpecBench**, a benchmark for evaluating how well AI agents collaborate with users during software specification design. Each task provides a high-level software request (the user's intent), the user's prior conversation history, and a limited budget for asking clarifying questions.

SpecBench evaluates agents along two dimensions:

- **Preference elicitation.** The agent predicts the user's answers to a set of design-choice questions. These questions capture ambiguities that are not resolved by the initial request and often have no single ground-truth answer. This measures whether the agent truly understands the user's preferences and priorities.

- **Specification drafting.** The agent produces a structured spec sheet containing sections such as project overview, core features, constraints, and success metrics. The resulting spec is evaluated on coverage, precision, consistency, insightfulness, and readability.

<img src="/images/exp_fig1.png" alt="Preference elicitation results across agents" style="max-width: 800px; width: 100%;">
<p><em>Figure 2. We evaluate Claude Code, Gemini CLI, Cursor CLI, and Buddy on how well they align with users' preferences when making software design decisions.</em></p>

Our results show that existing agents are increasingly capable as solo developers, but still have significant room to improve as collaborative peers.

For example, Gemini CLI gains surprisingly little from user interaction. Even when explicitly instructed to continue asking questions until it is confident about all unresolved design choices, it stops querying the user early. More concerningly, its performance can degrade with more rounds of interaction. This suggests that the agent is better at extracting information from static conversation history than at strategically querying the user.

Other agents, such as Claude Code and Cursor CLI (with GPT-5.4-mini), typically use the full question budget and continue asking whenever ambiguity remains. However, more questions do not necessarily lead to better specifications. This indicates that these agents lack an effective strategy for prioritizing which ambiguities matter most.

---

## Buddy: A User-Assistant Agent for Spec Sheet Writing

<img src="/images/buddy.jpg" alt="Buddy agent workflow" style="max-width: 800px; width: 100%;">
<p><em>Figure 3. Buddy is an agent that works with the user to create a structured spec sheet based on their needs.</em></p>

We introduce **Buddy**, a user-assistant agent designed specifically for collaborative specification writing.

Buddy is built around two key ideas:

1. Identifying the design dimensions that matter most for implementing the user's requested software.
2. Minimizing user interaction while still producing a specification that aligns with the user's preferences.

Buddy's workflow is inspired by classical morphological analysis, which decomposes a problem into independent dimensions and explores alternative possibilities for each. Concretely, Buddy:

- First constructs a **morphological chart** that captures candidate design alternatives corresponding to different user preferences.
- Then creates **simulated users** to infer and evaluate decisions that can already be predicted from the user's prior context.
- For unresolved decisions, Buddy **strategically plans** which questions to ask the real user.
- Finally, it incorporates the user's responses to refine, review, and complete the final specification document.

<img src="/images/exp_fig2.png" alt="Spec sheet quality comparison" style="max-width: 800px; width: 100%;">
<p><em>Figure 4. We compare Claude Code, Gemini CLI, Cursor CLI, and Buddy in terms of spec sheet quality across different evaluation axes (left), the number of times they query the user (middle), and head-to-head win rates (right).</em></p>

Buddy queries the user significantly fewer times than other agents, while still achieving the highest evaluation scores and outperforming all baselines in head-to-head comparisons. This is because Buddy first uses simulated users to resolve many design decisions, leaving only the most uncertain questions for the actual user.

---

## What's Next?

[Anthropic’s recent post](https://www.anthropic.com/engineering/harness-design-long-running-apps) shows that better coding agents require more than better models: they need a harness around them. In their setup, a planner expands a short prompt into a product spec, the generator and evaluator agree on what “done” means before coding begins, and the evaluator tests the running app against concrete criteria.

We argue that a critical part of this harness is still missing: a layer that collaborates with users before and during software development. SpecBench and Buddy are small steps toward filling this gap, but much needs to be done. This layer should maintain evolving memory to better understand each user, help users articulate their ideas, teach them how to use coding agents and new features more effectively, and ultimately improve over time while helping users improve as well.

---

If you're interested in learning more, please check out our 📄 [paper](https://haowang94.github.io/files/specbench.pdf), 💻 [code](https://github.com/haowang94/intent2spec), and 🤗 [data](https://huggingface.co/datasets/haowang94/specbench).