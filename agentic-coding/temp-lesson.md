# Best Practices

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
| Sandbox | An isolated execution environment that constrains what an agent can access or modify, limiting the blast radius of unintended actions. | Isolated environment | "Running the agent in a sandbox meant that even when it attempted to modify a file outside the project directory, the operation was blocked." |
| Container | A lightweight, isolated runtime environment that packages an application with its dependencies and gives it its own view of the filesystem, network, and processes, separate from the host system. | Docker container, isolated environment | "We mounted only the project directory into the container, so the agent had no visibility into the rest of the host filesystem." |
| Container image | A read-only template that defines the filesystem, dependencies, and configuration used to create a container. Running a container is always based on an image. | Docker image, base image | "The team built a container image with the project's language runtime and toolchain pre-installed so the agent could start work without a setup step." |
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

### Sandboxing Tools

Sandboxing exists on a spectrum. We don't always need full container isolation, and understanding the tools available at each level helps us match the approach to what the situation actually calls for.

#### Level 1: Built-in Tool Permission Controls

Most agentic coding tools ship with some form of permission gating at the application level. These are the coarsest controls and the first line of defense.

In practice, this looks like configuring which categories of action the agent is allowed to take without pausing for approval. Common categories include: file reads, file writes, shell command execution, and network requests. Many tools let us set each category to "allow," "deny," or "ask," where "ask" means the agent pauses and surfaces a prompt before proceeding.

This is the default sandbox for most developers and is often good enough for low-stakes tasks. If we're asking an agent to help us draft documentation or refactor a single file, application-level permission controls may be all we need. The limitation is precision: we're drawing boundaries around categories of action, not specific paths or commands. An agent with write access can write anywhere, not just to the places we intended.

Some tools allow more granular specification. It's possible in some environments to configure path-specific rules ("allow writes to `src/`, deny writes to `migrations/`") or to allowlist specific shell commands by pattern. This is more controlled than category-level permissions, but it still has a meaningful gap: a command like `git` can behave as read-only or destructively depending on the subcommand and flags it receives. Pattern-matching on command strings doesn't give us reliable guarantees about side effects.

#### Level 2: OS-Level Process Isolation

On macOS and Linux, the operating system provides tools for constraining what a process can do at a lower level than application permissions.

On macOS, **Seatbelt** (accessed via the `sandbox-exec` command) is a profile-based sandboxing system that constrains what a process is allowed to do at the OS level. A Seatbelt profile is a policy file that specifies which filesystem paths the process can read or write, whether it can open network connections, and whether it can spawn child processes. When an agent's shell process runs inside a Seatbelt policy, any operation outside the policy's rules fails with a permission error, regardless of what the agent was instructed to do.

On Linux, **seccomp** (short for secure computing mode) operates at the system call level. Rather than specifying what files a process can touch, seccomp specifies which system calls a process is allowed to make at all. A seccomp filter can, for example, block `unlink` (the call that deletes files) or `fork` (the call that creates child processes) entirely, regardless of how a program tries to invoke them. This is a more powerful but more complex tool than filesystem path restrictions.

OS-level tools are significantly more reliable than application-level permission categories because they operate below the application: even if a tool or agent framework tries to bypass a restriction, the OS enforces the boundary. The tradeoff is setup complexity. These tools require writing and maintaining configuration files, and mistakes in policy definitions can break the development environment in ways that take time to diagnose.

#### Level 3: Container Isolation

The strongest commonly-used boundary for local development is a container. Docker and similar container runtimes create fully isolated environments with their own filesystem, network stack, and process space. Anything running inside the container has no visibility into the host system beyond what is explicitly mounted in.

The basic pattern for running an agent inside a container involves:

1. **Mounting the project directory** into the container as a volume, giving the agent read/write access to project files without access to anything else on the host filesystem
2. **Restricting network access** if the agent doesn't need internet connectivity for the task
3. **Running as a non-root user** inside the container, so that even within the container the agent can't modify system files
4. **Setting resource limits** to cap CPU and memory consumption, which prevents a runaway loop from affecting the host

An example Docker invocation that applies these principles:

