# Intro 

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: c4e69c28-6d60-4201-8460-18730bd2d1b7
* title: Coding with Agents
##### !question

Which of the following most accurately describes what a language model is doing when it produces a plan or a code change as part of an agent session?

##### !end-question
##### !options

* It is applying logical reasoning to analyze the problem and determine the correct solution.
* It is searching its training data for examples from similar projects.
* It is generating text by predicting what should follow the current context window, based on patterns from training.
* It is executing the plan internally before outputting it to verify the result is correct.

##### !end-options
##### !answer

* It is generating text by predicting what should follow the current context window, based on patterns from training.

##### !end-answer
##### !explanation

A language model is a statistical text prediction system. Given a context window, it produces the text most statistically likely to follow — including plans, code, and tool call specifications. It does not apply logical reasoning in the human sense, search training data at runtime, or execute code internally to verify outputs. Understanding this is important for knowing why guardrails, sandboxing, and human review are necessary components of a well-designed agentic workflow.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: 78d4c53f-65d4-4fdd-83e9-15a0ae170576
* title: Coding with Agents
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
* title: Coding with Agents
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
* title: Coding with Agents
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
* id: e0aa3896-bd9b-4f45-90f8-177d5ea8da88
* title: Coding with Agents
##### !question

A team sets up a coding agent with access to their file system and test runner. The agent reads several source files, generates a code change, runs the tests, reads the failure output, and generates a revised change — all without the team prompting each step. What makes this multi-step behavior possible?

##### !end-question
##### !options

* The language model has been trained to understand the team's specific codebase.
* A control loop appends tool results back to the context window, and the model generates successive outputs based on the updated context.
* The steering file contains a step-by-step script that the agent follows for every task.
* The agent stores the results of each step in long-term memory between sessions.

##### !end-options
##### !answer

* A control loop appends tool results back to the context window, and the model generates successive outputs based on the updated context.

##### !end-answer
##### !explanation

What makes multi-step agentic behavior possible is the ReAct loop: the model generates output that specifies a tool call, the tool runs and returns a result, that result is appended to the context window, and the model then generates the next output. This cycle repeats without requiring a new human prompt at each step. The model itself doesn't have inherent knowledge of the codebase; it works from what's in the context window at each iteration.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: 4f7d8180-683c-4b75-a667-6b47561dca9a
* title: Coding with Agents
##### !question

Why is it important to configure sandboxes, guardrails, and human review checkpoints when working with coding agents?

##### !end-question
##### !options

* Agents are known to produce malicious outputs without these restrictions in place.
* The control loop runs based on statistical model outputs rather than human judgment, so confident-looking results can still be incorrect or out of scope without these safeguards.
* Sandboxes prevent the agent from consuming too many tokens during a session.
* Human review checkpoints allow the agent to ask clarifying questions before proceeding.

##### !end-options
##### !answer

* The control loop runs based on statistical model outputs rather than human judgment, so confident-looking results can still be incorrect or out of scope without these safeguards.

##### !end-answer
##### !explanation

The agent loop is driven by a language model doing statistical text prediction — it produces outputs that look confident and coherent regardless of whether they are correct. Without guardrails, tool permission limits, and human review, the agent can take unintended actions or produce plausible-looking but wrong results. These safeguards are not about agent intent; they're how we manage the gap between statistical pattern matching and the judgment calls our codebases actually require.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: 98a02df3-85a6-4424-bfbc-769ee70205c9
* title: Coding with Agents
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

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: 7da86d44-e4d8-4d47-b639-2c5bb8e93c3b
* title: Coding with Agents
##### !question

Which of the following most accurately describes what a large language model is doing when it operates as the core of a coding agent?

##### !end-question
##### !options

* It reasons through the problem using logic and understanding of software architecture.
* It produces text outputs by predicting what text should follow its current context window, based on patterns from training.
* It executes code and uses the results to update its internal knowledge about the codebase.
* It searches its training data for solutions that have been used in similar projects.

##### !end-options
##### !answer

* It produces text outputs by predicting what text should follow its current context window, based on patterns from training.

##### !end-answer
##### !explanation

A large language model is a statistical system that generates outputs by predicting what text should follow a given input, based on patterns learned during training. It does not reason in the human sense, execute code directly, or search its training data at runtime. The capabilities we associate with capable coding agents — exploring files, running tests, iterating on failures — come from the control loop and tool infrastructure built around the model, not from reasoning or understanding inside the model itself.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->