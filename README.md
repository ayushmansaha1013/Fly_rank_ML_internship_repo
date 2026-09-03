# Search-Intel-Scout
>VIDEO LINK: https://youtu.be/V3si_I3IC4Y
> **Autonomous Search Intelligence Research & Data Contract Validation Agent**
>WEBSITE LINK: https://deft-marshmallow-4c3163.netlify.app/

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/)
[![DuckDB](https://img.shields.io/badge/DuckDB-1.1.0-orange.svg)](https://duckdb.org/)
[![MCP Compliant](https://img.shields.io/badge/MCP-Filesystem%20%7C%20Shell-green.svg)](https://modelcontextprotocol.io/)

---

## 1. What It Does & Who It's For

`Search-Intel-Scout` is an autonomous AI research and data contract validation agent designed for **Search Intelligence and Machine Learning Engineers**. 

* **The Problem:** ML engineers spend hours manually inspecting newly ingested `.parquet` panel datasets, running validation queries in SQL consoles to verify row counts, checking for composite primary key duplicates, and drafting markdown data contracts.
* **The Solution:** The agent bridges natural language requests directly to local database systems inside Google Antigravity IDE. Connected via the **Model Context Protocol (MCP)**, the agent autonomously inspects code notebooks, executes live DuckDB SQL integrity checks, intercepts database runtime errors, and generates peer-review-ready Markdown Data Contracts in `./docs/`.

---

## 2. AI Transparency Statement (Framework Compliance)

> **How this was built:** This agent was developed using **Google Antigravity IDE** with AI pair-programming support. AI assisted in generating initial boilerplate bindings for the Model Context Protocol (MCP) server integration and parsing regex logic. 
> 
> **What I manually verified and wrote myself:** I personally wrote the core DuckDB SQL grain verification queries, debugged database catalog exception errors when handling raw Parquet binaries, configured local file permission boundaries, and built the 5-part evaluation test suite.

---

## 3. Architecture & Data Flow

```text
+-------------------------------------------------------------------+
|                           USER PROMPT                             |
|  "Inspect March 2026 panel, verify grain duplicates, publish doc" |
+-------------------------------------------------------------------+
                                  |
                                  v
+-------------------------------------------------------------------+
|                        CLAUDE / GEMINI AGENT LOOP                 |
|  1. Formulates execution plan & selects tool                      |
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
