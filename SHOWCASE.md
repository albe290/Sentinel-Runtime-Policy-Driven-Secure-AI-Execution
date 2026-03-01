# 🏆 Safe Agent Skeleton: The Ultimate Showcase Guide

Welcome to the **Safe Agent Skeleton**. If you are looking for a standard "Hello World" LangChain tutorial, you are in the wrong place. 

This project is a **Policy-Driven, Adversarially-Tested, Universal AI Runtime**. It is designed to act as a secure "cage" or "referee" to test arbitrary LLM agents against dangerous capabilities without compromising the host machine.

---

## 🏗️ The Architecture: "Safe by Design"

Traditional AI applications trust the LLM to make decisions. This architecture **never trusts the LLM**. 

1. **The Rulebook (`policy.yaml`)**: Security logic is not buried in Python `if` statements. We use a Declarative Security Authority. You can change the entire risk posture of the agent instantly without touching code.
2. **The Risk Engine (`risk.py`)**: Every action the agent attempts is evaluated mathematically.
    - Base Tool Risk + Network Exposure + Data Sensitivity + Step Pressure = **Total Risk Score**.
3. **The Human-in-the-Loop (`runner.py`)**: Based on dynamic `policy.yaml` thresholds (`allow`, `review`, `block`), execution is either instant, paused for human oversight, or immediately terminated.
4. **The ASOC Dashboard (`dashboard/app.py`)**: Real-time telemetry parsing the immutable `audit.jsonl` to provide visual proof of governance (Total Blocks, Avg Risk Score, MITRE Mitigations).

---

## ⚔️ The Adversarial Playground (MITRE ATLAS)

We don't just *claim* the agent is safe; we prove it. The project includes a built-in **Red Team Harness** (`redteam/harness.py`) that attacks the Risk Engine using industry-standard MITRE ATLAS techniques.

By giving the agent access to high-risk simulated tools (`shell_exec`, `delete_file`), we run automated stress tests:

*   **AML.T0054 (Prompt Injection)**: STOPPED 🛑 (Static Guardrail Detection)
*   **AML.T0053 (DNS Exfiltration)**: STOPPED 🛑 (Deep Payload Inspection patching a Phase 2 breach)
*   **AML.T0051 (Cloud Metadata SSRF)**: STOPPED 🛑 (Network Isolation Guardrail)
*   **AML.T0042 (Ransomware Simulation)**: BLOCKED 🛑 (Risk Engine Threshold Exceeded)
*   **AML.T0040 (Lateral Movement)**: BLOCKED 🛑 (Risk Engine Threshold Exceeded)

---

## 🚀 How to Run the Showcase

Want to see an Enterprise-Grade AI Security Operations Center in action?

Step 1: **Start the Telemetry Dashboard** (in Terminal A)
```powershell
python -m dashboard.app
```
*Watch the "Rich" UI boot up and monitor `audit.jsonl` in real-time.*

Step 2: **Run the AI Red Team Harness** (in Terminal B)
```powershell
python -m redteam.harness
```
*Watch the Red Team script launch adversarial payloads against the Risk Engine, and look at Terminal A as the Dashboard immediately registers the "Blocks" and updates the MITRE ATLAS mitigation status.*

Step 3: **Examine the Rulebook**
Open `policy/policy.yaml`. Lower the `block` threshold from `90` to `30`. Run the harness again. Watch how the entire system's behavior changes without compiling a single line of code.

---

## 🧠 Why This Project Matters

As LLMs become more capable, the barrier to building AI features goes down. The barrier to building *Safe* AI features goes up.

The **Safe Agent Skeleton** demonstrates operational maturity. It proves that before we give autonomous software the keys to the kingdom, we must build the **Auditable, Policy-Driven infrastructure** required to keep it on a leash.
