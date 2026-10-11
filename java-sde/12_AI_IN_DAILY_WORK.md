# 12 — "How Do You Use AI in Your Day-to-Day Work?"

## ⭐ Fundamentals First (start here)
**Time:** 1-month plan **1 h** · 2-month plan **2 h** (see [README](README.md)).

**Do this**
1. Read the **90-second answer framework** (§1) and the **use-cases table** (§2) — 20 min
2. Write **3 STAR stories from your own real usage** (§3: productivity, debugging, quality-guardrail) with numbers — 30–40 min
3. Memorize your **responsible-use rules** (§4): approved tools only, no secrets/PII, verify everything, you own what you commit — 10 min
4. (2-month) Rehearse the follow-up table (§5) aloud — 1 h

**Done when:** you deliver the answer in ~90 seconds with 3 concrete use-cases, 1 safeguard and 1 metric — and answer "tell me about a time AI was wrong" honestly.
**Skip if short on time:** agentic/MCP discussion, team-rollout plan. Never invent experience — use real examples.

> Increasingly asked of senior engineers (often by hiring managers). They are testing: **Are you effective with modern tools? Do you retain judgment, security discipline and ownership?**
> Not "do you use AI" but **"how, where, with what safeguards, with what measurable impact."**

## How to Prepare (Method)
1. **Inventory (30 min):** list the real tasks where you used AI in the last 1–2 months (coding, debugging, tests, docs, design, learning, code review).
2. **Pick 4–5 concrete examples** and write them as STAR stories with *numbers* (time saved, bugs found, coverage gained) and *what you verified*.
3. **Add guardrails:** prepare a crisp answer on security/privacy, hallucination risk, review discipline, licensing, and company policy.
4. **Practice a 90-second answer** + a 3-minute deep-dive; prepare follow-up answers (§5).
5. **If you've used little AI yet:** spend a week deliberately using it in the ways below (on non-confidential or permitted work) so your answers are genuine. Never fabricate experience.
Time: 1 h (1-month) / 2 h (2-month).

---

## 1. Answer Framework (90 seconds)
**Structure: Where → How → Safeguards → Impact → Limits**

> "I use AI assistants as a **pair-programmer and accelerator**, not an autopilot.
> **Where:** (1) understanding unfamiliar code/legacy modules, (2) generating boilerplate, tests and migration scripts, (3) debugging — pasting sanitized stack traces/logs to get hypotheses, (4) design brainstorming and trade-off checks, (5) docs/PR descriptions, (6) learning new tech faster.
> **How:** I give context and constraints, ask for a plan first, work iteratively in small steps, and ask for tests alongside code.
> **Safeguards:** I never paste secrets/PII/proprietary code into unapproved tools; I **review every line** as if from a junior; run tests, linters, Sonar/Snyk; verify APIs against docs because models hallucinate.
> **Impact:** e.g., unit-test authoring time dropped ~50%, root-causing a prod issue went from 3h to 1h, onboarding to a new service from days to hours.
> **Limits:** I don't delegate architecture decisions, security-sensitive logic, or anything I can't explain and defend. Accountability stays with me."

## 2. Use-Cases by SDLC Phase (with prompt patterns)

