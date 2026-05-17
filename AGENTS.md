# 🤖 Agent Testing Documentation (agents.md)

This document defines the testing protocols, environment configurations, and validation cases for AI agents within the **OpenCode** ecosystem.

---

## 🧪 Testing Environment
To ensure security and prevent unintended side effects on the production codebase, all agent evaluations must be executed within a **sandboxed container**.

* **Runtime:** Docker (using the `opencode-testing-env` image).
* **Network Isolation:** Outbound traffic is disabled by default.
* **Resource Constraints:** * **CPU:** 1 Core (Maximum)
    * **RAM:** 2GB (Maximum)

---

## 📋 Test Cases

| Test ID | Agent Component | Description | Expected Outcome |
| :--- | :--- | :--- | :--- |
| **TC-01** | `CodeGenerator` | Generate a Python sorting function via prompt. | Code passes unit tests and PEP8 linting. |
| **TC-02** | `SecurityAgent` | Scan a provided snippet for SQL Injection. | Agent identifies the flaw and suggests parameterized queries. |
| **TC-03** | `RefactorAgent` | Convert nested loops into list comprehensions. | Optimized code with 100% functional parity. |
| **TC-04** | `DocAgent` | Generate documentation for an existing class. | Complete Google-style docstrings generated for all methods. |

---

## 📏 Performance Metrics

We evaluate agent efficiency using the **Agent Success Rate (ASR)** formula:

$$ASR = \left( \frac{S_{pass}}{T_{total}} \right) \times 100\%$$

Where:
* $S_{pass}$ = Number of successfully completed tasks.
* $T_{total}$ = Total number of test attempts.

---

## ⚠️ Security Guardrails

> [!WARNING]
> Agents are strictly forbidden from modifying files outside the designated `/workspace/sandbox/` directory.

1.  **Restricted Commands:** Usage of `rm -rf`, `mkfs`, or `dd` will trigger an immediate process termination.
2.  **Human-in-the-Loop (HITL):** Any system-level permission changes (`chmod`, `chown`) require manual approval.
3.  **Execution Timeout:** Maximum execution time per task is capped at **180 seconds**.

---

## 🚀 Running the Tests 

To initiate the test suite for a specific agent, execute the following command in your terminal:

```bash
python scripts/run_agent_tests.py --agent-id "generator-v1" --verbose


