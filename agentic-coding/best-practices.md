# Best Practices

The previous lessons gave us foundations: what agents are and how they work, how context windows determine the output and usefulness of a session, and the shape of workflows that experienced practitioners tend to converge on. 

This lesson shifts focus from *what* to do toward *how* to do it well. We'll connect the practices described here back to the workflow we've already seen, because best practices don't exist independently of the systems they support. Understanding *why* a practice matters makes it easier to adapt them to our specific situations and projects.

## Learning Goals

- Explain the purpose of a sandbox in an agentic coding environment and describe what level of access to scope for different phases of work.
- Distinguish what belongs in a steering file versus a skill, and explain how that distinction affects context window usage.
- Define the structure of a skill and write a basic skill file.
- Explain how model selection affects workflow performance and cost.
- Identify when subagents are worth the added complexity and describe how to define one.

## Vocabulary and Synonyms

| Vocab | Definition | Synonyms | How to Use in a Sentence |
| --------- | --------- | -------- | --------- |
| Sandbox | An isolated execution environment that constrains what an agent can access or modify, limiting the blast radius of unintended actions. | Isolated environment | "Running the agent in a sandbox meant that even when it attempted to modify a file outside the project directory, the operation was blocked." |
| Container | A lightweight, isolated runtime environment that packages an application with its dependencies and gives it its own view of the filesystem, network, and processes, separate from the host system. | Docker container, isolated environment | "We mounted only the project directory into the container, so the agent had no visibility into the rest of the host filesystem." |
| Container image | A read-only template that defines the filesystem, dependencies, and configuration used to create a container. Running a container is always based on an image. | Docker image, base image | "The team built a container image with the project's language runtime and toolchain pre-installed so the agent could start work without a setup step." |
| Steering file | A persistent markdown document loaded into every agent session that provides project-level context the model cannot otherwise know. | AGENTS.md, CLAUDE.md, project config | "We kept the steering file short and focused on universal conventions across our project." |
| Skill | A markdown document that provides an agent with procedural instructions for a specific task, loaded on demand when relevant. | Agent skill, SKILL.md | "The team wrote a skill for their API documentation format so every agent session produces consistent endpoint documentation without re-explaining the format each time." |
| YAML frontmatter | A structured block of metadata written in YAML syntax that appears at the top of a markdown file, enclosed between two lines of triple dashes (`---`). | Front matter, metadata block | "The YAML frontmatter in our skill file sets the name and description the agent uses to decide when to load it." |

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

Many agentic coding tools are beginning to ship with optional container-based execution, either via a bundled Docker image or as a configurable mode. This is the direction the industry is moving for longer-term autonomous work, precisely because the guarantee container isolation provides is qualitatively stronger than what application-level or OS-level tools can offer.

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

Sandboxes don't only prevent problems, they change what we're comfortable letting agents do. There's a version of this that practitioners sometimes articulate as "security enables capability." When we know the agent can't accidentally modify our home directory, read our SSH keys, or escape the project scope, we're able to give it more latitude within those bounds. We can let it run scripts, explore freely, iterate on failing tests, and retry operations without monitoring every step. The constraint is what makes that confidence possible!

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

#### Skill Structure

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

##### Skills Can Include More Than Instructions

Beyond the main `SKILL.md` file, the skill folder structure supports:

- **References**: A `references/` subdirectory for supporting documentation like style guides, example files, and detailed configuration references. These are fetched on demand by the agent when it needs more depth on a step of a skill.
- **Scripts**: A `scripts/` subdirectory for executable code the agent can run as part of the skill workflow.

Scripts are particularly useful when a task involves a precise sequence of commands that needs to run the same way every time. Rather than relying on the agent to produce the command correctly from its training data, the script encodes exactly what runs. 
- Scripts should be treated with the same security awareness as any other executable code: they run in the agent's execution environment and belong inside a sandbox.

#### Writing New Skills

One approach to creating skills is to sit down and write out what you want from the start. A problem many developers run into with this tactic, is that they miss a lot of finer details in that first draft. This tends to produce skills that are either too abstract to be useful or that miss the specific decision points where an agent needs guidance. Skills built this way can require significant debugging and updates to catch edge cases and fill in any steps or information that weren't top of mind when the skill was written.

Writing instructions for a task we haven't watched an agent attempt is similar to writing a training guide for a job we've never observed being done. Rather than us trying to remember and document everything clearly without prompting, the most reliable method for writing a skill that consistently works is to work through the task that we want to build a skill for with an agent first. We observe where it succeeds and where it goes wrong, and then convert that proven interaction into a skill.

