# Agentic Coding Problem Set

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
* Human review is most valuable before the agent executes on a plan and before agent-generated changes are accepted into the codebase.
* Human review should happen at every individual loop iteration to ensure each tool call is appropriate.
* Human review is no longer necessary once the steering file and skills are configured correctly.

##### !end-options
##### !answer

* Human review is most valuable before the agent executes on a plan and before agent-generated changes are accepted into the codebase.

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

* Create a custom agent with the 12-step process as the agent's system prompt.
* Add it to the steering file so it is loaded into every session by default.
* Create a skill file for the scaffolding process, which the agent loads when a relevant task is identified.
* Configure a RAG system to retrieve the process from the team's internal wiki on demand.

##### !end-options
##### !answer

* Create a skill file for the scaffolding process, which the agent loads when a relevant task is identified.

##### !end-answer
##### !explanation

A skill file is the right component for providing procedural knowledge for a specific task type. The skill's description acts as a trigger: when the agent encounters a task that matches, it loads the full instructions. This keeps the information modular, reusable, and out of the context window until it's actually needed. A steering file is better suited for persistent project-wide context that applies to every session, not task-specific procedures. RAG is for large document stores.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: WdKkIKffPFELO9dXA1IPX7MKvp
* title: Agentic Coding Problem Set
##### !question

Which of the following is re-sent to the model on every turn of an agent session, making it a compounding cost over the course of a long session?

##### !end-question
##### !options

a| Only the most recent message and response pair.
b| The entire conversation history, including the system prompt, steering configuration, tool definitions, and all prior messages.
c| The system prompt and steering configuration only; prior messages are cached and not re-processed.
d| Only the messages the user sent; agent-generated responses are excluded from re-processing.

##### !end-options
##### !answer

b|

##### !end-answer
##### !explanation

On every turn of an agent session, the model receives the full contents of the context window: the system prompt, steering configuration, tool and skill definitions, and every prior message and response in the conversation. This means content added at the start of a session is not a one-time cost. It is re-processed with every subsequent exchange. A large steering file or a verbose set of tool definitions adds overhead on every single turn, not just when it was first loaded.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: gntpHcOjKZXlTP4YHRv1Vjucnz
* title: Agentic Coding Problem Set
##### !question

A team's primary agent needs to explore a large library of documentation to answer a design question. The team is concerned about how this research will affect their main session's context window. What is the most effective approach?

##### !end-question
##### !options

a| Have the primary agent read all documentation files sequentially and store the full contents in a running notes file.
b| Ask the agent to summarize the documentation in chat before reading any files, then read the ones the summary identifies as relevant.
c| Delegate the research task to a subagent, which works in its own isolated context and returns only a summary of relevant findings to the primary session.
d| Disconnect all MCP servers before the research phase to free up token space for the documentation.

##### !end-options
##### !answer

c|

##### !end-answer
##### !explanation

Research and file exploration tasks are among the fastest ways to fill a context window, because every file read enters the window in full. Delegating this work to a subagent means all of that exploration happens in a separate, isolated context. The subagent returns only a compact summary of what it found, which is all that enters the primary session's context window. This keeps the main session lean for the implementation work that follows. It is also a good illustration of why building strong single-session context habits first matters: the discipline of keeping context focused applies to subagent sessions as well.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: Whrt6FNV82JQikj3c6BaT4dghR
* title: Agentic Coding Problem Set
##### !question

At approximately what context window usage level is it recommended to take proactive action to manage context, and why?

##### !end-question
##### !options

a| At 100%, because automatic compaction handles everything before that point and no manual action is needed.
b| At 95%, because that is when most tools trigger automatic compaction and a manual intervention should happen at the same time.
c| Around 60%, because behavioral degradation can begin well before the window is full, and acting early preserves output quality.
d| At 25%, because any more than a quarter of the window filled indicates the session is being used inefficiently.

##### !end-options
##### !answer

c|

##### !end-answer
##### !explanation

Context window degradation does not wait for the window to hit its ceiling. Research and practitioner experience consistently show that model outputs become less reliable as the window fills, with content in the middle of a long session receiving less effective attention. Most tools begin automatic compaction somewhere between 80 and 95 percent capacity, but degradation is often already underway by that point. Checking context usage regularly and taking action around the 60 percent mark, whether that means compacting, starting a fresh session, or restructuring the remaining work, gives us the best chance of maintaining consistent output quality throughout a session.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: TfloAaWkoVd85BV3TCS9m3Ki3E
* title: Agentic Coding Problem Set
##### !question

