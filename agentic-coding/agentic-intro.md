# Coding with Agents

Let's start with a perhaps familiar scenario. When we ask an AI tool to help debug a function, we might: 
1. paste in some context like code or an error
2. describe the problem
3. read the response (maybe follow up with questions to ensure understanding or explore a concept)
4. decide if it makes sense and check for accuracy
5. apply the changes 
This flow is useful, but it's very manual; we are the one coordinating each of the steps.

Agentic coding is a different setup. Rather than a single exchange, we configure a system that runs a loop: 
1. an AI model generates an output
2. that output triggers a tool call (reading a file, running a test, writing code)
3. the result comes back
4. the model generates the next output based on the updated context. 

This keeps going, without a human prompting each turn, until the task is done or the system reaches a stopping condition. 
- The underlying engine is still a language model doing text prediction, but the infrastructure around it (tools, structured context, orchestration) enables coding agents to do sustained, multi-step work across a codebase.

In this lesson, we'll build a shared vocabulary for the concepts that make this possible. We'll define what an agent actually is under the hood, introduce the key components of an agentic setup, and clarify what is and is not different about this approach compared to one-off prompting.

## Learning Goals

- Describe the control loop that makes an AI agent capable of multi-step work.
- Explain the role of agents, subagents, skills, and steering files in an agentic coding setup.
- Describe at a high level how MCP and RAG extend what agents can do.
- Identify what changes about the developer's role when moving from one-off prompting to working with agents.

## Vocabulary and Synonyms

| Vocab | Definition | Synonyms | How to Use in a Sentence |
| ----- | ---------- | -------- | ------------------------- |
| Agent | A system that wraps a language model with tools and a control loop, enabling it to take actions, receive results, and produce successive outputs toward a goal. | AI agent, coding agent, autonomous agent | "We configured the agent with access to our file system and test runner so it could work through the failing tests without us intervening at each step." |
| Context Window | The body of text that a language model has the ability to work with at any given point. Models have fixed size context windows and can only generate output based on the current contents of the window. | working memory | "When the context window filled with debugging output, earlier instructions were no longer in scope and the agent's outputs became less consistent." |
| Subagent | An agent instance spawned to handle a specific, isolated subtask. | worker agent, child agent | "The primary agent spawned a subagent for the research phase so its own context window wouldn't be consumed by library documentation." |
| Steering File | A persistent markdown document loaded into context at the start of every agent session. | project instructions, AGENTS.md, CLAUDE.md, agent config | "The steering file specifies which directories contain auto-generated code so agents don't attempt to modify them." |
| Skill | A markdown document that provides an agent with procedural instructions for a specific task. | agent skill, skill file | "Adding a code review skill gave the agent consistent instructions for summarizing pull request feedback in our team's format." |

## LLMs & Agents

To use agents well, it helps to have a clear picture of the technology that coding agents are built around. To that end, let's quickly refresh ourselves on Large Language Models (LLMs)!

### LLM Refresher

A **large language model (LLM)** is a statistical system trained on large amounts of text. When an AI is given some input, that input is added to the session's current context window, and the AI produces an output by predicting what text should come next, based on patterns from training. This is true whether the output is a poem, a SQL query, or a plan for refactoring a service. 
- The model has no awareness of the outside world, no persistent memory between sessions, and no intent. It generates text that is statistically likely given the input.

This is important to keep in mind because it shapes how we should think about the entire system. An agent is not exercising judgment. It is not choosing an approach because it understands our goals. It is producing outputs like tool call specifications, plans, and code, based on what its training and context make statistically probable. 

While these prediction engines are powerful and genuinely useful, it also means:
- Confident-looking outputs can be wrong.
- The model has no awareness of what it shouldn't do unless that's specified in its context.
- Scope, permissions, and review processes are our responsibility, not tasks that the model enforces on its own.

This is why well-designed agentic workflows include:
- **sandboxes** - environments that limit access to files and what tools can actually do
- **guardrails** - rules embedded in the system context to help prevent malicious use
- **human review checkpoints** - moments where a person verifies the output before the next phase proceeds 

In many organizations and production environments these aren't optional; they're required structures teams use to address the gap between statistical prediction and reliable software engineering.

### What Is an AI Agent?

At the core of any AI agent is a **large language model (LLM)**. What transforms a language model into an **agent** is the infrastructure built around it:

- **Structured context**: System instructions, project files, skill documents, and conversation history all live in the context window and shape what the model produces.
- **Tools**: External programs the agent can call — read a file, run a command, invoke an API, execute a test suite. The model's output specifies which tool to call and with what inputs; the tool runs and returns a result.
- **A control loop**: Rather than generating one response and stopping, the agent runs in a cycle. The model produces output, the output triggers a tool call, the tool result is added to the context, and the model generates the next output. This repeats until the configured stopping conditions are met.

This combination is what makes an AI agent capable of multi-step, sustained work. The model itself hasn't changed, it's still doing text prediction, but the architecture around it enables us to use LLM-backed agents to complete complex tasks.

### The Agent Loop

The typical control loop used in agentic systems is called the **ReAct loop** (short for Reasoning and Acting). In practice, it works like this:

1. **Input**: The model receives the current context window: system instructions, task description, tool results from prior steps, loaded files, and any conversation history.
2. **Generate output**: The model produces text. This might be a step-by-step plan, a tool call specification, a code snippet, or a summary of what happened.
3. **Execute tools**: If the output specifies a tool call, the tool runs and the result is appended to the context.
4. **Repeat**: The updated context is fed back in, and the model generates the next output.
5. **Stop**: The loop ends when the model's output matches a stopping condition, for example, it produces an answer with no further tool calls, or a hard limit on iterations is reached.

Most of the behavior we observe in capable coding agents like exploring a codebase, building an implementation plan, running tests, and iterating on failing code, is the result of this loop running many times with well-configured tools and context. The apparent sophistication comes from the loop structure and the tools, not from anything the model is doing beyond predicting the next useful token.

This is also why guardrails, sandboxes, and keeping a human in the loop matter so deeply. Because the loop runs based on statistical outputs rather than human judgment, it can produce incorrect results, take actions outside the intended scope, or get stuck in unproductive cycles (an AI that gets stuck in a loop can use up a vast amount of tokens). 
- The infrastructure we build around agents: what tools they can access, what permissions they have, when humans review their work, are how we manage the risks.

## The Components of an Agentic Coding Setup

When we work with coding agents, we're configuring a system to work in. We'll touch on the core building blocks below, then dive deeper into all of these topics in the coming lessons!

### Agents and Subagents

In a simple workflow, a single agent handles an entire task. In more complex ones, a **primary agent**, also called an **orchestrator**, manages the overall goals and plan, then delegates portions of the work to **subagents**.

A subagent is a regular agent instance with its own context window, that is given a focused set of instructions for a specific piece of work. The subagent runs through the ReAct loop on that task, produces a result, and terminates. The primary agent receives only the output; the subagent's full context stays private. 

We can think of agents and subagents like functions and helper functions. 
- A helper function is still an ordinary function; it's just handling a smaller piece of the overall problem and returning the result back to the larger function. 
- Similarly, a subagent is an agent instance just like a primary agent. It gets spun up to handle a small piece of the overall task and returns the result to the primary agent to continue the work. 

The subagent pattern is valuable for two main reasons: 
- it keeps the primary agent's context window from being consumed by content that doesn't need to stay in scope for the full duration of the overall goal
- it allows multiple subtasks to run in parallel since multiple subagents can run simultaneously on independent tasks

### Skills

Large language models have broad general knowledge but no built-in awareness of how work is done by a specific organization or project. A **skill** is how we provide that procedural knowledge in a reusable form.

A skill is typically a markdown file containing:
- A **name and description** that tell the system what the skill covers and when it applies
- **Instructions** in the form of step-by-step guidance for the task (a code review format, a migration scaffolding process, a documentation standard, etc.)

Since skills are text files that can be source controlled, they are modular and reusable! The same skill can be shared across different agents, different projects, and updated independently.

Agents need access to skills, but at the same time, not every agent needs every skill for every task. If we were to add the instructions for every skill to every agent at launch, we'd burn through tokens and fill up context windows very quickly.

To help address these issues, skills use a tiered loading approach: 
1. At startup, agents load a lightweight index that contains only the name and description of each configured skill. 
2. When an agent encounters a task that matches up with the description of a skill, the full instructions for the skill are added to the context window at that time. 

This means an agent can have access to a large library of skills without paying the full context cost of all of them upfront. It is worth noting that the "lightweight index" of skills is still a source of context window and token usage that grows with the number of available skills. 

