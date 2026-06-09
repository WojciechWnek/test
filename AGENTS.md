# 🤖 Agent Testing Documentation (agents.md)

This document defines the testing protocols, environment configurations, validation cases, and continuous integration standards for AI agents within the **OpenCode** ecosystem.

---

## ⚙️ Prerequisites

Before initiating any testing procedures, ensure your local or remote machine meets the following requirements:

* **Docker Desktop / Daemon:** Version 24.0.0 or higher.
* **Python:** Version 3.10+.
* **API Keys:** Valid LLM provider keys set in the `.env.testing` file (e.g., `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`).
* **Mock Server:** Access to the local OpenCode Mock API running on `localhost:8080`.

---

## 🧪 Testing Environment

To ensure security and prevent unintended side effects on the production codebase, all agent evaluations must be executed within a **sandboxed container**.

* **Runtime:** Docker (using the `opencode-testing-env:latest` image).
* **Network Isolation:** Outbound traffic is disabled by default. Whitelisted domains (e.g., specific LLM endpoints) are routed through a managed proxy.
* **Resource Constraints:** * **CPU:** 1 Core (Maximum)
* **RAM:** 2GB (Maximum)
* **Disk I/O:** Limited to 50MB/s


* **State Management:** The sandbox is ephemeral. All state is wiped completely between individual test executions to prevent data leakage.

---

## 📋 Test Cases

The test suite covers functional, security, and edge-case scenarios.

| Test ID | Agent Component | Category | Description | Expected Outcome |
| --- | --- | --- | --- | --- |
| **TC-01** | `CodeGenerator` | Functional | Generate a Python sorting function via prompt. | Code passes unit tests and PEP8 linting. |
| **TC-02** | `SecurityAgent` | Security | Scan a provided snippet for SQL Injection. | Agent identifies the flaw and suggests parameterized queries. |
| **TC-03** | `RefactorAgent` | Optimization | Convert nested loops into list comprehensions. | Optimized code with 100% functional parity. |
| **TC-04** | `DocAgent` | Documentation | Generate documentation for an existing class. | Complete Google-style docstrings generated for all methods. |
| **TC-05** | `CodeGenerator` | Edge Case | Provide an ambiguous or contradictory prompt. | Agent asks clarifying questions instead of guessing. |
| **TC-06** | `All Agents` | Context Limit | Inject a 50,000-token codebase for analysis. | Agent gracefully summarizes or requests chunking without crashing. |
| **TC-07** | `SecurityAgent` | Hallucination | Ask to patch a non-existent CVE in a standard library. | Agent correctly identifies the CVE as fake and refuses the patch. |

---

## 📏 Performance Metrics

We evaluate agent efficiency and cost-effectiveness using three primary metrics:

**1. Agent Success Rate (ASR)**
Measures the reliability of the agent completing its assigned tasks.

$$ASR = \left( \frac{S_{pass}}{T_{total}} \right) \times 100\%$$

Where:

* $S_{pass}$ = Number of successfully completed tasks.
* $T_{total}$ = Total number of test attempts.

**2. Token Efficiency Ratio (TER)**
Measures how concisely the agent achieves the goal, penalizing overly verbose chain-of-thought loops.

$$TER = \frac{Tokens_{optimal}}{Tokens_{actual}}$$

**3. Average Latency (AL)**
Measured in seconds from the dispatch of the prompt to the receipt of the final executable artifact. Tests exceeding $AL > 45s$ are flagged for review.

---

## ⚠️ Security & Privacy Guardrails

> [!WARNING]
> Agents are strictly forbidden from modifying files outside the designated `/workspace/sandbox/` directory.

1. **Restricted Commands:** Usage of `rm -rf`, `mkfs`, `dd`, or any shell reverse-tcp payloads will trigger an immediate process termination and alert the security team.
2. **Human-in-the-Loop (HITL):** Any system-level permission changes (`chmod`, `chown`) or external API calls require manual approval via the CLI prompt.
3. **Execution Timeout:** Maximum execution time per task is strictly capped at **180 seconds**.
4. **PII/Secrets Redaction:** All prompts passing into the agent must go through the `SecretScanner` middleware. Hardcoded passwords or API keys will result in an immediate `TC-FAIL`.

---

## 🚀 Running the Tests

### Local Execution

To initiate the test suite for a specific agent, execute the following command in your terminal. Use the `--report` flag to generate a detailed HTML output.

```bash
python scripts/run_agent_tests.py --agent-id "generator-v1" --verbose --report ./artifacts/report.html

```

### Running the Full Suite

To run tests across all agents in parallel (requires minimum 8GB RAM):

```bash
make test-agents-all

```

---

## 🔄 CI/CD Integration (GitHub Actions)

Agent tests are automatically integrated into our CI pipeline. They are triggered on:

* Pull Requests targeting the `main` or `develop` branches.
* Modifications to any file within the `opencode/agents/` directory.

**Workflow File:** `.github/workflows/agent-evaluation.yml`

*Note: If the ASR falls below 90% during a PR check, the pipeline will fail and block the merge.*
