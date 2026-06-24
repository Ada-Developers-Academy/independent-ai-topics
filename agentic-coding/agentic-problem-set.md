# Agentic Coding Problem Set

## Intro 

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: 78d4c53f-65d4-4fdd-83e9-15a0ae170576
* title: Agentic Coding Problem Set
##### !question

A team is planning an agentic coding workflow and wants to know where human review is most important. Based on what we know about how the ReAct loop works, which answer best reflects where human involvement remains critical?

##### !end-question
##### !options

* Human review is only needed when the agent reports an error, since successful-looking outputs can be assumed to be correct.
* Human review is most valuable before the agent executes on a plan and before agent-generated changes are accepted into the codebase, because confident-looking outputs can still be incorrect.
* Human review should happen at every individual loop iteration to ensure each tool call is appropriate.
* Human review is no longer necessary once the steering file and skills are configured correctly.

##### !end-options
##### !answer

* Human review is most valuable before the agent executes on a plan and before agent-generated changes are accepted into the codebase, because confident-looking outputs can still be incorrect.

##### !end-answer
##### !explanation

Because the loop produces outputs based on statistical text prediction rather than judgment, confident-looking results can be wrong or out of scope. The two highest-leverage points for human review are: before the agent executes on a plan (catching problems early, before changes are made) and before accepting outputs into the codebase (ensuring the work is correct and appropriate). Reviewing every loop iteration would eliminate the productivity benefit of agents, and good configuration reduces but doesn't eliminate the need for review.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: 2d8aaaac-510a-4345-b826-c51e92ec1f76
* title: Agentic Coding Problem Set
##### !question

A language model running as a coding agent has no persistent memory between sessions. Without additional infrastructure, it begins each session with no knowledge of the project it's working on. Which component is specifically designed to solve this problem?

##### !end-question
##### !options

* A subagent, which can be configured to carry context from prior sessions.
* A skill file, which stores the project's procedures between sessions.
* A steering file, which is loaded into the context window at the start of every session to provide foundational project context.
* RAG, which retrieves prior session transcripts and injects them into context.

##### !end-options
##### !answer

* A steering file, which is loaded into the context window at the start of every session to provide foundational project context.

##### !end-answer
##### !explanation

A steering file is a persistent markdown document loaded at the start of every session. It provides the agent with project-level context — architecture, conventions, constraints, goals — that the model cannot retain on its own between sessions. Skill files cover procedural knowledge for specific task types, not persistent project context. Subagents operate in isolated contexts. RAG retrieves from external knowledge stores but isn't the mechanism for delivering project-level session context.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: eb8334f6-cb0c-4776-b798-98c8f3311703
* title: Agentic Coding Problem Set
##### !question

An agent is configured with access to a large internal knowledge base — hundreds of documents covering architecture decision records, API references, and service runbooks. Loading all of this into the context window at startup would exceed token limits. Which mechanism is designed to handle this scenario?

##### !end-question
##### !options

* Skills, because they load full content on demand when a matching task is identified.
* MCP, because it allows agents to call external services that can serve the documents.
* RAG, because it stores the content externally and retrieves only the most relevant portions into context at query time.
* A steering file, because it provides a summary of the knowledge base for every session.

##### !end-options
##### !answer

* RAG, because it stores the content externally and retrieves only the most relevant portions into context at query time.

##### !end-answer
##### !explanation

RAG (Retrieval-Augmented Generation) is specifically designed for this use case: making large knowledge stores accessible to agents without loading them entirely into the context window. Content is stored externally; a retrieval step identifies the most relevant chunks for the current query and injects only those into context. Skills are for procedural task instructions, not large document stores. MCP connects agents to live external services and APIs but isn't a document retrieval system. A steering file provides persistent session context, not on-demand document retrieval.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: 98a02df3-85a6-4424-bfbc-769ee70205c9
* title: Agentic Coding Problem Set
##### !question

A team wants their coding agent to follow their team's 12-step process for scaffolding a new service, including specific file naming conventions and required boilerplate. This process is not something a general-purpose language model would know. What is the most appropriate way to provide this knowledge to the agent?

##### !end-question
##### !options

* Include the 12-step process as a tool the agent can execute.
* Add it to the steering file so it is loaded into every session by default.
* Create a skill file for the scaffolding process, which the agent loads when a relevant task is identified.
* Configure a RAG system to retrieve the process from the team's internal wiki on demand.

##### !end-options
##### !answer

* Create a skill file for the scaffolding process, which the agent loads when a relevant task is identified.

##### !end-answer
##### !explanation

A skill file is the right component for providing procedural knowledge for a specific task type. The skill's description acts as a trigger: when the agent encounters a task that matches, it loads the full instructions. This keeps the information modular, reusable, and out of the context window until it's actually needed. A steering file is better suited for persistent project-wide context that applies to every session, not task-specific procedures. RAG is for large document stores; a 12-step process is better maintained as a skill.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

## Context Windows

## Recommended Workflows

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: 2Wj5Ds9Lq0Nt7Mc4Xr1Ak8Fv3Yz6Bp
* title: Agentic Coding Problem Set
##### !question

A team runs adversarial review on a newly implemented authentication module. Three critic sessions evaluate architectural conformance, security, and edge case coverage. The security critic returns a list of specific violations. What happens next in the workflow?

##### !end-question
##### !options

a| The findings are discarded because no single critic can produce a definitive result without consensus from all three.
b| The findings are passed to the implementation phase as a concrete list of issues to address, then the review runs again.
c| The entire implementation is discarded and the planning phase restarts from scratch.
d| A fourth critic session is automatically spawned to verify the first three critics' findings.

##### !end-options
##### !answer

b|

##### !end-answer
##### !explanation

The adversarial review pattern creates a structured loop: critic sessions return specific, actionable findings, those findings go back to implementation for fixes, and then the review runs again. This continues until the critics pass or a defined iteration threshold is reached. The goal is targeted remediation, not wholesale restart. Discarding findings from a single dimension would undermine the purpose of running focused critics, and requiring consensus across all critics before acting on any finding would delay fixes unnecessarily.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->