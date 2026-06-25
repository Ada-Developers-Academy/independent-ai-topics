# Best Practices A


### Matching the Sandbox to the Phase of Work

Not all phases of the workflow we covered in Lesson 3 require the same level of access. One approach practitioners use is to match the sandbox's permissions to what the current phase actually needs:

- **Planning and research**: Read-only access is often sufficient. The agent needs to explore the codebase and documentation but should not be writing anything. Restricting write access during this phase provides a strong guarantee that exploration won't accidentally modify files.
- **Implementation**: Write access to the project directory is necessary, but access outside it usually isn't. A container or virtual environment that scopes the agent to the project root, with no access to the broader filesystem, is a reasonable boundary for most implementation work.
- **Automated review**: Read-only access again. The reviewer agent is consuming output, not producing it. Giving review agents write access creates unnecessary risk without adding capability.

Some agentic tooling ships with default sandboxing already configured. For developers new to agentic coding, working within those defaults is a reasonable starting point. As we get more experience with what our workflows actually need, we can tune permissions to be more or less restrictive based on what the task requires.

### Sandboxing Enables Capability, Not Just Safety

A point worth internalizing: sandboxes don't just prevent problems, they change what we're comfortable letting agents do. When we know an agent is operating inside a container with no access to our broader filesystem or credentials, we can let it run more aggressive exploration steps, download and test dependencies, or iterate through failing test cases without our intervention. The security boundary is what makes that confidence possible.

---

## Adding Instructions: Steering Files and Skills

Once we have a safe execution environment, the next major lever is how we give agents the context they need to work well. In Lesson 1, we introduced steering files and skills at a high level. Here, we'll look at how to use them effectively, and just as importantly, how to avoid the common mistake of putting too much in the wrong place.

### Steering Files: What Belongs Here

A steering file is loaded into the context window at the start of every session. That single fact shapes everything about what belongs in one: content that is relevant to every task, every session, every agent.

Good candidates for steering file content include:
- Explicit rules or prohibitions that apply universally ("Always start new features in a dedicated branch", "Never modify files under `src/generated/`")
- Project-specific conventions the model wouldn't know from training, like naming patterns, import conventions, or folder organization
- Pointers to canonical examples in the codebase the model should reference when producing similar work
- The test runner command and how to interpret results for this project

What steers people wrong is treating the steering file like a knowledge base. Team members start adding edge case documentation, onboarding notes, guides for specific workflows, and the file grows to several thousand tokens. Because it's loaded on every message for the entire session, this overhead compounds continuously. A bloated steering file is one of the most consistently expensive things we can do to our token usage.

A useful heuristic: if we would need this content in a session focused on writing a new API endpoint *and* a session focused on a database migration *and* a session focused on updating documentation, it belongs in the steering file. If it's only relevant in some of those, it likely belongs in a skill.

**Example: a lean steering file for a TypeScript project**

```markdown
# Project Conventions

This is a Node.js/TypeScript project using Express for routing and PostgreSQL via pg-promise.

## Structure
- `src/routes/` — API route handlers
- `src/services/` — Business logic layer
- `src/db/` — Query files and migration scripts
- `src/generated/` — Auto-generated OpenAPI types. Do not modify directly.

## Standards
- All public functions must have JSDoc comments
- Run `npm test` to execute the test suite before committing
- Use the service layer for business logic; route handlers should stay thin
- Branch naming: `feat/<ticket>`, `fix/<ticket>`, `chore/<description>`
```

This file is short enough that it adds minimal overhead per message, but dense enough that an agent picking up any task in the project has the context it needs to stay within the team's norms.

### Skills: What Belongs Here

A skill covers procedural knowledge for a specific type of task: how to do something particular that the agent can't reliably do from training data alone. Skills are loaded on demand rather than at startup, which means we can maintain a large skill library without paying for all of it on every session.

The basic structure of a skill file looks like this:

```markdown
---
name: api-endpoint
description: Use when creating a new API endpoint. Covers route handler structure, 
             service layer setup, validation patterns, and test file scaffolding.
---

## Steps

1. Create the route handler in `src/routes/` following the existing handler structure.
2. Add the service method in `src/services/` for any business logic.
3. Use the `validate()` middleware for input validation. See `src/middleware/validate.ts` for usage.
4. Write integration tests in `tests/routes/` following the pattern in `tests/routes/users.test.ts`.
5. Register the new route in `src/app.ts`.
```

The `name` and `description` fields are critical because the description is what the agent uses to determine whether the skill applies to the current task. A description that is too vague ("general coding help") will either never load or load when it shouldn't. A description that precisely names the scenarios where the skill applies ("use when creating a new API endpoint") gives the agent a reliable trigger condition.

Keep the main SKILL.md file focused and under roughly 500 lines. Move detailed reference material such as example input/output pairs, long configuration templates, or supporting documentation into a `references/` subdirectory within the skill folder. The agent can pull these in if it needs them, without paying the token cost for them when it doesn't.