##### The Walkthrough Approach

The process starts with a fresh session and a real instance of the task. We give the agent the goal and watch what happens:
- Where does it make correct choices without instruction?
- Where does it go wrong or produce something inconsistent with what we want?
- What steps does it invent on its own that we don't actually want?
- Where does it get stuck or ask for clarification?

At each point where the agent deviates from what we want, we correct it in the session: "No, check the staging environment first before touching production configs" or "The output should be a markdown table, not a prose summary." We keep going until the agent completes the task in a way we'd be comfortable seeing repeated.

That corrected walkthrough becomes the skill! We've now observed the actual decision points that matter for this task in our specific environment, and we've identified which ones the agent won't get right on its own. 
- Everything we corrected becomes a step or a gotcha in the skill file.

**Ask the agent to document what it just did**

Once a walkthrough is complete and we're satisfied with the result, we can ask the agent to draft the skill file directly from the session: 

> "Review what you just did to complete this task. Write a SKILL.md that would guide an agent through the same process reliably."

The agent has the full session context and can produce a reasonable first draft of the steps, ordered as it actually executed them. We review this draft, add any steps it omitted, sharpen the description field, and move the file into the skills directory.

This is significantly faster than drafting instructions from memory and produces a skill that reflects how the task actually runs rather than how we remember it.

**Scenario: a release notes skill**

Imagine a team that wants a consistent format for their release notes. A developer tries asking an agent to generate release notes from a set of merged pull requests and gets back something that's technically accurate but uses a different structure every time, includes internal ticket references that shouldn't be customer-facing, and buries breaking changes rather than calling them out at the top.

Rather than writing a skill based on their style preferences, the developer works through one release with the agent in a live session, correcting the output at each point: moving breaking changes to a dedicated section at the top, stripping internal ticket numbers, adjusting the summary format. Once the output matches what they'd actually ship, they ask the agent to generate a SKILL.md from the session.

The resulting skill includes the exact format they ended up with, a note that ticket numbers matching the internal pattern should be excluded, and a checklist step to verify breaking changes are surfaced first. 

**Refining the skill over time**

A walkthrough produces a first version, not a final one. Each time we run the skill and notice the agent making a mistake, we add the correction to the skill file. A "gotchas" section in the skill works well for this: concrete, environment-specific facts that contradict what the agent would assume by default.

```markdown
## Gotchas

- The `v2/` API endpoints require a `X-API-Version` header. The v1 endpoints 
  do not. Omitting this header on v2 calls returns a 200 with empty data, 
  not an error.
- Draft PRs are included in the GitHub API response for merged PRs. 
  Filter to `state: merged` and `draft: false` explicitly.
```

This kind of detail can be hard to derive from planning alone, it comes from running the task and observing where things break. The walkthrough approach builds those observations directly into the skill creation process rather than treating them as corrections to fix after the fact.

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

When sitting down to work in an agentic set up, we may not be instructing the primary agent to spin up a specific subagent for a task often, but it's useful for our understanding of the systems to know when subagents are useful, and when a system is likely to spin up a subagent session that will impact our usage and costs. 

Subagents are usually not necessary for short, simple tasks where the cost of creating an isolated context exceeds the benefit of isolation. Splitting off work to subagents is worth the cost and coordination when:
- The task will generate significant context that isn't useful to the primary session afterward (research, exploration, debugging a specific module)
- Multiple independent tasks can run in parallel, and running them sequentially in the primary session would be slower without any benefit
- We're doing adversarial review, where structural separation between the agent that wrote the code and the agent that reviews it is the point
- The primary session is deep into a long-running project and we want implementation of a distinct phase to happen in a clean context

### !callout-info

## A Note on How Much to Think About Subagents

Experienced practitioners frequently note that once we've established a workflow we're comfortable with, we don't need to actively manage subagent configuration day-to-day. The tooling handles a lot of this, so we should think more actively about our agent and subagent setup when:

- We are experimenting with a new model and want to compare its output to our current default
- We notice performance degradation that could be addressed by changing which model handles which task
- We are scaling up to a larger or more complex project where the default setup is showing strain

The goal is to understand the system well enough to tune it when necessary, we should not need to micromanage it constantly.

### !end-callout

### Creating Custom Agents

