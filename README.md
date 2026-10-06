# Project 8 — AI Red Team Operations

### Anyaugo Space-Tech Ltd. · Security Training | Lab Evidence

**A hands-on red-team case study: take the *hardened* bank-style AI assistant from Project 7, attack it as a knowledgeable adversary, find what still gets past the guardrails, fix one weakness, and prove the fix.** Every finding is dual-mapped to the **OWASP Top 10 for LLM Applications** and **MITRE ATLAS**. All work was done in an isolated VirtualBox lab using only synthetic data and a fully local model — no client systems, no real data, no live internet targets.

---

## The business question
> The assistant from Project 7 was hardened with layered guardrails. **What does a determined attacker still get through — and how would a bank find, fix, and prove the fix?**

## The verified result
**In the controlled lab, three weaknesses still got through the guarded build** — a roleplay/persona **jailbreak** that hijacks the assistant's output, **indirect injection** through a single poisoned knowledge-base document, and **unauthenticated cross-customer data access** through a connected tool. All three were independently corroborated with Microsoft's **PyRIT** red-team framework. Two controls were confirmed to **hold** (model-initiated email is scope-blocked; the output canary-filter catches secret leaks, though it is brittle). One finding — the jailbreak — was then **fixed at the input screen and the fix proven by retest** (the roleplay prompt moved from *allowed* to *blocked*, with no effect on normal customer questions). The other two are documented as **open**, with recommended fixes.

## The report
**[Project 8 — AI Red Team Operations — Unified Evidence Portfolio (PDF)](Test-08-Final-Report-and-Client-Workflow/Project%208%20-%20AI%20Red%20Team%20Operations%20-%20Unified%20Evidence%20Portfolio.pdf)** — the full report: the cover, each attack with its business impact, the fix-and-retest, the OWASP × MITRE ATLAS mapping, and the evidence limits. Start with the cover and the Findings sections.

---

## How to read the evidence
Each stage is a `Test-NN` folder with self-describing screenshots; every finding stage also has a plain-language `Test-NN-finding-notes.md` written record.

| Test | Stage | What it establishes |
|---|---|---|
| [01](Test-01-Scope-and-Authorization/) | Scope & Authorization | The red-team rules of engagement and lab boundary, fixed before testing |
| [02](Test-02-Target-Confirmation-and-Reconnaissance/) | Target Confirmation & Reconnaissance | The guarded Project 7 build is live and relabeled; known overt attacks are blocked |
| [03](Test-03-Prompt-Layer-Jailbreak-and-Override/) | Prompt-Layer Jailbreak / Override | **P08-F1** — a roleplay/persona jailbreak slips past the input screen and hijacks the output |
| [04](Test-04-Context-Layer-Indirect-Injection/) | Context-Layer Indirect Injection | **P08-F2** — one poisoned document hijacks the answer to an innocent customer's question |
| [05](Test-05-Application-Layer-Tool-Abuse/) | Application-Layer Tool Abuse | **P08-F3** — cross-customer data pulled with no authentication; the email tool's scope holds |
| [06](Test-06-Multi-Turn-Orchestrated-Attack/) | Multi-Turn Orchestrated Attack (PyRIT) | Microsoft's PyRIT reproduces the findings in a single run |
| [07](Test-07-Fix-and-Retest/) | Fix-then-Retest | P08-F1 fixed at the input screen and the fix proven (no regression on benign traffic) |
| [08](Test-08-Final-Report-and-Client-Workflow/) | Final Report & Client Workflow | The unified PDF deliverable |
| [09](Test-09-Privacy-and-AI-Use/) | Privacy & AI Use | Responsible handling of evidence and AI-assisted work (standalone record page) |

## Findings at a glance (business impact)
| # | Finding | Severity | If this were a real bank | Status |
|---|---|---|---|---|
| P08-F1 | Roleplay jailbreak hijacks the assistant's output | High | The AI is made to emit attacker-chosen content — a phishing link, false financial advice, offensive text — under the bank's name | **Fixed** |
| P08-F2 | Indirect injection via a poisoned document | High | Innocent customers are fed attacker-controlled misinformation, with no malicious input from them and nothing they ever see | **Open** |
| P08-F3 | Unauthenticated cross-customer data access | High | Anyone can retrieve any customer's name, email, account number, and balance by simply *claiming* to be them | **Open** |

> **Honesty note on P08-F2:** the injection is proven at the model level (the model relays the planted falsehood verbatim). On the *live* guarded path the tested payloads were *incidentally* blocked by the canary filter — a brittle side effect, not a defence against the injection class. A canary-avoiding payload would reach the customer. The PDF carries the full nuance.

## Controls that held
- **`send_email` scope** — a model-initiated send was attempted and **blocked** (`model-initiated email disabled by guardrails`). Exfiltration via the email tool is prevented.
- **Output canary-filter** — catches the planted secret when the model leaks it. It works, but it is a **narrow, brittle backstop** (it only knows the one marker string), which is itself a finding.

---

## The lab
The Project 7 lab, reused deliberately (the premise is *the same app, now attacked*). Two VirtualBox machines on an isolated internal network (`P07-IR-Lab`, `10.77.7.0/24`):
- **Kali** attacker — `10.77.7.10` (all attacks originate here).
- **Ubuntu** target — `10.77.7.20`, running the **guarded** self-built assistant (`Ollama · llama3.2:3b · RAG · tools · input screen · output filter`) behind a support API.

The protected information the assistant must guard is a single planted marker, **`P07-CANARY-7F3A9C`** — a stand-in for real secrets, not a real credential. All customer records are invented.

## Tool stack (free / open-source — the real industry stack)
| Used here | Commercial equivalents |
|---|---|
| **PyRIT** (Microsoft) — red-team orchestration | Lakera Red, HiddenLayer |
| `curl` / `jq`, custom Python attack scripts | — |
| **OWASP Top 10 for LLM Applications**, **MITRE ATLAS** (reporting / mapping frameworks) | — |

## Boundary & honesty note
This is a **controlled training-lab portfolio**, not a client assessment or a legal opinion. No client or company authorization is claimed; all work was done inside the agency's own lab on invented data, attacking a target the agency built and owns. Observed lab results are kept separate from the **proposed** client workflow, and recommendations are marked as proposed practice, not completed work. Where the model's output is non-deterministic, findings are written around the invariant (e.g. the output was attacker-controlled), and incidental effects (such as the canary filter masking a reply) are identified and set aside rather than claimed as defences. See the PDF's *Assurance Boundaries* section for exactly what is and is not established.

---

*References: OWASP Top 10 for LLM Applications · MITRE ATLAS · NIST SP 800-115 · NIST AI 600-1. Prepared by Anyaugo Space-Tech Ltd., Security Training track.*

*© 2026 Anyaugo Space-Tech Ltd. Licensed under [CC BY-NC-ND 4.0](LICENSE) — share with attribution; no commercial use or derivatives.*