| Phase | How AI helps | Example prompt pattern | What you must verify |
|---|---|---|---|
| **Understand code** | Explain modules, trace flows, summarize PRs | "Explain how request X flows from controller to DB in this module; list classes involved and gotchas." | Cross-check with actual code/debugger |
| **Design** | Brainstorm options, trade-offs, failure modes, ADR drafts | "Compare Outbox+Debezium vs transactional polling for our Postgres→Kafka; list failure modes and ops cost." | Validate against your constraints/NFRs; you decide |
| **Coding** | Boilerplate (DTOs, mappers, REST clients), refactors, idiomatic Java, regex, SQL | "Refactor this method to streams without changing behavior; keep null-safety; show tests." | Behavior parity, perf, readability, no hallucinated APIs |
| **Testing** | JUnit/Mockito tests, edge cases, parametrized data, test data builders | "Write JUnit5+Mockito tests for `OrderService.place` covering payment failure, timeout, duplicate request." | Tests actually fail when code breaks (mutation check); no over-mocking |
| **Debugging** | Hypotheses from stack traces/logs/thread dumps/GC logs, query plan analysis | "Here's a sanitized thread dump summary; which threads are blocked and why?" | Reproduce; confirm root cause with evidence |
| **Code review** | Pre-review own changes, spot smells/security issues, suggest naming | "Review this diff for concurrency bugs, N+1 queries, and security issues." | Don't blindly accept; combine with human review |
| **DB/SQL** | Query drafts, index suggestions, explain plan reading | "Given this EXPLAIN ANALYZE, why seq scan? propose index." | Run on realistic data; measure |
| **DevOps/Infra** | Dockerfiles, K8s manifests, CI snippets, runbooks | "Generate a hardened Dockerfile for a Spring Boot 3 layered jar." | Security scanning, least privilege, test in staging |
| **Docs/communication** | READMEs, PR descriptions, release notes, runbooks, interview/standup summaries | "Draft PR description from this diff: context, changes, risks, test plan." | Accuracy; remove fluff |
| **Learning** | Explain concepts with examples, quiz me, compare tech | "Quiz me on Kafka rebalancing with 10 questions; grade my answers." | Confirm with official docs |
| **Migration/modernization** | Java 8→17/21, `javax`→`jakarta`, JUnit4→5, Spring Boot upgrades | "List breaking changes upgrading Boot 2.7→3.2 for this pom and propose steps." | Compile + tests + staged rollout |

## 3. STAR Story Templates (fill with your real data)

**Story A — Productivity: tests & legacy**
- **S:** Legacy `PricingService` (~800 LOC) with no tests, blocking a feature change.
- **T:** Add safe change with confidence in 2 days.
- **A:** Used AI to explain flows, generate **characterization tests**, propose refactor steps; I reviewed/modified tests, added edge cases, ran mutation tests; refactored in small commits with AI suggesting extractions.
- **R:** Coverage 0→78%, change shipped in 2 days, no regressions; team adopted the approach.

**Story B — Debugging**
- **S:** Intermittent latency spikes in prod service.
- **T:** Find root cause fast.
- **A:** Collected metrics, GC logs and thread dump (sanitized); used AI to form hypotheses (lock contention vs GC); validated with JFR; root cause = synchronized block around remote call; replaced with async + bounded pool.
- **R:** p99 reduced 1.8s → 300ms; investigation time cut from est. 1 day to ~3 hrs. Documented runbook.

**Story C — Design acceleration**
- **S:** Needed event-driven integration between Order & Inventory.
- **A:** Used AI to enumerate patterns/failure modes; drafted ADR; I challenged assumptions with team's constraints; chose Outbox + idempotent consumers.
- **R:** Design doc in a day vs several; fewer review iterations.

**Story D — Quality guardrail (shows judgment)**
- **S:** AI-suggested code used a non-existent method / unsafe SQL concatenation.
- **A:** Caught via compile + Snyk/Sonar + review; added checklist & prompt guidelines for the team.
- **R:** Prevented a SQL-injection-prone merge; established team review practice for AI-assisted code.

**Story E — Team enablement**
- Created shared prompt templates, a PR checklist for AI-assisted code, lunch-and-learn; measured review-cycle time and defect rate.

