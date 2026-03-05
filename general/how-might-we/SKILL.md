---
name: how-might-we
description: Facilitate "How Might We?" brainstorming sessions for software engineering challenges. Reframes problems into HMW questions from multiple angles, then brainstorms solutions for each. Use when the user wants to explore a problem space, brainstorm approaches to a technical challenge, or run a structured ideation session.
---

# How Might We?

Facilitate a structured brainstorming session using the "How Might We?" (HMW) technique. The session turns a software engineering problem into actionable HMW questions, brainstorms solutions for each, and produces a summary document.

## Session Flow

### Phase 1: Problem Framing

Ask the user to describe their challenge. Clarify until you can state the problem in one or two sentences. Confirm the problem statement before continuing.

Good problem statements are specific and bounded:
- "Users abandon the checkout flow because the page takes 8 seconds to load on mobile."
- "Our test suite takes 45 minutes and developers skip it locally."
- "Three teams are duplicating auth logic because there's no shared library."

If the user's description is vague, ask targeted questions:
- Who is affected?
- What is the current impact?
- What constraints exist (timeline, tech stack, team size)?

### Phase 2: Generate HMW Questions

Reframe the problem into 5–8 HMW questions across these angles:

- **User/Developer experience** — pain points from the person's perspective
  _e.g. HMW make the deploy process feel instant to developers?_
- **Technical** — architecture, performance, reliability
  _e.g. HMW decouple the notification service so a failure doesn't cascade?_
- **Process** — workflows, team practices, communication
  _e.g. HMW surface flaky tests before they block the release train?_
- **Scope/Simplification** — reducing the problem to its essence
  _e.g. HMW solve this for just the top 3 API consumers first?_
- **Inversion** — flipping the problem on its head
  _e.g. HMW make it impossible to deploy a broken migration?_

Guidelines for good HMW questions:
- Broad enough to invite multiple solutions, narrow enough to be actionable.
- Avoid embedding a solution in the question ("HMW use Redis to cache..." is too prescriptive).
- Each question should open a distinct solution space.

Present all questions at once. Ask the user to pick 2–4 they want to brainstorm on (or accept all).

### Phase 3: Brainstorm Solutions

For each selected HMW question:

1. **Generate 3–5 ideas** ranging from quick wins to ambitious bets.
2. **Tag each idea** with effort and impact:
   - Effort: `low` | `medium` | `high`
   - Impact: `low` | `medium` | `high`
3. **Invite the user** to add, modify, or remove ideas before moving to the next question.

Keep ideas concrete and technical. Instead of "improve caching," say "add a read-through Redis cache in front of the user-profile query with a 5-minute TTL."

### Phase 4: Prioritize

After brainstorming all selected questions, present a consolidated view and ask the user to pick their top 3–5 ideas to pursue. Use the effort/impact tags to guide the conversation — high-impact/low-effort ideas are natural starting points.

### Phase 5: Summary Document

Produce a structured markdown document the user can save or share:

```markdown
# HMW Session: <short title>

**Date:** <today's date>
**Problem:** <1–2 sentence problem statement>

## HMW Questions

1. How might we ...?
2. How might we ...?
...

## Brainstorm Results

### HMW: <question 1>

1. **<idea>** — effort: low, impact: high
2. **<idea>** — effort: medium, impact: medium

### HMW: <question 2>
...

## Selected Next Steps

1. **<idea>** (from: <HMW question>) — effort: low, impact: high
2. **<idea>** (from: <HMW question>) — effort: medium, impact: high
```

## Tips for Effective Facilitation

- **Stay neutral.** Present ideas without advocating for a particular solution.
- **Push for specificity.** "Make it faster" → "Reduce p95 latency of the search endpoint from 800ms to 200ms."
- **Name tensions.** If two ideas conflict, surface the tradeoff explicitly rather than silently dropping one.
- **Time-box yourself.** Don't generate more than 5 ideas per question — depth beats breadth.