```bash
docker run \
  --rm \
  --user $(id -u):$(id -g) \
  --network none \
  --memory 2g \
  --cpus 1.5 \
  -v $(pwd):/workspace:rw \
  -v /home/dev/.ssh:/home/dev/.ssh:ro \
  -w /workspace \
  my-agent-image
```

Breaking this command down:
- `--rm` removes the container when it exits, so containers do not accumulate on the host
- `--user` runs the process as our current user ID rather than root
- `--network none` disables all network access
- `--memory` and `--cpus` cap memory and compute resource consumption
- `-v $(pwd):/workspace:rw` mounts the current directory into `/workspace` with read/write access
- `-v /home/dev/.ssh:/home/dev/.ssh:ro` mounts the SSH directory in read-only mode as a separate volume in case the agent needs git access over SSH
- `-w /workspace` runs in the current project directory at `/workspace` with read & write privileges
- `my-agent-image` is the Docker image that is being run to create a container with the settings above

Inside the container, the agent's file operations are limited to `/workspace`. Attempts to write to `/etc/`, `/home/`, or any other host path don't work because those paths don't exist in the container's view of the filesystem.

Container isolation is the approach to reach for when:
- An agent needs to run arbitrary scripts it generates, including fetching and executing code from the internet
- The task involves long autonomous runs where we won't be actively monitoring every action
- The development environment contains credentials or sensitive files that must not be accessible
- We want the strongest available guarantee that agent operations are contained to the project scope

Many agentic coding tools are beginning to ship with optional container-based execution, either via a bundled Docker image or as a configurable mode. This is the direction the industry is moving for longer-horizon autonomous work, precisely because the guarantee container isolation provides is qualitatively stronger than what application-level or even OS-level tools can offer.

#### Choosing the Right Sandbox

For most development work, the choice comes down to risk profile and workflow needs:

| Level | Good for | Limitations |
|---|---|---|
| Application permissions | Quick tasks, supervised sessions, low-sensitivity projects | Category-level only, below-application bypasses aren't blocked |
| OS tools (Seatbelt, seccomp, firejail) | Single-machine setups where containers add too much overhead | Setup complexity, misconfiguration risk |
| Container isolation | Autonomous runs, scripts from external sources, sensitive environments | Docker required, initial setup overhead, volume mounts need care |

Starting with whatever sandbox the agentic tool provides by default is reasonable for getting started. As we take on work that runs for longer, touches more of the filesystem, or operates closer to sensitive resources, moving toward container isolation often becomes worth the setup cost.

### Scoping Access to the Phase of Work

Not every phase of a workflow requires the same access. Leaning on the principle of least priviledge, a practical approach is to configure permissions that match what the current phase needs and nothing more:

#### Research and planning

During research, read-only access is appropriate. Reading through code, reviewing documentation, and drafting a specification don't require write access to production files. 

When research is complete and we are creating the specification, we should allow minimal write access, even if that's a single allowed file so that the implementation plan can be saved to disk for review.

#### Implementation

Write access to the project directory is necessary. Access outside it typically isn't. A container scoped to the project root provides a solid boundary for most implementation work.

#### Review

Like research, review agents consume output rather than produce it. Read-only access reduces risk without reducing capability, however, we may want limited write access to allow the agent to write the outcomes of the review to disk for persistance.

Some teams maintain different sandbox configurations for each phase, switching between them as the workflow progresses. This is more overhead to set up but produces stronger guarantees about what each phase can and cannot do.

### Sandboxing Enables Capability, Not Just Safety

Sandboxes don't only prevent problems, they change what we're comfortable letting agents do. There's a version of this that practitioners sometimes articulate as "security enables capability." When we know the agent can't accidentally modify our home directory, read our SSH keys, or escape the project scope, we're able to give it more latitude within those bounds. We can let it run scripts, explore freely, iterate on failing tests, and retry operations without monitoring every step. The constraint is what makes that confidence possible.

## Adding Instructions: Steering Files and Skills

Earlier we introduced steering files and skills at a high level, here, we'll look at how to use them effectively. Both steering files and skills are ways of providing agents with context and instructions. Understanding the distinction between them and when to add to a steering file or create a new skill matters a great deal for context window usage and overall agent performance.

