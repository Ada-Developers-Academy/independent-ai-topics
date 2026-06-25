# Best Practices A

### Defining a Custom Agent

A custom agent is a saved configuration that bundles a model, a set of tool permissions, and a system prompt into a reusable profile. Rather than reconstructing that setup manually at the start of every session, we define it once and activate it by name whenever the same kind of work comes up.

The most common reason to define a custom agent is that a recurring workflow requires a specific combination of capabilities that doesn't match the general-purpose default. A planning agent that should only read code but never modify it has fundamentally different tool access requirements than an implementation agent that needs full write access. Switching between those manually is error-prone and tedious. A named custom agent encodes the configuration so that switching is a single action.

**What a custom agent file contains**

Custom agents are typically defined as markdown files with a YAML frontmatter section at the top. The file location varies by tool but is usually a project-level folder like `.github/agents/`, `.claude/agents/`, or a user-level directory for agents that should be available across all projects.

The frontmatter defines the agent's configuration:

```markdown
---
name: planner
description: Generates implementation plans for new features. Read-only access only.
tools: ['search/codebase', 'web/fetch']
model: claude-opus-4-6
---

You are in planning mode. Your task is to research the relevant parts of the codebase
and produce a concrete implementation plan. Do not make any code edits.

The plan should include:
- A brief summary of what will change and why
- Which files will be modified and how
- The testing strategy
- Any risks or open questions to resolve before implementation begins

Write the completed plan to `docs/plans/<feature-name>.md`.
```

The key fields:

- **name**: How we refer to and activate this agent
- **description**: What the agent is for; some tools surface this in the UI
- **tools**: The specific tools this agent is allowed to use. Restricting tools here is how we enforce phase-appropriate access without relying on the agent to self-limit.
- **model**: The model to use for this agent's sessions. This is where we can pin a higher-capability model for planning work and a lighter one for agents doing routine tasks.
- **body**: The system prompt that defines the agent's behavior and any specific instructions

**When to define a custom agent versus using a skill**

Skills provide procedural instructions for specific tasks, but they don't change the agent's model, tool access, or base behavior. A custom agent does all three.

A useful distinction: if we want the agent to know *how* to do something differently, a skill is appropriate. If we want to change *what the agent is allowed to do* or *which model it uses*, a custom agent is the right tool. In practice, most well-configured custom agents also load relevant skills.

Some scenarios where creating a custom agent is worth the setup:

- A **planning agent** with read-only tools and a higher-capability model, configured to always output a structured spec file
- A **review agent** with read-only access and specific instructions for the team's code review format
- A **documentation agent** scoped to documentation directories only, preventing it from touching implementation files
- A **research agent** with a lighter model that is explicitly allowed to fetch external documentation and summarize it

**Scope: project-level versus user-level**

Most tools support defining agents at two levels. Project-level agents live in the repository and are available to everyone who clones it, making them useful for team-shared workflows. User-level agents live in a personal directory and are available across all projects on our machine, which is useful for general-purpose roles like a personal planning or review agent we want available everywhere.

For most team-facing workflows, project-level agents are preferable. They version-control alongside the code, get updated when the workflow changes, and don't require each team member to configure them manually.

## Starting New Sessions

This concept came up in Lesson 2 in the context of managing context windows. Here, we'll revisit it as a deliberate practice rather than a fallback for when things get too cluttered.

### A Fresh Context Is a Feature

Starting a new session doesn't mean starting from scratch. If we've followed the practice of writing plans, decisions, and summaries to files, we lose almost nothing by closing a session and opening a new one. What we leave behind is the accumulated noise: the exploratory tangents, the discarded approaches, the back-and-forth that produced the current direction but doesn't need to persist once that direction is set.

A fresh context receives the plan, the current state of the codebase, and the specific task. It doesn't bring in the context of the three things we tried that didn't work. That clean signal often produces more consistent, focused outputs than continuing in a cluttered history.

**Scenario:** A team has been in a planning session for several hours, iterating on an implementation plan for a new authentication flow. The session includes several rounds of "what if we approached it this way instead" exchanges, a long tangent about a library they eventually decided not to use, and an early version of the plan that was significantly revised. They now have a final plan written to `docs/auth-implementation-plan.md`.

Rather than passing this implementation plan to the current session with its full history, they open a fresh session, load the steering file and the plan document, and begin implementation. The agent starts with exactly what's needed: the conventions, the plan, and the current codebase state.

### Recognizing the Signals

A few patterns tend to indicate that a fresh session would serve us better than continuing the current one:

- The current session has a lot of context from a phase of work that is now complete (research, exploration, planning) and we're moving into a different phase (implementation, review)
- We've changed direction significantly and the earlier context now represents paths we're not taking
- Output quality has drifted and we've noticed the agent producing less focused results than it did earlier in the session
- We're starting a task that is genuinely independent of what we've been doing, even if it's in the same project

None of these are hard rules. Sometimes it makes sense to continue. But default-continuing a session because starting fresh feels like lost work is worth questioning. If the important outputs are in files, the "lost work" is mostly accumulated noise.

### The End-of-Session Habit

One practice that makes new sessions feel less costly is building a summary step into the end of each session. Before closing a session, we ask the agent to write a brief summary file capturing:

- The decisions made and their rationale
- Which files were modified and why
- Open questions or next steps
- Any constraints or patterns discovered that should inform future work

This summary can be loaded into the next session's startup context, often alongside the steering file and the relevant plan documents. The new session gets the essential continuity without the clutter.

Over time, on a longer project, these session summary files build into a lightweight decision log, which is valuable for onboarding new collaborators, human or otherwise.