### Steering Files

A **steering file** is a persistent markdown document loaded into the context window at the start of every session. It is meant to give agents project-level context that they have no other way of knowing.

The contents of steering files will vary greatly based on the requirements of the project, team, or organization, but they typically include content like:
- Explicit instructions or prohibitions that apply to every workflow (e.g., "Start each feature in a new branch")
- Team or organization-specific coding conventions and style standards
- Common patterns or shortcuts the team uses that are applicable to every project or task

A steering file is how we provide the persistent context that models cannot retain on their own between sessions. Without a steering file, agents fall back on training data patterns, which often means default conventions rather than team-specific ones. 

### MCP and RAG: Extending Capabilities and Knowledge

Two additional mechanisms significantly expand what an agent can do. Model Context Protocol (MCP) and Retrieval-Augmented Generation (RAG) solve two distinct problems that both stem from the same root issue: AI models, on their own, are isolated. They only know what's in their training data and their current context window. We'll cover each further in a later lesson, but here's the high-level distinction:

**MCP (Model Context Protocol)** is an open standard for connecting agents to external tools and services. MCP helps solve the problem of reach: A model can't natively call an API, query a database, or interact with an external service. When an MCP server is configured, the agent can call out to live systems and receive real data back as part of its context. Similar to skills, there is a token and context tradeoff since every MCP server's tool definitions are loaded into the context window upfront.

**RAG (Retrieval-Augmented Generation)** is a technique for making large knowledge bases available to agents without loading them entirely upfront. RAG solves the problem of stale or missing factual knowledge. A model's training data has a cutoff and can't include your private codebase, internal docs, or recent information. RAG lets an agent pull in relevant chunks of text from an external knowledge base at runtime. The knowledge is stored externally; when a query arrives, a retrieval system identifies the most relevant chunks and injects them into the context window. 

In short: MCP expands what an agent can *do*, RAG expands what an agent can *know*.

## What Changes About Our Role?

When we work with agents rather than one-off prompting, the shape of our involvement changes, but the need for it doesn't:

| Prompt-Based Assistance | Agentic Coding |
|-------------------------|----------------|
| Single turn: we ask, it answers | Multi-step loop: the agent takes actions and produces successive outputs |
| We manage context by what we paste in | Context is managed through persistent files, skills, and retrieval systems |
| We evaluate and apply each output | The agent executes changes directly, requiring us to review them |
| We direct every next step | The loop generates successive outputs; we create the overall workflow and structure then review at key checkpoints |
| Each session starts from scratch | Steering files and skills provide continuity across sessions |
| Stakes of each exchange are low | The agent can take many actions; scope and permissions matter more |

Because the agent loop runs on statistical text prediction rather than human judgment, our oversight remains essential. We remain responsible for:
- Reviewing plans before the agent executes changes
- Checking outputs before accepting them into the codebase
- Configuring appropriate sandboxes and tool permissions
- Steering the agent when it produces off-target outputs

The goal isn't automation that replaces engineering judgment. It's a workflow where the agent handles a broader scope of mechanical work between our checkpoints, while we focus on the decisions that require context and judgment the agent doesn't have.

## Summary

Moving from prompt-based AI assistance to agentic coding means shifting from single-turn exchanges to a configured system that runs on its own between human checkpoints. At the core of that system is still a language model doing text prediction.

The **ReAct** loop is what makes agents capable of sustained, multi-step work. Tool results get appended to the context window, which feeds the model's next output, which can trigger the next tool call, and so on. The more capable and well-configured the tools, the more useful the loop becomes. Because the loop runs on statistical prediction rather than judgment, it requires guardrails, sandboxing, and meaningful human review to make it safe to use in real environments.

The components of an agentic setup each address a specific part of making this system reliable and manageable: 
- **Skills** give agents procedural knowledge for specific tasks without loading everything upfront, using tiered loading to control context costs. 
- **Steering files** give agents the project-level context they can't retain on their own between sessions. 
- **Subagents** keep the primary agent's context clean by handling isolated subtasks in their own windows and returning only what's needed. 
- **MCP** connects agents to live external services, and **RAG** makes large knowledge stores accessible at runtime without loading them all at once.

