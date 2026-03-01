# Sentinel-Runtime

A secure, enterprise-grade runtime for autonomous agents with policy gating, auditable automation, and human-in-the-loop controls.

## Core Features

- **Policy Enforcement**: Centralized authority (`policy.yaml`) to define allowed tools, domain allowlists, and dynamic risk weights.
- **MITRE ATLAS Attack Lab**: Built-in AI Red Team harness (`redteam/harness.py`) to simulate adversarial payloads (Prompt Injection, DNS Exfiltration, Ransomware) against the guardrails.
- **Risk Engine**: A sophisticated scoring model that calculates risk dynamically based on tool weights, network exposure, data sensitivity, and execution scope.
- **Security Telemetry (ASOC)**: A "Rich" terminal dashboard (`dashboard/app.py`) providing real-time observability of execution metrics and threat mitigations parsed from immutable audit logs.
- **Manual Approval (HIL)**: The runner identifies high-risk tools or actions (guided by the policy thresholds) and pauses execution for manual user approval.
- **Granular Tool Permissions**: High-risk OS capabilities (`shell_exec`, `delete_file`) constrained by the Risk Engine and domain allowlists for networking tools.
- **Structured Audit Logs**: JSONL logging (`logs/audit.jsonl`) with automatic redaction of sensitive credentials.

## Project Structure

```text
├── agent/       # "Dumb" planners that generate AgentPlan steps
├── dashboard/   # Real-time Telemetry Visualizer (ASOC)
├── validator/   # Security guardrails, deep DNS inspection, and Policy logic
├── runner/      # Execution engine with HIL and mock FS/OS tools
├── redteam/     # MITRE ATLAS Attack Lab and Adversarial Payloads
├── logs/        # Immutable JSONL audit trails
├── policy/      # Declarative Security Authority (policy.yaml)
├── contracts/   # JSON Schema definitions for plans and results
├── tests/       # Unit and integration test suites (18 tests)
└── main.py      # Entry point demonstrating secure execution
```

## Getting Started

1. **Activate Environment**:
   ```powershell
   .\.venv\Scripts\Activate.ps1
   ```

2. **Run Demo**:
   ```powershell
   python main.py
   ```
   *Note: This will pause and request approval for the HTTP step.*

   <img src="docs/main.py.png" alt="Main Agent Execution" />

3. **Run ASOC Dashboard**:
   ```powershell
   python -m dashboard.app
   ```
   *Note: Run this in a separate terminal to watch real-time telemetry.*

4. **Run Red Team Attack Lab**:
   ```powershell
   python -m redteam.harness
   ```

5. **Run Security Tests**:
   ```powershell
   pytest -v
   ```
   *(Verifies 18/18 core security gates).*

   <img src="docs/Security%20tests%20.png" alt="Security Tests Passing" />

## Security Design

This project follows the **Safe by Design** principle: security is enforced at the runtime boundary, not inside the agent's logic. Even if an agent is compromised or hallucinates a malicious step (e.g., SSRF, prompt injection, or attempting an unapproved tool), the `Validator` and `Runner` will block the action before it hits the network or OS.

## Showcase: The Power of Policy-Driven Security

The true power of this architecture is the Declarative Security Authority (`policy.yaml`). You can fundamentally change the agent's risk posture without touching Python code.

**State 1: "Trust but Verify" (Block Threshold: 90)**
In this default state, the agent has room to operate. The Risk Engine relies on specific guardrails (like Domain Allowlists or Regex patterns) to intercept known bad payloads:

<img src="docs/Block%20Threshold%2090%20image%201.png" alt="Trust but Verify Terminal Phase" />
<br>
<img src="docs/Block%20Threshold%2090%20image%202.png" alt="Trust but Verify Dashboard Phase" />

```text
[ATTACK] Testing: supply_chain_poison
Mitigation: Host 'trusted-source.com' not in allowlist
```

**State 2: "Zero-Trust Lockdown" (Block Threshold: 30)**
By simply lowering the `block` number in `policy.yaml` to `30`, the entire system enters a lockdown state. The Risk Engine becomes so strict that it kills actions mathematically, *before* static guardrails even evaluate them:

<img src="docs/Zero%20Trust%20Lockdown%20Block%20Threshold%2030.png" alt="Zero-Trust Lockdown Terminal Phase" />
<br>
<img src="docs/Zero%20Trust%20Lockdown%20Block%20Threshold%2030%20image%202.png" alt="Zero-Trust Lockdown Dashboard Phase" />

```text
[ATTACK] Testing: supply_chain_poison
Mitigation: Risk BLOCKED (55): policy_base_risk:30; network_access:+15; internal_data:+10
```
*The architecture intercepted the adversarial payload purely because the context became too dangerous.*

### Live Environment Previews
<img src="docs/dashboard_execution.png" alt="Dashboard Execution Tracking" />
<br>
<img src="docs/redteam_execution.png" alt="Red Team Attack Lab Defense" />


## AI Security Architecture: Attack → Defense Mapping

As part of the threat modeling for this runtime, we map known AI failure modes and attacks to specific defensive layers implemented in this project.

| Attack Vector / Failure Mode | Risk Profile | Defense Layer (Sentinel-Runtime) | Mitigation Strategy |
| :--- | :--- | :--- | :--- |
| **Prompt Injection** / Instruction Override | Agent executes unintended commands | `Validator` (Input Pre-processing) | Integrity checks on inputs; rejecting known adversarial prompts before reaching the LLM. |
| **Jailbreaking** / Role Manipulation | Agent bypasses policy or impersonates admin | `Policy Engine` (Execution Bounds) | Hard authority bounds. The LLM is never the authority; the `Policy` object dictates allowed actions. |
| **Data Exfiltration** / Leaking Secrets | Sensitive data sent to external servers | `Runner` (Action/Output Filter) | Granular tool constraints (e.g., domain allowlists, HTTP response size limits) and output filtering. |
| **Autonomous Runaway** | Agent recursively calls APIs at scale | `Runner` / `Risk Engine` | `ALLOW`, `REVIEW`, `BLOCK` action gating. High-risk actions strictly require Human-in-the-Loop (HIL) approval. |
| **Server-Side Request Forgery (SSRF)** | Agent scans or attacks internal network | `Policy Engine` | Strict domain allowlists on HTTP tool. Internal IPs/localhost are blocked by definition unless explicitly allowed. |
| **Sensitive Data Exposure** in Logs | PII or API Keys leaked in audit trails | `Audit Logger` | Automatic redaction of credentials and secure JSONL structured logging. |

> *Note: This table is actively updated as new adversarial patterns are researched and mitigated.*