#### What We Can Do With Skills

Skills can contain more than just written instructions. The skill folder structure supports:
- **References**: Additional documentation files the agent can retrieve when it needs more detail on a step
- **Scripts**: Executable code (Python, JavaScript, or shell) the agent can run as part of the skill's workflow

Scripts are particularly useful for tasks where a specific sequence of commands needs to run reliably, like a database migration scaffold or a code generation step. Rather than hoping the agent produces the right command from its training, a skill script encodes exactly what needs to run. That said, any skill that includes scripts should be treated with the same scrutiny as any other code being run in the environment: script approval gates and sandboxing apply here too.

#### Skills and Context Windows: The Connection

We covered in Lesson 2 that context windows fill up quickly and need to be managed carefully. Skills are one of the primary tools for doing that without sacrificing capability. Because only the name and description of each skill are loaded at startup (roughly 100 tokens per skill), we can configure an agent with dozens of skills and only pay for the full instructions of the ones that are relevant to the current task.

This also means skills are worth sharing across agents and projects. A code review skill, a changelog generation skill, or an API documentation skill can be maintained in a shared library and referenced by multiple agents without duplicating instructions across steering files.

---

## Agents and Subagents

The previous lesson covered the planning, implementation, and review workflow. In the context of best practices, agents and subagents are primarily about *how we execute* that workflow efficiently at scale. There are three main levers here: which model we use for which task, when subagents are worth spinning up, and how subagents help us manage the context window of a long-running project.

### Choosing Models for the Task

Different models have meaningfully different cost and capability profiles. A model optimized for extended analytical processing will generally produce better planning documents and architectural analysis than a smaller, faster model. But it also costs more per token and may be slower. Matching the model to the phase of work is one of the most effective ways to control cost without sacrificing output quality.

A practical breakdown:

| Phase | What's Needed | Model Characteristics |
|-------|--------------|----------------------|
| Planning / architecture | Nuanced analysis, handling ambiguity, considering tradeoffs | Higher-capability models are worth the cost here; errors in the plan compound |
| Implementation | Reliable code generation to spec | Mid-tier models are often sufficient for routine implementation given a solid plan |
| Research / exploration | Reading and summarizing files, retrieving documentation | Smaller, faster models handle this well and cost significantly less |
| Boilerplate / scaffolding | Generating repetitive structures that follow a template | Smallest capable model; these tasks don't require sophistication |

This doesn't require active management on every task. Most practitioners develop a default model for their primary session and a lighter model for subagents doing routine work. As we get more experience with how our workflow actually runs, we can refine this further.

### Defining Subagents

A subagent is an agent instance given a focused task to complete in its own isolated context window. As we covered in Lesson 1, it's analogous to a helper function: it handles a specific piece of work and returns the result, without its internal process affecting the caller's context.

The key question when considering a subagent is: *will this task's context be useful to the primary session after it's done?* 

If a primary agent is implementing a feature and needs to research how a third-party library handles rate limiting, that research is valuable for the implementation but won't be needed again once the implementation is complete. Delegating the research to a subagent means the primary session only receives a focused summary of what it needs, not pages of library documentation.

**Scenarios where subagents add clear value:**

- Research and exploration tasks that consume a lot of context but produce a small, targeted output
- Parallel workstreams where independent tasks can run simultaneously (fan-out, covered in Lesson 3)
- Adversarial review, where structural isolation between author and reviewer is the entire point
- Long-running projects where delegating implementation phases to subagents keeps the orchestrator's context focused on the overall plan

**How to define a subagent**: This varies somewhat by tooling, but the general pattern is to specify the model to use, the task instructions, what files or context to give it access to, and what the expected output format is. The output should be concrete enough that the primary agent can incorporate it without ambiguity: a summary file path, a completed implementation, a structured review report.

### Subagents as a Context Window Strategy

We touched on this in Lesson 2, but it's worth reinforcing in the context of best practices: for long-running projects, the primary agent's context window is a finite resource. Every subagent we spin up for a well-defined subtask keeps the primary session's context from filling with details that don't need to persist.

The practical pattern is to use the primary agent as an orchestrator: it holds the plan, tracks what has been completed, and delegates focused chunks of work to subagents. The subagents do the heavy lifting in clean contexts and return summaries. The orchestrator accumulates results and determines the next step.

### When We Don't Need to Think About Subagents

One thing experienced practitioners note is that once we've established a workflow we're comfortable with, we often don't need to actively manage subagent configuration day-to-day. The tooling handles a lot of this. We should think more actively about our agent and subagent setup when:

- We are experimenting with a new model and want to compare its output to our current default
- We notice performance degradation that could be addressed by changing which model handles which task
- We are scaling up to a larger or more complex project where the default setup is showing strain

Otherwise, a setup that works well can run without active adjustment. The goal is to understand the system well enough to tune it when necessary, not to micromanage it constantly.

---

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