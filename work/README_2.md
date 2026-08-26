# Search-Intel-Scout (FL-09 Documentation)

> **Autonomous Search Intelligence Research & Data Contract Validation Agent**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/)
[![DuckDB](https://img.shields.io/badge/DuckDB-1.1.0-orange.svg)](https://duckdb.org/)
[![MCP Compliant](https://img.shields.io/badge/MCP-Filesystem%20%7C%20Shell-green.svg)](https://modelcontextprotocol.io/)

---

## 1. Overview & Core Mission

`Search-Intel-Scout` is an autonomous research and validation agent designed for **Search Intelligence and Machine Learning Engineers**. 

### The Problem
Data engineers spend hours manually inspecting newly ingested `.parquet` panel data, running validation queries in SQL consoles to verify row counts, checking for duplicate composite keys, and updating markdown data contracts.

### The Solution
`Search-Intel-Scout` bridges open-ended natural language requests to local database systems. Powered by Claude and connected via the **Model Context Protocol (MCP)**, the agent autonomously inspects local notebooks, executes live DuckDB SQL integrity checks, catches catalog errors, and generates peer-review-ready Markdown Data Contracts in `./docs/`.

---

## 2. Demo Video

🎥 **Watch the 4-Minute Unlisted Walkthrough:** [https://youtu.be/Unlisted_Search_Intel_Scout_Demo](https://youtu.be/Unlisted_Search_Intel_Scout_Demo)

### Video Timestamps & Agenda
* **`0:00 - 0:45`** — System Architecture & Problem Statement
* **`0:45 - 2:15`** — **Live End-to-End Execution:** Agent reading notebook, executing live DuckDB query, and publishing contract document.
* **`2:15 - 3:15`** — **Design Decision:** Why we chose local MCP over remote API gateways to prevent data secret leakage.
* **`3:15 - 4:00`** — **Guardrail & Known Limitation:** Intercepting DuckDB catalog errors and enforcing zero-write rules on raw `.parquet` binaries.

---

## 3. System Architecture & Data Flow

```text
+-------------------------------------------------------------------+
|                           USER PROMPT                             |
|  "Inspect March 2026 panel, verify grain duplicates, publish doc" |
+-------------------------------------------------------------------+
                                  |
                                  v
+-------------------------------------------------------------------+
|                        CLAUDE AGENT LOOP                          |
|  1. Formulates plan & selects tool                                |
|  2. Intercepts runtime errors & adapts queries                    |
+-------------------------------------------------------------------+
       /                                                     \
      / Tool Call: filesystem.read_file                       \ Tool Call: terminal.execute
     v                                                         v
+-----------------------------------+             +-----------------------------------+
|   MCP FILESYSTEM SERVER           |             |   LOCAL DUCKDB SHELL RUNNER       |
|   `@modelcontextprotocol/server-` |             |   Executes: `SELECT COUNT(*)`     |
|   `filesystem`                    |             |   `read_parquet('march_2026')`    |
+-----------------------------------+             +-----------------------------------+
     |                                                         |
     v                                                         v
 Reads `./notebooks/w03_contract.py`             Returns: `DUPLICATES_FOUND: 0`
     \                                                         /
      \_________________________   ___________________________/
                                \ /
                                 v
+-------------------------------------------------------------------+
|                     FINAL AUTONOMOUS OUTPUT                       |
|        File Created: `./docs/DATA_CONTRACT_V2.md`                 |
+-------------------------------------------------------------------+