Most agentic coding tools ship with a default general-purpose agent. That default is a reasonable starting point, but as our workflows mature, we may find that certain tasks benefit from having specific configurations locked in at start up. 

A custom agent is a saved profile that combines a name, a model preference, a set of permitted tools, and a system prompt. We define the agent file once and activate it by name whenever the same kind of work comes up. This allows us to switch roles rather than rebuild configurations from scratch!

#### Agent Files

Custom agents are typically defined as markdown files. The file location varies by tool but is usually a project-level folder like `.github/agents/`, `.claude/agents/`, or a user-level directory for agents that should be available across all projects.
- Project-level agents live in the repository and are available to the whole team. 
- User-level agents are stored in a personal directory and follow us across projects. 

For team workflows, project-level is generally preferred: the agent configuration versions alongside the code, updates are shared automatically, and new team members get the configuration without setup.

The file has two parts: 
1. a YAML frontmatter block that sets configuration
2. a body that contains the system prompt

For example, a custom agent for code reviewing might look something like:

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

The key fields:

- **`name`**: How we refer to and activate this agent
- **`description`**: What the agent is for; some tools surface this in the UI
- **`tools`**: The specific tools this agent is allowed to use. Restricting tools here is how we enforce phase-appropriate access without relying on the agent to self-limit.
- **`model`**: The model to use for this agent's sessions. This is where we can pin a higher-capability model for planning work and a lighter one for agents doing routine tasks.
- **`body`**: The prompt loaded on start up that defines the agent's behavior and any specific instructions. The body prompt should be specific enough that the agent behaves consistently without us restating context every time we activate it.

**When a custom agent is the right tool**

A one-off task rarely justifies a custom agent. The overhead of defining a custom agent pays off when a workflow recurs often enough that rebuilding its configuration manually each time becomes a friction point. 
- When we find ourselves repeatedly selecting the same model, enabling the same tools, and restating the same framing before getting to work, that's the strongest signal that a custom agent would serve us better. 

Another signal is when the stakes of misconfiguration are high enough that we don't want to rely on getting it right each time. A planning agent that accidentally has write access, or a review agent that's running a smaller model than the task warrants, produces subtly worse results in ways that aren't always obvious in the moment. Encoding the right model and the right tool permissions into a named agent removes that variability. 

As we mentioned briefly earlier, custom agents become particularly valuable when they're shared across a team. A project-level agent file that everyone uses means the whole team is running planning sessions with the same model, the same tool constraints, and the same base instructions. That consistency matters more than it might seem: 
- it makes workflow outputs more predictable
- it makes it easier to identify when something goes wrong
- it means improvements to the agent benefit everyone automatically rather than requiring each team member to update their own setup

A few practical scenarios where custom agents can be useful:
- A **planning agent** configured with read-only tools and a more capable model, instructed to output a structured spec file, so there's no risk of it writing code during a research-heavy planning phase and a plan is saved to disk for review.
- An **adversarial review agent** with read-only access and explicit instructions to look for violations and edge cases the implementation agent might have missed.
- A **documentation agent** scoped to write only within `docs/` directories, preventing it from touching source files even if given a broad prompt.
- A **migration agent** for a specific recurring task, like schema migrations, with the relevant skill pre-loaded and tool access scoped to the database migration toolchain.

## Summary

The practices in this lesson share a common thread: matching our configuration to what the task actually requires, rather than accepting defaults or over-provisioning everything.

Sandboxes protect us not just from security failures, but from the kinds of unintended side effects that happen when agents operate with more access than the task needs. The right sandbox level depends on the risk profile of the work:
- Quick, supervised tasks may only need application-level permission controls.
- Autonomous runs in environments with sensitive credentials call for container isolation, where the agent's filesystem view is strictly bounded by what we mount in. 

Scoping access to the current phase of work reduces risk further by ensuring a research agent can't write to production files and a review agent can't do anything destructive.

Steering files and skills are complementary tools with distinct cost profiles. Everything in a steering file is paid for continuously, throughout every session, because it's present in the context window at all times. Skills shift that cost to on-demand: the agent pays for a skill's contents only when it encounters a task that matches the skill's description. 
- Keeping steering files lean and moving task-specific knowledge into skills is one of the most direct ways to reduce ongoing token costs. 
- Building skills through live walkthroughs rather than from memory produces more accurate instructions because it captures the actual failure points an agent will encounter, not just the steps we think to document in advance.

