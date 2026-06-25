# Best Practices WIP

The previous lessons gave us foundations: what agents are and how they work, how context windows determine the output and usefulness of a session, and the shape of workflows that experienced practitioners tend to converge on. 

This lesson shifts focus from *what* to do toward *how* to do it well. We'll connect the practices described here back to the workflow we've already seen, because best practices don't exist independently of the systems they support. Understanding *why* a practice matters makes it easier to adapt them to our specific situations and projects.

## Learning Goals

- Explain the purpose of a sandbox in an agentic coding environment and describe what level of access to scope for different phases of work.
- Distinguish what belongs in a steering file versus a skill, and explain how that distinction affects context window usage.
- Define the structure of a skill and write a basic skill file.
- Explain how model selection affects workflow performance and cost.
- Identify when subagents are worth the added complexity and describe how to define one.
- Describe when starting a new session is preferable to continuing an existing one.

## Vocabulary and Synonyms

| Vocab | Definition | Synonyms | How to Use in a Sentence |
| --------- | --------- | -------- | --------- |
| Sandbox | An isolated execution environment that constrains what an agent can access or modify, limiting the blast radius of unintended actions. | Isolated environment, container | "Running the agent in a sandbox meant that even when it attempted to modify a file outside the project directory, the operation was blocked." |
| Steering file | A persistent markdown document loaded into every agent session that provides project-level context the model cannot otherwise know. | AGENTS.md, CLAUDE.md, project config | "We kept the steering file short and focused on universal conventions across our project." |
| Skill | A markdown document that provides an agent with procedural instructions for a specific task, loaded on demand when relevant. | Agent skill, SKILL.md | "The team wrote a skill for their API documentation format so every agent session produces consistent endpoint documentation without re-explaining the format each time." |

## Sandboxing and Scoping Access

We've established in earlier lessons that agents operate on statistical text prediction. An agent doesn't know that it shouldn't write to a directory outside our project; it simply produces whatever output its context makes statistically likely. 

Sandboxing is how we enforce constraints that prompts alone cannot reliably maintain. A **sandbox** is an isolated environment that constrains what an agent can reach. By scoping the access that AI agents have, we limit the potential blast radius of unexpected agent actions. 

### Why Sandboxes Matter

Without sandboxing, an agent with file system access can write to any location our user account has permission to write to. An agent with shell access can run any command available in that shell. This is rarely what we want! The most common problems sandboxes protect against are:

**Scope creep in the filesystem**: Without boundaries, an agent with file system access can write anywhere the current user has permissions. In a typical development environment, that includes dotfiles, other project directories, shell configuration, and potentially SSH keys or local environment variables. None of this is intentional, and most of it would be the result of an agent that missed the goal of an instruction or got stuck in a loop.

**Credential exposure**: Local development environments often have API keys, database credentials, or cloud credentials in environment variables or dotfiles. An agent with unrestricted file system access can read these and inadvertently include them in its output or logs.

**Runaway iteration**: An agent stuck retrying a failing operation will keep retrying. Without rate limits or operation counts configured in the sandbox, a loop can exhaust API quotas, make hundreds of file writes, or run indefinitely before we notice.

Let's explore a potential scenario: Imagine an agent helping with a data migration. The task is to transform a set of CSV files in `data/input/` and write the output to `data/output/`. While working, the agent finds a similar `data/output/` path in a different project's directory and tries to write the output in that folder.
- If a sandbox is configured that restricts writes to the current project root, then the write operation is blocked, the agent cannot write files to the other project's folder. 
- Without sandboxing, the error might not surface until someone notices the output is "missing" or someone sees unexpected files in a different project.

### Scoping Access to the Phase of Work

Not every phase of a workflow requires the same access. Leaning on the principle of least priviledge, a practical approach is to configure permissions that match what the current phase needs and nothing more:

**Research and planning**: 

During research, a read-only sandbox is appropriate. Reading through code, reviewing documentation, and drafting a specification don't require write access to production files. This also makes it safe to run more aggressive exploration steps, since the worst case is a failed read rather than an unintended write. 

When research is complete and we are creating the specification, we should allow minimal write access, even if that's a single allowed file so that the implementation plan can be saved to disk for review.

**Implementation**: 

Write access to the project directory is necessary. Access outside it typically isn't. A container scoped to the project root provides a solid boundary for most implementation work.

**Review**: Like research, review agents consume output rather than produce it. Read-only access reduces risk without reducing capability.

Some teams take a step further and maintain different sandbox configurations for each phase, switching between them as the workflow progresses. This is more overhead to set up but produces stronger guarantees about what each phase can and cannot do.

### The Counterintuitive Benefit

There's a version of this that practitioners sometimes articulate as "security enables capability." When we know the agent can't accidentally modify our home directory, read our SSH keys, or escape the project scope, we're able to give it more latitude within those bounds. We can let it run scripts, explore freely, iterate on failing tests, and retry operations without monitoring every step. The constraint is what makes that confidence possible.

