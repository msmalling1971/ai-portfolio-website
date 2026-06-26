# Case Study Framework

## Purpose

MatthewSmalling.com documents engineering decisions made under real-world constraints.

The site should help another infrastructure leader understand how Matt thinks, not just what technologies he has used. Tools, platforms, diagrams, and metrics are evidence. The story is the judgment behind the work.

Core principle:

> Every case study must teach an engineering decision.
>
> Technology is evidence. Judgment is the story.

## Case Study Structure

### 1. Executive Summary

Purpose: Give a senior reader the point of the case study quickly.

What question it answers: What was built, why did it matter, and what should I remember?

Writing guidance: Keep this short. Lead with the operational purpose, not the tool list. Mention the most important decision or result.

Example phrasing: "NovaLab is not an AI demo. It is a controlled runtime platform for understanding how model serving, GPU infrastructure, observability, benchmarking, and recovery fit together."

### 2. Business Context

Purpose: Explain why the project mattered beyond personal learning or technical interest.

What question it answers: What operational or business risk made this worth doing?

Writing guidance: Connect the work to ownership, reliability, cost, risk, recovery, security, governance, or service quality.

Example phrasing: "Unmanaged AI experimentation creates risk when ownership, observability, recovery, cost, and privacy expectations are unclear."

### 3. Real-World Relevance

Purpose: Show how the lab or project maps to situations an organization would actually face.

What question it answers: Why should a CTO, CIO, or infrastructure leader care?

Writing guidance: Avoid pretending a lab is the same as production. Instead, explain which real operating patterns it helps evaluate.

Example phrasing: "The lab does not claim to be production. It creates a controlled place to understand the decisions production would require."

### 4. Problem Statement

Purpose: Define the problem clearly before explaining the solution.

What question it answers: What was wrong, missing, unclear, risky, or unknown?

Writing guidance: State the problem in practical terms. Avoid vague transformation language.

Example phrasing: "The environment could run a model, but it could not yet explain runtime health, load behavior, GPU saturation, or recovery state."

### 5. Constraints

Purpose: Show the boundaries that shaped the work.

What question it answers: What limits had to be respected?

Writing guidance: Treat constraints as evidence of judgment. Include budget, hardware, licensing, time, risk, existing infrastructure, and skills being developed.

Example phrasing: "The platform had to fit within a single RTX 3090 24GB, avoid unnecessary cloud spend, and remain recoverable during experimentation."

### 6. Options Considered

Purpose: Show that the final approach was chosen, not stumbled into.

What question it answers: What alternatives were available, and why were they accepted or rejected?

Writing guidance: Keep the comparison honest and concise. Do not make rejected options look foolish. Explain fit, tradeoffs, risk, and timing.

Example phrasing: "Ollama was useful for fast experimentation, but vLLM better supported the OpenAI-compatible API, metrics, and load-testing path needed for this case study."

### 7. Engineering Decision

Purpose: Make the central judgment explicit.

What question it answers: What decision did Matt make, and why?

Writing guidance: Use a simple decision/reason/outcome pattern. This is often the most important section.

Example phrasing: "Decision: rebuild from a clean baseline. Reason: the environment had accumulated drift and troubleshooting noise. Outcome: produced a known-good platform state that could be validated, backed up, and extended."

### 8. Architecture

Purpose: Explain the system shape.

What question it answers: How do the parts fit together?

Writing guidance: Use diagrams where possible. Keep prose focused on responsibilities, boundaries, dependencies, and flows.

Example phrasing: "The architecture separates the user interface, runtime API, GPU execution layer, observability stack, load testing path, and recovery boundary."

### 9. Implementation

Purpose: Explain what was built without turning the page into a runbook.

What question it answers: What changed in the environment?

Writing guidance: Summarize implementation decisions and major components. Save command-level detail for reference notes or appendices.

Example phrasing: "The runtime was moved behind an OpenAI-compatible API so the same path could support WebUI access, validation, and future service patterns."

### 10. Validation

Purpose: Show how the work was tested.

What question it answers: How did Matt know it worked?

Writing guidance: Explain the validation method, test path, expected behavior, and what was observed. Do not invent numbers.

Example phrasing: "Locust generated load against the same API path real clients would use, while Grafana showed runtime, latency, queueing, and GPU behavior."

### 11. Evidence

Purpose: Support claims with proof.

What question it answers: What can the reader inspect?

Writing guidance: Evidence should support claims, not decorate the page. Use screenshots, diagrams, configuration excerpts, logs, or commit history only when they clarify the story.

Example phrasing: "Evidence includes Grafana dashboards, Locust results, Prometheus targets, and GPU telemetry captured during load."

### 12. Operational Results

Purpose: Explain what changed after the decision or implementation.

What question it answers: What became more reliable, visible, recoverable, understandable, or supportable?

Writing guidance: Be specific and restrained. If the result is a validated baseline, say that. Do not inflate it into enterprise impact.

Example phrasing: "The platform moved from a working demo to a measurable baseline that could be tested, backed up, and extended."

### 13. Lessons Learned

Purpose: Extract reusable judgment from the work.

What question it answers: What would another engineer or leader learn from this?

Writing guidance: Write lessons as principles, not diary entries. Keep them direct.

Example phrasing: "VRAM is a budget. Model choice, context, format, and quantization all spend from the same limited pool."

### 14. What I'd Do Differently

Purpose: Show reflection and maturity.

What question it answers: What changed Matt's thinking, and what would improve the next pass?