A team is running a long implementation session with an AI coding agent. After about an hour, a developer checks in and notices the agent has been attempting the same fix repeatedly without making progress. The tests are still failing and the agent is on its twelfth iteration.

What is the most appropriate response to this situation?

##### !end-question
##### !options

a| Let the agent continue iterating, since agents always resolve issues if given enough attempts.
b| Restart the entire project from scratch, since a stuck agent indicates the original plan was invalid.
c| Interrupt the session to reorient the agent, since cycling on a failing approach continues to consume tokens without making progress.
d| Increase the context window limit so the agent has more capacity to work through the problem.

##### !end-options
##### !answer

c|

##### !end-answer
##### !explanation

When an agent is stuck in a loop, continuing to iterate does not resolve the underlying issue and burns through tokens with no return. This is a situation that calls for human intervention: stopping the session, assessing what went wrong, and providing specific direction before resuming. Agents do not self-correct from a bad approach just by being given more attempts. Restarting the entire project is not warranted; the plan itself may still be sound, and the fix may be a targeted adjustment. Expanding the context window does not resolve an agent cycling on a flawed approach.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: vYpJlhATJ0Kw4D58rbGadHY4NP
* title: Agentic Coding Problem Set
##### !question

A team needs to refactor a large codebase. The work can be split into four independent modules, each of which can be refactored without needing information from the others during execution. The team is concerned a single agent session will fill its context window before completing all four modules.

Which workflow pattern is best suited to this situation?

##### !end-question
##### !options

a| Adversarial review, where a second agent session evaluates the output of the first before any code is committed.
b| Fan-out, where an orchestrating agent delegates each module to a separate subagent running in its own context window.
c| A single extended session with compaction enabled, so the agent can work through all four modules sequentially without interruption.
d| Human-in-the-loop review at the end of the full refactor, where a developer approves all changes after the agent finishes.

##### !end-options
##### !answer

b|

##### !end-answer
##### !explanation

Fan-out is designed for exactly this situation: work that can be decomposed into independent units that do not need to share information during execution. Each subagent operates in a clean context window focused on a single module, which keeps each session manageable and produces more consistent output than a single session juggling all four modules at once. Adversarial review addresses verification quality, not context window capacity. A single session with compaction may still hit limits and does not take advantage of parallelism. Human review at the end is a good practice but does not address the context window constraint during implementation.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

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

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: k4Rp9mXvL2nQwZ8dY3bJcT6fH
* title: Agentic Coding Problem Set
##### !question

A development team is configuring an agent to run an autonomous data migration task overnight. The environment includes cloud credentials stored in local environment variables. Which sandboxing approach is most appropriate?

##### !end-question
##### !options

a| Application-level permission controls set to "ask" for file writes
b| OS-level file permissions that forbid reading any `.env` files
c| Container isolation with the project directory mounted and network access restricted
d| No sandbox is needed because the agent will only be working with CSV files

##### !end-options
##### !answer

c|

##### !end-answer
##### !explanation

Container isolation is the appropriate choice when an agent will run autonomously for extended periods in an environment with sensitive credentials. Mounting only the project directory prevents the agent from accessing environment variables or credential files outside the container's scope. Application-level controls and OS-level tools provide weaker guarantees for long autonomous runs, and the presence of credentials in the environment is a specific risk container isolation is designed to address.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
### !challenge
* type: checkbox
* id: H2bXcF9mKpQrZ4nVjL7wYgT0s
* title: Agentic Coding Problem Set
##### !question

Select all of the scenarios below where we should create a skill rather than add to a steering file.

##### !end-question
##### !options

a| Step-by-step instructions for generating API endpoint documentation in the team's required format
b| A detailed guide for running the database migration workflow specific to the project's ORM
c| Organization or team-specific branch naming conventions 
d| Configuration templates for the deployment pipeline used in production releases
e| Project-specific workflow requirements around branching and testing applicable to all new features 

##### !end-options
##### !answer

a|
b|
d|

