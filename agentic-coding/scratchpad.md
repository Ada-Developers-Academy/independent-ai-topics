# Scratchpad

## Intro lesson

This window includes system instructions, conversation history, loaded files, and tool outputs.

| ReAct Loop | The control loop used in most agentic systems: the model generates output, tools execute based on that output, results are appended to the context, and the model generates the next output. Repeats until a stopping condition is met. | agent loop, reasoning-and-acting loop | "Each iteration of the ReAct loop added new tool results to the context, so the agent's later outputs reflected the state of the codebase at each step." |
| MCP (Model Context Protocol) | An open standard for connecting agents to external tools and services — APIs, databases, file systems, and more — as callable functions. | tool protocol, agent tool integration | "Adding an MCP connection to our internal documentation system let the agent look up service specs without us pasting them in." |
| RAG (Retrieval-Augmented Generation) | A technique for making large external knowledge bases available to agents by retrieving only the most relevant content and injecting it into the context at query time. | retrieval augmentation | "RAG let us give the agent access to thousands of pages of internal documentation without overloading the context window." |
MCP (Model Context Protocol)A standard for connecting agents to external tools and services — such as databases, APIs, or search engines — as callable functions.tool protocol, agent tool integration"With an MCP connection to our issue tracker, the agent could pull ticket details directly rather than us having to paste them in."
RAG (Retrieval-Augmented Generation)A technique where relevant content is retrieved from an external knowledge base and injected into the context window at query time, allowing large document stores to be accessed without loading them entirely upfront.retrieval augmentation"We used RAG to give the agent access to our internal API documentation without loading the entire knowledge base into every session."


Managing the context window of our agent sessions is an important part of agentic coding flows, and subagents are a key tool for that management. We'll dive much deeper on this in an upcoming lesson, but subagents keep the primary agent's context window from being consumed by work that doesn't need to stay in scope. 
- Think about a task like exploring and documenting a code base. File contents need to be added to the context window to be available for processing, but once a summary that maps the code base has been created, is it helpful to keep working in a session whose context window if full of . 

## Context Windows & usage