### Steering Files: Always On, Always Cost

A steering file is loaded into every session at startup. From a context window perspective, it is always present. As we covered previously, anything present in the context window accumulates cost on every message throughout the session.

This shapes what should go in a steering file: only things that are universally relevant, across every session, every task, every phase of work. If a piece of content is relevant for API endpoint work but not for database migration work, it does not belong in the steering file.

Good steering file candidates:
- Rules and prohibitions that apply to everything: branch naming, files that must never be modified, required tests before committing
- Project or team-specific conventions the model wouldn't know from training, like naming patterns or import conventions
- Pointers to canonical examples in the codebase the model should reference when producing similar work
- Project structure and folder organization 
    - Depending on the size of the mapping/our code base, this architectural information can be offloaded to it's own file if every agent does not need to know the structure of the code base at all times.

What often steers people wrong is treating the steering file like a knowledge base. Team members start adding edge case documentation, onboarding notes, guides for specific workflows, and the file grows to several thousand tokens. Because it's loaded on every message for the entire session, this overhead compounds continuously. 
- A bloated steering file is one of the most consistently expensive things we can do to our token usage.

Steering files often live at the root of a project repo. As we get more comfortable with steering contents, we can look into settings to apply steering files at the workspace or user level if there is steering information we find useful to apply to all projects we work with.

**Example steering file for a TypeScript project**

```markdown
# Project Conventions

This is a Node.js/TypeScript project using Express for routing and PostgreSQL via pg-promise. `package.json` contains the approved packages in use for this project.

## Structure
- `src/routes/` — API route handlers
- `src/services/` — Business logic layer
- `src/db/` — Query files and migration scripts
- `src/generated/` — Auto-generated types. Do not modify directly.

## Standards
- All public functions must have JSDoc comments
- Run `npm test` to execute the test suite and ensure all tests pass before committing
- Use the service layer for business logic; route handlers should stay thin
- Branch naming: `feat/<ticket>`, `fix/<ticket>`, `chore/<description>`
```

This file is short enough that it adds minimal overhead per message, but helps orient an agent picking up any task in the project so they can work within the team's norms.

A useful tactic to keep our steering file lean is to start by adding information as a skill. If we find that we're needing that skill for every task, then it's likely worth migrating into the steering file. 

### Skills: On-Demand Procedural Knowledge

A quick refresher on what we learned about skills previously:

- Skills provide agents with step-by-step instructions for specific tasks that the agent can't reliably do from training data alone. 

- Unlike the steering file, skills load a light index at session start up and the full contents of skills are only loaded when an agent encounters a task that matches the skill's description. They allow a large library of team knowledge to be available to agents without deeply impacting token usage until that knowledge is actually needed.

- Skills are shareable across projects and agents. A code review skill, a deployment checklist skill, or an API documentation skill developed once can be reused across every project that benefits from it.

Skill files may live in different locations depending on our IDE. 
1. Typically there will be a hidden folder in the root of a project named something like `.agents`, `.claude`, or `.cursor` where we can create a folder named `skills` to add our custom skills.
2. For each skill, we create a new folder named after the skill and create a file named `SKILL.md` in that folder
3. Each `SKILL.md` file must contain a name, a short description of the skill it provides, and the steps the AI must take to complete the described task.

The basic structure of a skill file looks like:

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

The name and description fields act as the trigger mechanism: the description needs to be specific enough that the agent can reliably identify when the skill applies. 
- A description that is too vague ("general coding help") will either never load or load when it doesn't apply. 
- A description that precisely names the scenarios where the skill applies ("use when creating a new API endpoint") gives the agent a reliable trigger condition.

Skill file sizes will range depending on what they describe, but in general, SKILL.md files should be focused on a single task and under roughly 500 lines. For more complicated skills, detailed reference material such as example input/output pairs, long configuration templates, or supporting documentation can be placed into a `references/` subdirectory within the folder for that specific skill. The agent can pull these reference documents in if it needs them, without paying the token cost for them when it doesn't. 