---

## Steering Files and Skills: Getting the Separation Right

Both steering files and skills are ways of providing agents with context and instructions. The most common mistake in using them is not understanding that the distinction between them matters a great deal for context window usage and overall agent performance.

### Steering Files: Always On, Always Cost

A steering file is loaded into every session at startup. From a context window perspective, it is always present. As we covered in Lesson 2, anything present in the context window accumulates cost on every message throughout the session.

This shapes what should go in a steering file: only things that are universally relevant, across every session, every task, every phase of work. If a piece of content is relevant for API endpoint work but not for database migration work, it does not belong in the steering file.

Good steering file candidates:
- Rules and prohibitions that apply to everything: branch naming, files that must never be modified, required tests before committing
- Project structure and folder organization, so any agent knows where things live
- Naming conventions, import patterns, and style guidelines that apply project-wide
- Pointers to canonical examples in the codebase

What tends to bloat steering files unnecessarily:
- Onboarding documentation or background context
- Workflow guides that only apply to specific tasks
- Long explanations for things skills should handle
- Documentation of edge cases for specific features

**A lean steering file example for a Python project:**

```markdown
# Project Conventions

This is a Python/FastAPI project using SQLAlchemy for ORM and pytest for testing.

## Structure
- `app/routes/` — API route definitions
- `app/services/` — Business logic (keep routes thin)
- `app/models/` — SQLAlchemy models
- `migrations/` — Alembic migration scripts. Do not edit manually.

## Standards
- All functions must have type annotations
- Run `pytest` before any commit
- Use Black for formatting: `black app/`
- New features start in a branch: `feat/<ticket-number>-<short-description>`
```

This file loads on every message throughout the session. Its job is to ensure any agent working on any task in this project knows the basic rules and structure. That's all it needs to do.

### Skills: On-Demand Procedural Knowledge

A skill provides an agent with step-by-step instructions for a specific kind of task. Unlike the steering file, it loads only when the agent encounters a task that matches the skill's description.

The structure of a skill file:

```markdown
---
name: database-migration
description: Use when creating a new Alembic migration, modifying the database schema,
             or troubleshooting migration conflicts.
---

## Steps

1. Create the migration with `alembic revision --autogenerate -m "<description>"`.
2. Review the generated file in `migrations/versions/`. Autogenerate is not always accurate.
3. Check that `upgrade()` and `downgrade()` are both implemented correctly.
4. Test locally with `alembic upgrade head` before committing.
5. Note in the PR that a migration is included so reviewers know to run it.
```

The name and description fields are the trigger mechanism. The description needs to be specific enough that the agent can reliably identify when the skill applies. "Use when creating database migrations" is more reliable than "database help" because it leaves less ambiguity about when to load the skill.

#### Skills Can Include More Than Instructions

Beyond the main SKILL.md file, a skill folder can contain:

- A `references/` subdirectory for supporting documentation: style guides, example files, detailed configuration references. These are fetched on demand by the agent when it needs more depth on a step.
- A `scripts/` subdirectory for executable code the agent can run as part of the skill workflow.

Scripts are particularly useful when a task involves a precise sequence of commands that needs to run the same way every time. Rather than relying on the agent to produce the command correctly from its training data, the script encodes exactly what runs. Scripts should be treated with the same security awareness as any other executable code: they run in the agent's execution environment and belong inside a sandbox.

#### Skills and the Context Window

Because skills are loaded on demand, they allow a large library of team knowledge to be available to agents without affecting token usage until that knowledge is actually needed. Only the short advertised descriptions of each skill contribute to startup cost. The full instructions are only loaded when they match a task.

This also means skills are shareable across projects and agents. A code review skill, a deployment checklist skill, or an API documentation skill developed once can be reused across every project that benefits from it.

---

## Agents and Subagents in Practice

Lesson 1 introduced agents and subagents conceptually; Lesson 3 showed how they fit into the recommended workflow through patterns like fan-out. Here, we focus on the practical configuration decisions: which model to use, when a subagent is worth the added complexity, and how to define one.

### Model Selection

AI models vary significantly in cost, speed, and capability. For most agentic setups, the same model doesn't need to run every task. Matching the model to what the task actually requires is one of the most effective ways to reduce cost without sacrificing the quality of important outputs.

A general framework:

**Planning and architecture work** benefits from higher-capability models. Planning documents and architectural decisions set the direction for everything that follows. Errors at this stage are expensive to fix downstream. The higher cost per token is justified by the higher stakes of the output.

**Routine implementation** given a clear and well-specified plan is a less demanding task. A mid-tier model working from a solid plan is often as effective as a higher-capability model working from a vague one, and costs less.

**Research and file exploration** is largely pattern matching over text. Smaller, faster models are well-suited to reading files, surfacing relevant sections, and returning summaries. Running exploration in a subagent with a lighter model is both cheaper and keeps the primary session's context clean.