Model selection shapes both cost and output quality. The most expensive model is not always the right one; it's most defensible for planning work where downstream errors compound, less so for research or boilerplate tasks where lighter, less-costly, models perform well. Custom agents make model choice persistent, encoding the right model alongside the right tool permissions and a base system prompt into a named configuration that can be activated by name and shared across a team.

## Check for Understanding

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: R3kJmN8pQvL0wDxHbT5cYeA6f
* title: Best Practices
##### !question

An agent is configured with application-level permission controls and given write access to the entire file system. While working on a task scoped to `data/output/`, the agent writes files to a different project's directory. What does this scenario illustrate?

##### !end-question
##### !options

a| Application-level controls are sufficient, but the agent's system prompt needs to be more specific about the target directory.
b| Application-level controls typically allow writes to all accessible files rather than supporting scoping to specific paths, so they cannot prevent writes to unintended directories.
c| The agent should have been given read-only access for all phases of the workflow.
d| This behavior is expected and unavoidable without rewriting the task prompt.

##### !end-options
##### !answer

b|

##### !end-answer
##### !explanation

Application-level permission controls operate at the category level, meaning "allow file writes" permits writes anywhere the user account has permissions. They cannot restrict writes to a specific directory. This is a key limitation of application-level sandboxing compared to OS-level or container-based approaches, which can enforce path-specific boundaries.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: m7TsA1eWqN5bUjXcF0rKgY4vP
* title: Best Practices
##### !question

A team's steering file has grown to include onboarding documentation, API workflow guides, and edge case notes for several features. What is the most significant problem with this approach?

##### !end-question
##### !options

a| Steering files do not support markdown formatting, so the content will not render correctly.
b| The steering file contents are loaded on every session and re-processed on every message, so the added content increases token costs continuously throughout every session.
c| Agents will ignore content in a steering file that exceeds 500 lines.
d| Steering files can only be read by one agent at a time, so multiple team members working simultaneously will encounter conflicts.

##### !end-options
##### !answer

b|

##### !end-answer
##### !explanation

Steering file contents are present in the context window for the full duration of every session, and that context is re-processed on every message. Content that is not universally relevant to every task compounds token costs without providing benefit for sessions where it doesn't apply. Task-specific knowledge is better placed in skills, which are only loaded when a relevant task is encountered.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: P6nT1dJqBzX8mWkYsC4rHvL0a
* title: Best Practices
##### !question

A developer writes a skill for a deployment checklist by documenting the steps from memory before testing it. During the first few runs, the agent repeatedly makes the same mistake at one specific step where a choice needs to be made. What practice would have been most likely to prevent this?

##### !end-question
##### !options

a| Writing a longer skill description so the agent has more context about when to load it.
b| Running a live session where the actual deployment task is performed and correcting the agent's output at each misstep, then building the skill from that session.
c| Adding examples of the desired final outputs in a `references/` subdirectory.
d| Writing a second skill for just the step where the agent has issues.

##### !end-options
##### !answer

b|

##### !end-answer
##### !explanation

The walkthrough approach works through the task with a live agent session and corrects missteps as they happen. Those corrections become steps and gotchas in the skill file. Writing a skill from memory skips this discovery process, which means edge cases and environment-specific failure points, like the one this developer encountered, often don't get documented until they surface during actual use.

Adding examples of desired output in a `references/` subdirectory might help, but it's not a guarantee, depending on how many steps there are and where the AI is failing before producing that final output. Writing a longer skill description or creating a new skill do not address the core issue of the AI failing to correctly predict what it should do next with the given steps.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: G9wMpK2xNqB5rJcYtS7hZnL3v
* title: Best Practices
##### !question

A team wants to ensure their code review agent always runs with read-only tool access and a specific higher-capability model, regardless of who activates it. What is the most reliable way to accomplish this?

##### !end-question
##### !options

a| Add a note to the steering file reminding team members to select the correct model before starting a review session.
b| Create a custom agent file with the model, permitted tools, and a base prompt around code reviewing, and store it in the project repository.
c| Document the correct configuration in the project README so team members can manually apply it each session.
d| Configure the review settings in each team member's personal user preferences.

##### !end-options
##### !answer

b|

##### !end-answer
##### !explanation

A custom agent file encodes model preference and tool access directly into a named configuration that anyone on the team activates by name. Storing it as a project-level agent file means it versions alongside the code, is automatically available to new team members, and any improvements to the configuration apply to everyone. Relying on documentation, README notes, or personal preferences introduces variability and depends on team members remembering to apply settings correctly each time.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->