##### !end-answer
##### !explanation

Steering files should contain content that applies universally across every session and every task type. Branch naming conventions and protected file rules are relevant no matter what an agent is working on. Workflow guides, documentation formats, and deployment configurations are task-specific and belong in skills, where they are loaded on demand rather than consuming context window space in every session.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: W5nGqT9rBxM0kJaHpL4dZcF2y
* title: Agentic Coding Problem Set
##### !question

A team is building a new feature and plans to use AI agents throughout the process. During the planning phase, they want to analyze the existing codebase and draft an architecture document. During implementation, a separate agent will write code from that plan. Which model configuration best fits this workflow?

##### !end-question
##### !options

a| Use the smallest available model for planning, since planning tasks are shorter than implementation tasks.
b| Use the same model for both phases to keep output consistent and avoid configuration overhead.
c| Use a higher-capability model for planning and a mid-tier model for implementation, since errors in the plan are more expensive to fix than errors in routine code generation.
d| Use a higher-capability model for implementation and a lighter model for planning, since writing code requires more processing than drafting a document.

##### !end-options
##### !answer

c|

##### !end-answer
##### !explanation

Planning and architecture work benefits from higher-capability models because errors at this stage compound into implementation problems that are costly to fix later. Once a well-specified plan exists, routine code generation tasks can often be handled reliably by a mid-tier model at lower cost. This approach matches model capability to where it has the most impact rather than applying the same model uniformly.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: 6GE0x0FW84vza8M0pM5x2uRE
* title: Adding Outside Knowledge: MCP & RAG
##### !question

A team connects five MCP servers to their agent "just in case" they're needed, even though most sessions only use tools from one of them. What is the most accurate description of the cost this creates?

##### !end-question
##### !options

a| No added cost, since unused tools are automatically removed from context
b| A one-time cost paid only the first time each server is connected
c| A steady token cost across every message in the session, since all connected tool definitions load into context at the start
d| A per-query cost that only applies when a tool from one of the servers is actually called

##### !end-options
##### !answer

c|

##### !end-answer
##### !explanation

Every tool definition from every connected server gets loaded into the context window at the start of a session and stays there for the duration, whether or not it's used. Connecting several servers "just in case" means paying that token cost on every single message in the session, not just once or only when a tool is actually called. This is different from RAG, where cost is tied to the number of chunks retrieved per query.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: wExkv5ahmovSSJgLP4L22bVX
* title: Adding Outside Knowledge: MCP & RAG
##### !question

How does the security boundary of a remote MCP server typically differ from that of a local MCP server?

##### !end-question
##### !options

a| A remote server's boundary is defined by our account permissions on the connected service; a local server's boundary is defined by our user permissions or arguments we give the server when starting it.
b| A remote server has no security boundary at all, since it runs outside our machine
c| A local server's boundary is defined by OAuth scopes, while a remote server's boundary is defined by folder paths
d| Both use the exact same boundary, since MCP is a single standardized protocol

##### !end-options
##### !answer

a|

##### !end-answer
##### !explanation

A local server runs with our own machine's permissions, so what it can touch is scoped by configuration details like which folders it was given access to. A remote server acts through an authenticated session, so what it can touch is defined by whatever access our account has been granted on that service, commonly established through OAuth or an API key rather than through local configuration.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: wuQo1d0TUqHwgS0cYEiOeY9T
* title: Adding Outside Knowledge: MCP & RAG
##### !question

Before a RAG system can answer any questions, source material has to go through a setup phase. Which sequence correctly describes that phase?

##### !end-question
##### !options

a| The query is converted to an embedding, then compared against tool schemas, then injected into the context window
b| Source material is broken into chunks, each chunk is converted into an embedding, and those embeddings are stored in a vector database
c| Source material is sent directly to the model, which memorizes it permanently for future sessions
d| An MCP server is connected first, and it automatically generates embeddings for any file it can access

##### !end-options
##### !answer

b|

##### !end-answer
##### !explanation

Building a RAG knowledge base happens ahead of time and involves three steps: splitting source material into smaller chunks, converting each chunk into an embedding that captures its meaning, and storing those embeddings in a vector database for fast comparison later. Retrieval and injection happen afterward, at the time a query is actually made, not during this setup phase.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->