**Boilerplate and scaffolding** generation is the lowest-stakes, most templated category. A lighter model that can follow the existing pattern in the codebase is sufficient for this kind of work.

**A practical scenario**: A team is building out a new notification service. They use a higher-capability model to draft the architecture and implementation plan with a human reviewing the result at each step. Once the plan is finalized, they run implementation in a subagent using a mid-tier model. Two research subagents using a smaller model run in parallel to pull documentation for the notification library and summarize existing related code in the project. The orchestrating session, still running the higher-capability model, coordinates results and handles the parts of the work that require cross-context judgment.

This isn't over-engineering. It's recognizing that different phases of the same project have different requirements, and letting the configuration reflect that.

### When to Use Subagents

We've covered subagents extensively already. In the context of best practices, the key question is: when does the overhead of setting up a subagent actually pay off?

Subagents are worth the added configuration when:

- The task will generate significant context that isn't useful to the primary session afterward (research, exploration, debugging a specific module)
- Multiple independent tasks can run in parallel, and running them sequentially in the primary session would be slower without any benefit
- We're doing adversarial review, where structural separation between the agent that wrote the code and the agent that reviews it is the point
- The primary session is deep into a long-running project and we want implementation of a distinct phase to happen in a clean context

Subagents are usually not worth it for short, simple tasks where the overhead of spinning up an isolated context exceeds the benefit of isolation.

#### Defining a Subagent

The mechanics of defining a subagent vary by tooling, but the key elements are consistent:

1. The model to use for the subagent
2. The task instructions: specific enough that the subagent doesn't need to ask clarifying questions
3. What files or context to provide
4. The expected output format: a file path, a structured JSON summary, a completed implementation

The output specification matters more than it might seem. A subagent that returns unstructured prose is harder for an orchestrator to act on than one that returns a summary in a predictable format or writes its output to a specified file path.

### A Note on How Much to Think About This

Experienced practitioners consistently note that once a workflow is running well, they rarely think about agents and subagents actively. The tooling handles the orchestration. It's worth spending time upfront to configure things well, but the goal is a setup that can run without constant adjustment. 

The moments to revisit agent and subagent configuration are:
- When experimenting with a new model and wanting to compare its output
- When a workflow that's been working well starts showing signs of strain (context saturation, inconsistent outputs, ballooning costs)
- When taking on a significantly more complex or larger-scale project than previous work

Outside of those moments, a well-configured setup runs in the background.

---

## When to Start a New Session

In Lesson 2, we covered starting a new session as one of the tools for managing a full context window. Here, we want to reframe it: starting a new session isn't a fallback for when things go wrong. It's a deliberate choice that often produces better results than continuing.

### What We're Actually Leaving Behind

When we start a new session, we don't lose:
- Anything written to a file
- Plan documents
- Code changes committed to version control
- Steering file configuration
- Skill files

What we leave behind is the accumulated conversation history: the exploratory tangents, the alternatives we considered and discarded, the back-and-forth that produced the current direction. For a session that has been running long enough to accumulate significant history, most of that history is noise from the perspective of the next task.

A clean session working from a written plan and the current codebase state often produces more focused output than a session carrying hours of exploratory context.

### Recognizing When to Switch

Some clear signals that a new session would serve us better:

**Phase transitions**: Moving from planning to implementation is often a good natural point to start fresh. The planning session produced a specification; the implementation session loads that specification and the codebase. There's no benefit to bringing the planning discussion along.

**Completed topics**: If a session included significant investigation into an issue that's now resolved, the context from that investigation doesn't help with the next task. It just adds to the overhead the model processes on every message.

**Output quality drift**: Agents don't flag when context saturation is affecting their outputs. If we notice outputs becoming less specific, more generic, or less consistent with instructions that were clear earlier in the session, this is often a sign that the context window is working against us.

**A genuinely new task**: Starting a different feature, a separate bug fix, or a task in a different part of the codebase is almost always better done in a fresh session. There's no reason to pay for unrelated context on every message.

### Building the Session Handoff Habit

The practice that makes new sessions feel less disruptive is treating the end of each session as a handoff step. Before closing a session that has done substantial work, it's worth asking the agent to write a brief summary file:

```
Please write a session summary to docs/session-notes/2024-03-15.md including:
- Decisions made and why
- Files modified
- Open questions or blockers
- What the next session should start with
```

This summary file becomes the starting context for the next session. It can be loaded alongside the steering file and the relevant plan documents to give the new session the essential continuity without the clutter of the full prior history.

Over the course of a longer project, these session summaries build into something useful beyond just session continuity: a chronological record of how the project evolved, which decisions were made and why, and what was tried and abandoned. This kind of record is genuinely useful for project retrospectives, onboarding new team members, or revisiting a decision that was made months earlier.