Writing guidance: Be honest without over-apologizing. Focus on sequencing, validation, documentation, recovery, and decisions that became clearer later.

Example phrasing: "I would establish the observability baseline earlier so runtime changes could be evaluated against known signals from the start."

### 15. Future Improvements

Purpose: Define the next logical work without pretending it is already complete.

What question it answers: What should be built, tested, documented, or improved next?

Writing guidance: Keep this concrete. Separate planned work from validated work.

Example phrasing: "Future improvements include architecture diagrams, screenshot walkthroughs, recovery testing, model comparison, and a decision matrix for runtime selection."

### 16. Matt's Notes

Purpose: Add the human layer behind the technical work.

What question it answers: What did Matt notice, misunderstand, fix, or want to explore next?

Writing guidance: Keep this reflective and honest. It should sound like Matt, not polished marketing copy.

Example phrasing: "The biggest shift was realizing that a model running successfully is not the same as a platform being ready to operate."

### 17. Executive Takeaway

Purpose: Leave the reader with the leadership point.

What question it answers: What does this case study prove about Matt's judgment?

Writing guidance: Keep it short. Connect the technical work back to operational leadership.

Example phrasing: "This case study shows a practical approach to learning AI infrastructure: build the baseline, measure the system, understand the limits, and protect the working state before expanding."

## Required Thinking

Every case study should answer:

- What problem existed?
- Why did this project matter?
- What constraints shaped the solution?
- What options were considered?
- Why was this approach chosen?
- What tradeoffs were accepted?
- How was it validated?
- What evidence proves it?
- What changed operationally?
- What was learned?
- What would be different next time?

## Writing Rules

- Keep the tone calm, practical, and direct.
- Avoid marketing language.
- Avoid exaggerated claims.
- Avoid buzzwords.
- Do not invent metrics.
- Do not overstate business impact.
- Explain tradeoffs honestly.
- Let evidence do the heavy lifting.
- Keep technology names secondary to the engineering decision.
- Write for another infrastructure leader, CTO, or senior technical reviewer.

## Case Study Standards

Every case study should include:

- A clear business or operational reason.
- Explicit constraints.
- At least one engineering decision.
- At least one tradeoff.
- At least one lesson learned.
- Evidence where available.
- A short executive takeaway.

## Evidence Standards

Acceptable evidence examples include:

- Architecture diagrams
- Grafana dashboards
- Prometheus targets
- Locust results
- Terminal output
- API validation screenshots
- Configuration snippets
- Git commit history
- Photos of lab hardware
- Network diagrams
- Before/after comparisons

Evidence should support claims, not decorate the page.

## Constraints Section Guidance

Constraints are not weaknesses.

Constraints demonstrate judgment because they show the real boundaries of the work. A case study without constraints can read like a technology list. A case study with clear constraints shows decision-making.

Useful constraint categories include:

- Budget
- Hardware availability
- Licensing
- Time
- Skills being developed
- Risk tolerance
- Existing infrastructure
- Operational complexity

## Options Considered Guidance

The Options Considered section shows leadership thinking.

It should explain why an option was selected or rejected. The goal is not to make every alternative sound bad. The goal is to show fit, timing, operational value, and tradeoffs.

Examples:

NovaLab:

- Ollama
- llama.cpp
- vLLM

Raspberry Pi Network TAP:

- Enterprise TAP
- SPAN port
- Passive TAP
- Raspberry Pi bridge

Azure Enterprise:

- Zero Trust first
- Identity baseline first
- Collaboration baseline first

## Matt's Notes Guidance

Matt's Notes should remain reflective and honest.

They should capture:

- What surprised me
- What broke
- What I misunderstood
- What changed my thinking
- What I would do differently
- What I want to explore next

Matt's Notes should sound human, not polished marketing copy.

## Things To Avoid

Avoid:

- "World-class"
- "Cutting-edge"
- "Game-changing"
- "Best-in-class"
- "Revolutionary"
- Vague transformation language
- Fake ROI
- Unsupported metrics
- Overly polished corporate tone
- Writing that sounds like a vendor brochure

## Example Mini Template

```markdown
# Case Study Title

## Executive Summary

What was built, why it mattered, and the main engineering decision.

## Business Context

What operational or business problem made this worth doing.

## Real-World Relevance

How this maps to real infrastructure, leadership, or operating concerns.

## Problem Statement

The specific problem, gap, risk, or unknown.

## Constraints

- Constraint 1
- Constraint 2
- Constraint 3

## Options Considered

| Option | Why It Was Considered | Decision |
| --- | --- | --- |
| Option A | Reason | Selected / rejected and why |
| Option B | Reason | Selected / rejected and why |

## Engineering Decision

Decision:

Reason:

Tradeoff:

Outcome:

## Architecture

High-level diagram or description of the system shape, boundaries, and flows.

## Implementation

What was built or changed. Keep command-level details out unless they are evidence.

## Validation

How the work was tested and what was observed.

## Evidence

- Diagram
- Screenshot
- Dashboard
- Configuration excerpt
- Commit history

## Operational Results

What became more reliable, visible, recoverable, supportable, or understandable.

## Lessons Learned

- Lesson 1
- Lesson 2
- Lesson 3

## What I'd Do Differently

What should change in the next pass.

## Future Improvements

- Improvement 1
- Improvement 2
- Improvement 3

## Matt's Notes

Reflective notes about what surprised me, what broke, what I misunderstood, and what I want to explore next.

## Executive Takeaway

The short leadership takeaway from the case study.
```