Our role in this system is to design it well, configure the right guardrails, and stay engaged at the right moments. The agent extends our capacity for mechanical work; we retain responsibility for the judgment calls: defining requirements, reviewing plans before execution, checking outputs before they're committed, and steering when results drift from our intent.

## Check for Understanding

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: d57bbc0f-85a5-42cd-ab41-593a399c07b9
* title: Coding with Agents
##### !question

Which of the following most accurately describes what a language model is doing when it produces a code change as part of an agent session?

##### !end-question
##### !options

a| It analyzes the problem using software engineering principles to determine the most correct solution.
b| It executes the code internally before outputting it to verify the change works.
c| It searches its training data for matching examples from similar codebases.
d| It generates text by predicting what should follow the current context window, based on patterns from its training data.

##### !end-options
##### !answer

d|

##### !end-answer
##### !explanation

A language model is a statistical text prediction system. Given the current contents of the context window, it produces the text most statistically likely to follow, whether that's a plan, a code change, or a tool call specification. It does not apply software engineering reasoning in the human sense, execute code internally to check its work, or search its training data at runtime. This is what motivates the need for sandboxes, guardrails, and human review in agentic workflows.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: 7bd1db0e-d998-47e5-8faf-7844a580ecf6
* title: Coding with Agents
##### !question

A primary agent is working on a large feature. To research how a third-party library handles rate limits, it spawns a subagent, gives it a focused task, and receives a summary back before continuing with its implementation plan. What is the main benefit of this pattern?

##### !end-question
##### !options

a| It allows the research to run on a more capable model than the primary agent uses.
b| It prevents the primary agent's context window from being consumed by the research work, and returns only the relevant result.
c| It ensures the research results are verified by a human before the primary agent uses them.
d| It reduces the total number of tool calls made during the session.

##### !end-options
##### !answer

b|

##### !end-answer
##### !explanation

Subagents run in their own isolated context windows and return only their output to the primary agent. Their internal work (all the tool calls, intermediate outputs, and library documentation they loaded) stays private and doesn't persist in the primary agent's context. This keeps the primary agent's context window focused on the overall task and prevents it from filling up with content that was only relevant to the research subtask. Subagents can also run in parallel on independent tasks, which is a secondary benefit of the pattern.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: f4b0916f-cf3a-4f7e-b9e5-bef598cd086f
* title: Coding with Agents
##### !question

A team configures their coding agent with 40 skill files covering a wide range of tasks. They're concerned about the impact on token usage. Which aspect of how skills work helps address this concern?

##### !end-question
##### !options

a| Skills are stored in a compressed format that reduces their token footprint.
b| The agent automatically removes skills from the context window once a task using them is complete.
c| At startup, agents load only the name and description of each skill; full instructions are only added to the context when a matching task is identified.
d| Skills are cached outside the context window and do not affect token usage at all.

##### !end-options
##### !answer

c|

##### !end-answer
##### !explanation

Skills use a tiered loading approach to manage context costs. At startup, the agent loads a lightweight index containing only the name and description of each configured skill. The full instructions for a skill are only loaded into the context window when the agent encounters a task that matches that skill's description. This allows a large library of skills to be accessible without paying the full context cost of all of them upfront. It's still worth noting that the index itself grows with the number of available skills, so having many skills does carry some ongoing context cost.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: 981eef1a-982e-490a-9892-582509c71ee5
* title: Coding with Agents
##### !question

A developer moves from using a chat-based AI assistant to a fully configured coding agent. Which of the following most accurately describes how their role changes?

##### !end-question
##### !options

a| They shift from directing every individual step to designing the overall workflow, configuring guardrails, and reviewing plans and work at key checkpoints.
b| Their role is largely eliminated; the agent handles planning, execution, and review without needing input.
c| They become responsible for writing the agent's tool implementations rather than reviewing its outputs.
d| Their role stays essentially the same; the agent just handles formatting and boilerplate tasks.

##### !end-options
##### !answer

a|

##### !end-answer
##### !explanation

Agentic coding changes the nature of developer involvement, not whether it's needed. With a chat-based assistant, the developer directs every step manually. With a coding agent, the loop handles successive steps between human checkpoints, but the developer is still responsible for designing the overall workflow, configuring appropriate permissions and sandboxes, defining requirements, reviewing and approving plans before implementation, and checking outputs before they're committed. Accountability for actions AI takes under our supervision always remains with the developer.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