## 4. Responsible Use (Be Explicit — This Is a Senior Signal)
- [ ] **Data privacy/confidentiality:** no secrets, credentials, customer/PII data, or proprietary code in **unapproved** tools; use **company-approved** assistants only; sanitize logs; follow data-classification policy.
- [ ] **Policy & licensing:** follow company AI policy; be aware of license/IP implications of generated code; no copying large blocks blindly.
- [ ] **Verification:** compile, run tests, lint, SAST/SCA (Snyk/Sonar), check docs; **treat output as untrusted draft**; beware hallucinated APIs, outdated versions, subtle concurrency/security bugs.
- [ ] **Ownership/accountability:** "I'm accountable for anything I commit"; can explain every line.
- [ ] **Security-sensitive areas** (auth, crypto, payments, input validation): extra scrutiny; don't ask AI to invent crypto.
- [ ] **Prompt hygiene:** give context, constraints, examples, desired output format; iterate; ask for alternatives and trade-offs; ask AI to critique its own answer.
- [ ] **Don't lose skills:** write core logic yourself regularly; use AI to learn, not to skip understanding.
- [ ] **Prompt-injection awareness** when AI agents read external content/tools; least-privilege tool access; human approval for destructive actions.
- [ ] **Measure impact:** time saved, defect rate, review cycle time, coverage — avoid vague claims.

## 5. Likely Follow-Up Questions & Strong Answers
| Question | Strong-answer points |
|---|---|
| **What do you *not* use AI for?** | Final architecture decisions, security-critical code without review, anything needing confidential data in unapproved tools, code I can't explain. |
| **How do you verify AI output?** | Compile/run/tests, read line-by-line, check official docs, static analysis & security scans, benchmark if perf-related, peer review. |
| **Tell me about a time AI was wrong.** | Story D: hallucinated API / insecure suggestion → how caught → process improvement. |
| **How does AI change code review?** | Faster first-pass reviews/smell detection, but human reviews for design, domain, security; larger diffs need stricter review; label AI-assisted PRs per policy. |
| **Impact on junior devs/team?** | Pair juniors with guidelines: explain-before-accept, tests required, learning via "explain this to me"; guard against skill atrophy. |
| **How do you write good prompts?** | Role + context + constraints + examples + output format; plan-first, small steps; provide code/tests; iterate; ask for trade-offs/edge cases. |
| **Security/privacy concerns?** | Approved tools only, data classification, sanitization, secrets scanning, audit/logging, vendor data-retention terms. |
| **Productivity numbers?** | Give your honest measurable ones (tests, boilerplate, debugging, docs); admit it's less useful for deep novel design. |
| **Which tools & why?** | Name tools you actually use (IDE assistants, chat, CLI agents, code-search); how they fit; comparisons in context windows/accuracy/latency/policy. |
| **How would you introduce AI tooling to a team?** | Pilot with volunteers → guidelines (security, review checklist) → training → metrics (cycle time, defects) → iterate → share prompt library. |
| **Do you worry AI will replace engineers?** | Shifts effort from typing to problem framing, design, verification, and domain expertise; engineers who use AI well amplify impact; accountability & judgment remain human. |
| **How do you handle hallucinations?** | Ground with docs/code context (RAG), ask for citations, verify, use tests as oracle, smaller scoped tasks. |
| **Agentic AI / MCP / tool use?** | Know basics: agents plan + call tools; need least privilege, sandboxing, approval gates, observability; good for repetitive multi-step tasks (migrations, test generation) under supervision. |

## 6. Prompting Cheat-Sheet
**Prompt skeleton**
```
Role/Context: You are helping on a Spring Boot 3 / Java 17 service (Postgres, Kafka).
Task: <one clear task>
Constraints: <perf, style, libs allowed, no breaking API, security>
Inputs: <sanitized code / logs / schema>
Output format: <diff / table / steps / tests only>
Verify: List assumptions and edge cases; propose tests.
```
**Techniques:** plan-then-implement · ask for 2–3 alternatives with trade-offs · "critique your own answer" · provide failing test & ask for fix · small iterative diffs · ask "what could go wrong / what did you assume?" · give examples of desired style.

## 7. Self-Check (Be Ready To…)
- [ ] Deliver the 90-second answer smoothly with 3 concrete use-cases and 1 safeguard
- [ ] Present 3 STAR stories with numbers (productivity, debugging, quality-guardrail)
- [ ] Explain exactly how you verify AI output and your data-privacy rules
- [ ] State clearly what you don't delegate to AI and why
- [ ] Answer "tell me about a time AI was wrong" honestly
- [ ] Describe how you'd roll out AI tooling responsibly to a team
