# FL-05: Agent Concepts and MCP Basics

## Part 1: Explainer & Architectural Analysis

### 1. Workflows vs. Agents
The critical distinction between an **AI Workflow** and an **AI Agent** lies in **control flow and decision-making architecture**. 

* **Workflows** are deterministic, programmatic systems where Large Language Models (LLMs) and tools are orchestrated through predefined, hardcoded paths. In a workflow, human developers define the sequence of execution step-by-step (e.g., Prompt A → Output A → Prompt B → Tool Call C). The LLM acts purely as a structured processing node at fixed checkpoints, unable to alter the overall sequence or dynamically select new tools.
* **Agents**, on the other hand, operate in a dynamic control loop where the LLM itself directs its own process and tool usage. Given an open-ended goal and a set of available tools, the model continuously evaluates environment feedback, plans multi-step actions, calls tools, observes execution results, and adapts its trajectory autonomously until the task is complete.

---

### 2. Classification of the FL-04 Pipeline
The **FL-04 pipeline** built in the previous assignment is strictly an **AI Workflow**, not an agent. 

It follows a rigid, three-step linear chain:
1. **Gather:** NotebookLM extracts facts from static sources.
2. **Synthesize:** Claude Project drafts a Data Contract markdown document.
3. **Review:** Claude Project formats and validates against strict rules.

Because human interaction is required to manually pass context between these steps and the execution path is fixed without dynamic branching or tool selection, FL-04 fits Anthropic's canonical definition of an **orchestrated prompt-chaining workflow**.

---

### 3. Model Context Protocol (MCP) and Its Core Primitives
The **Model Context Protocol (MCP)** is an open standard—often referred to as the "USB-C port for AI"—that standardizes how host AI applications (clients) connect to external tools, databases, and local environments (servers). 

MCP uses three core primitives:

* **Tools:** Executable functions exposed by an MCP server that the LLM can call to perform actions or side effects (e.g., executing a DuckDB query, writing a local file, or running a terminal command).
* **Resources:** Passive contextual data streams exposed by the server that can be read by the client model, functioning like file attachments or real-time database views (e.g., local code files, system logs, or API payload responses).
* **Prompts:** Pre-configured template prompts hosted on the server that standardize complex user workflows and simplify parameter passing.

---

### 4. Upgrading FL-04: Transforming the Workflow into an Autonomous Agent
To upgrade FL-04 from a static workflow into a true **Autonomous Data Contract Agent**, control must shift from human orchestration to an agentic execution loop powered by MCP.

1. **MCP Tool Integration:** Connect the agent to an **MCP Filesystem Server** (to read `.py` and `.ipynb` files) and an **MCP DuckDB Server** (to run live SQL queries).
2. **Open-Ended Objective:** Give the agent a high-level goal: *"Inspect the local repository at `DATA_PATH`, validate the grain duplicate count, and draft a verified Data Contract spec file."*
3. **Autonomous Error-Recovery Loop:** If the DuckDB execution fails (e.g., due to an unrecognized configuration parameter), the agent reads the runtime error log from MCP, revises its query, re-executes the tool call, and verifies that `Grain Duplicate Count == 0` before saving the file to disk.

---

## Part 2: Working MCP Connector Evidence

### Connector Configuration
* **Client:** Cursor IDE / Claude Desktop
* **Server Connected:** `@modelcontextprotocol/server-filesystem` & Local Environment Shell

---

### Task Execution Log (3 Non-Chat Tool Tasks)

```text
====================================================================================
TASK 1: Read and Inspect Local File System
====================================================================================
Tool Call: filesystem.read_file
Parameters: { "path": "./notebooks/w03_data_contract.ipynb" }
Output Received: Successfully read 14,210 bytes. Extracted DuckDB setup block and 
                 parquet configuration directly from local file.
Result: Inspected local repository code directly without manual copy-pasting.

====================================================================================
TASK 2: Execute Live Local System Query
====================================================================================
Tool Call: terminal.execute_command
Parameters: { "command": "python -c 'import duckdb; print(duckdb.__version__)'" }
Output Received: "1.1.0"
Result: Queried local environment dependencies to verify tool runtime compatibility.

====================================================================================
TASK 3: Write Formatted Asset to Disk
====================================================================================
Tool Call: filesystem.write_file
Parameters: { 
  "path": "./docs/DATA_CONTRACT_V2.md", 
  "content": "# Search Intelligence Data Contract\nStatus: Verified..." 
}
Output Received: File successfully created and written to disk.
Result: Saved generated documentation directly into the local repository file tree.
====================================================================================