#### Skills Can Include More Than Instructions

Beyond the main `SKILL.md` file, the skill folder structure supports:

- **References**: A `references/` subdirectory for supporting documentation like style guides, example files, and detailed configuration references. These are fetched on demand by the agent when it needs more depth on a step of a skill.
- **Scripts**: A `scripts/` subdirectory for executable code the agent can run as part of the skill workflow.

Scripts are particularly useful when a task involves a precise sequence of commands that needs to run the same way every time. Rather than relying on the agent to produce the command correctly from its training data, the script encodes exactly what runs. 
- Scripts should be treated with the same security awareness as any other executable code: they run in the agent's execution environment and belong inside a sandbox.

## Agents and Subagents in Practice

In the context of best practices, agents and subagents are primarily about *how we execute* our workflow efficiently at scale. There are three main levers we'll look at in this section: 
- which model we use for which task
- when subagents are worth spinning up
- when it's worth defining our own agents

### Model Selection

AI models vary significantly in cost, speed, and capability. A model optimized for extended analytical processing will generally produce better planning documents and architectural analysis than a smaller, faster model, but it also costs more per token and may be slower. Matching  AI model to what the task actually requires is one of the most effective ways to reduce cost without sacrificing the quality of important outputs.

A general framework for choosing models looks like:

| Task Type | Required Capabilities | Model Characteristics |
|-------|--------------|----------------------|
| Planning / architecture | Nuanced analysis, handling ambiguity, considering tradeoffs | Higher-capability models are worth the cost here; errors in the plan compound and are expensive to fix downstream |
| Implementation | Reliable code generation to spec | Mid-tier models are often sufficient for routine implementation given a given a clear and well-specified plan |
| Research / exploration | Reading and summarizing files, retrieving documentation | Smaller, faster models handle this well and cost significantly less |
| Boilerplate / scaffolding | Generating repetitive structures that follow a template | Smallest capable model; A lighter model that can follow an existing pattern is sufficient for this kind of work. |

Model choice doesn't require active management for every task. Many practitioners choose a default model for their primary session and a lighter model for subagents doing routine work. Some versions of IDEs even have auto-model selection capabilities that try to match appropriate models for a task based on our prompt. 
- As we get more experience with how our workflow actually runs, we can refine our agent choices further!

As an example of how this can look in practice, let's assume we have a team that is building out a new notification service. They might map AI models to the tasks in the project like so: 
1. A higher-capability model is used to draft the architecture and implementation plan with a human reviewing the results at each step.
2. As part of planning, a research subagent uses a smaller model to pull documentation for the notification library and summarize existing related code in the project. 
3. Once the plan is finalized, the team has a subagent using a mid-tier model implement the code. 
4. The orchestrating session, still running the higher-capability model, coordinates results from subagents and keeps track of the progress through project tasks.

### Using Subagents

The reality is that as IDEs and agent harnesses continue to develop, they are getting more sophisticated around when to spin work off to subagents without us directly saying "Do some task in a subagent session". More and more frequently, using a subagent is not an explicit choice we are making, but is what's happening with our agentic coding tool set under the hood. 
- If you have experimented with agentic coding before, it's possible that you've already experienced working with subagents in a way that abstracted away the subagent sessions!

When sitting down to work in an agentic set up, we may not be instructing the primary agent to spin up a subagent for a task, but it's useful for our understanding of the systems to know when subagents are useful, and when a system is likely to spin up a subagent session that will impact our usage and costs. 

Subagents are usually not necessary for short, simple tasks where the cost of spinning up an isolated context exceeds the benefit of isolation. Splitting off work to subagents is worth the cost and coordination when:
- The task will generate significant context that isn't useful to the primary session afterward (research, exploration, debugging a specific module)
- Multiple independent tasks can run in parallel, and running them sequentially in the primary session would be slower without any benefit
- We're doing adversarial review, where structural separation between the agent that wrote the code and the agent that reviews it is the point
- The primary session is deep into a long-running project and we want implementation of a distinct phase to happen in a clean context

### Defining a Custom Agent

Most agentic coding tools ship with a default general-purpose agent. That default is a reasonable starting point, but as our workflows mature, we often find that different tasks want different configurations locked in rather than re-specified each time. This is where custom agents become useful.

A custom agent is a saved profile that combines a name, a model preference, a set of permitted tools, and a system prompt. When we activate it, the tool loads that configuration automatically. We switch roles rather than rebuild a configuration from scratch.

**The structure of a custom agent file**

Custom agents are typically defined as markdown files. Depending on the tool, they live in a folder like `.github/agents/`, `.claude/agents/`, or a user-level agents directory. The filename usually becomes the agent's identifier.

The file has two parts: a YAML frontmatter block that sets configuration, and a body that contains the system prompt.

```markdown
---
name: code-reviewer
description: Reviews implementation for correctness, edge cases, and security issues.
tools: ['search/codebase', 'search/usages']
model: claude-sonnet-4-6
---

You are reviewing code that has been implemented against a specification. Your job is
to identify problems, not to rewrite the code.

Focus your review on:
- Whether the implementation satisfies the requirements in the spec
- Missing edge case handling
- Security concerns (injection risks, credential handling, input validation)
- Anything that would be caught in a team code review but might be missed otherwise

Write your findings to `docs/reviews/<branch-name>-review.md` with a PASS or FAIL
verdict and an itemized list of issues.
```

The frontmatter fields that matter most:

- **tools**: This is the access control layer. A reviewer agent that lists only read-only tools cannot accidentally write files even if the body prompt doesn't explicitly say not to. This is the mechanism that makes phase-appropriate access reliable rather than prompt-dependent.
- **model**: Different phases of our workflow benefit from different models. Pinning a model at the agent level means we don't have to remember to switch it manually.
- **description**: Some tools use this to surface the agent in a UI picker or to help an orchestrating agent select the right subagent for a task. Worth writing clearly.

The body prompt is the system prompt for the agent's sessions. It should be specific enough that the agent behaves consistently without us restating context every time we activate it.

**When a custom agent is the right tool**

The distinction between a skill and a custom agent is worth being clear on. A skill adds procedural knowledge for a specific task, but it doesn't change tool access or the base model. A custom agent locks in a combination of tool permissions, a model, and a persistent set of instructions as a reusable role. Changing any of those three things is a signal that a custom agent is more appropriate than a skill.

A few practical scenarios:

- A **planning agent** configured with read-only tools and a more capable model, so there's no risk of it writing code during a research-heavy planning phase
- An **adversarial review agent** with read-only access and explicit instructions to look for violations and edge cases the implementation agent might have missed
- A **documentation agent** scoped to write only within `docs/` directories, preventing it from touching source files even if given a broad prompt
- A **migration agent** for a specific recurring task, like schema migrations, with the relevant skill pre-loaded and tool access scoped to the database migration toolchain

**Handoffs: connecting agents in sequence**

Some tools support defining handoff actions between agents. A handoff is a button that appears after an agent completes its work, letting us move directly into the next configured agent with relevant context already in scope. A planning agent can offer a handoff to an implementation agent. An implementation agent can hand off to a review agent.

This is useful for making the phases of the recommended workflow tangible and easy to navigate without manually reconfiguring things between steps. The plan phase ends, we select "Begin Implementation," and the implementation agent activates with the plan document already referenced in context.

**Where to store them**

Project-level agents live in the repository and are available to the whole team. User-level agents are stored in a personal directory and follow us across projects. For team workflows, project-level is generally better: the agent configuration versions alongside the code, updates are shared automatically, and new team members get the configuration without setup.

### A Note on How Much to Think About Subagents

Experienced practitioners frequently note that once we've established a workflow we're comfortable with, we often don't need to actively manage subagent configuration day-to-day. The tooling handles a lot of this, so we should think more actively about our agent and subagent setup when:

- We are experimenting with a new model and want to compare its output to our current default
- We notice performance degradation that could be addressed by changing which model handles which task
- We are scaling up to a larger or more complex project where the default setup is showing strain

The goal is to understand the system well enough to tune it when necessary, we should not need to micromanage it constantly.

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