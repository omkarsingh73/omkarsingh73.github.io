- [[#1. The Agile Manifesto (1001 words to remember → 4 values)|1. The Agile Manifesto (1001 words to remember → 4 values)]]
- [[#2. 12 Agile Principles (Grouped for fast recall)|2. 12 Agile Principles (Grouped for fast recall)]]
- [[#3. Agile vs Traditional (Waterfall)|3. Agile vs Traditional (Waterfall)]]
- [[#4. Agile Mindset vs Agile Process|4. Agile Mindset vs Agile Process]]
- [[#5. Agile Frameworks Overview|5. Agile Frameworks Overview]]
- [[#6. Agile Estimation|6. Agile Estimation]]
- [[#7. Agile Metrics|7. Agile Metrics]]
- [[#8. Definition of Ready vs Definition of Done|8. Definition of Ready vs Definition of Done]]
- [[#9. Scaling Agile (Brief Overview)|9. Scaling Agile (Brief Overview)]]
- [[#10. Agile Anti-Patterns (Senior Dev Gold)|10. Agile Anti-Patterns (Senior Dev Gold)]]
- [[#Key Takeaways|Key Takeaways]]


---

## 1. The Agile Manifesto (1001 words to remember → 4 values)

### 4 Core Values

| We value MORE → | Over → |
|---|---|
| **Individuals and interactions** | Processes and tools |
| **Working software** | Comprehensive documentation |
| **Customer collaboration** | Contract negotiation |
| **Responding to change** | Following a plan |

> 💡 **Interview Trap:** "Agile says documentation is not important." WRONG. The right side still has value — it's just *less* valued than the left side.

---

## 2. 12 Agile Principles (Grouped for fast recall)

### Customer & Delivery
- Satisfy customers through **early and continuous delivery** of valuable software
- **Welcome changing requirements**, even late in development
- **Deliver working software frequently** (weeks, not months)

### People & Collaboration
- **Business and developers collaborate daily**
- Build projects around **motivated individuals** — give them the environment and trust
- **Face-to-face conversation** is the most efficient communication
- **Sustainable pace** — agile teams should maintain indefinitely

### Quality & Design
- **Working software is the primary measure of progress**
- Continuous attention to **technical excellence and good design**
- **Simplicity** — maximizing work not done

### Inspection & Adaptation
- Best architectures emerge from **self-organizing teams**
- Teams **regularly reflect** and adjust behavior (retrospectives)

---

## 3. Agile vs Traditional (Waterfall)

| Aspect | Waterfall | Agile |
|---|---|---|
| Planning | Upfront, detailed | Adaptive, iterative |
| Requirements | Fixed at start | Evolve continuously |
| Delivery | End of project | Frequent increments |
| Risk handling | Late discovery | Early, continuous |
| Customer involvement | Start & end | Continuous |
| Change | Expensive | Embraced |
| Team structure | Silos (Dev, QA, BA) | Cross-functional |
| Success measure | On time/budget | Business value delivered |

---

## 4. Agile Mindset vs Agile Process

```
Agile Mindset:          Agile Process:
- Embrace uncertainty   - Scrum, Kanban, XP
- Empiricism            - Ceremonies, artifacts
- Collaboration         - Metrics, velocity
- Value delivery        - Tools (Jira, Confluence)

You can follow Agile processes without the mindset → "Zombie Scrum"
```

---

## 5. Agile Frameworks Overview

| Framework | Best For | Key Concept |
|---|---|---|
| **Scrum** | Teams with evolving requirements | Sprints, ceremonies, roles |
| **Kanban** | Continuous flow, ops, maintenance | WIP limits, flow |
| **XP (Extreme Programming)** | Engineering excellence | TDD, pair programming, CI |
| **SAFe** | Large enterprise scaled agile | ARTs, PI Planning |
| **LeSS** | Multi-team Scrum | Shared backlog, minimal bureaucracy |
| **Nexus** | 3-9 Scrum teams | Integration team, dependencies |
| **Crystal** | Team-tailored | Osmotic communication |
| **DSDM** | Business-driven | MoSCoW, timeboxing |

---

## 6. Agile Estimation

### Story Points vs Hours

| Story Points | Hours |
|---|---|
| Relative complexity | Absolute time |
| Team-specific | Individual-specific |
| Includes uncertainty | Assumes known |
| Doesn't change with team size | Changes with who does it |

### Planning Poker

```
1. PO presents user story
2. Each dev privately picks a card (Fibonacci: 1,2,3,5,8,13,21,?)
3. All reveal simultaneously
4. Discuss outliers (highest and lowest)
5. Re-estimate until consensus
```

### T-Shirt Sizing

```
XS, S, M, L, XL, XXL — used for epics/features in early planning
Later broken into story points for sprints
```

### Fibonacci Scale & Why

```
1, 2, 3, 5, 8, 13, 21, 34, 55, 89...
As complexity grows, uncertainty grows proportionally
Gaps between large numbers prevent false precision
```

---

## 7. Agile Metrics

| Metric | Definition | Use |
|---|---|---|
| **Velocity** | Story points completed per sprint | Forecasting future sprints |
| **Burndown Chart** | Remaining work vs time (sprint) | Sprint progress |
| **Burnup Chart** | Work done vs total scope | Shows scope creep |
| **Lead Time** | Idea → delivered to customer | End-to-end efficiency |
| **Cycle Time** | Dev started → done | Team throughput |
| **Escaped Defects** | Bugs found in production | Quality indicator |
| **Technical Debt** | Accumulated shortcuts | Code health |
| **WIP** | Items in progress | Flow efficiency |

---

## 8. Definition of Ready vs Definition of Done

### Definition of Ready (DoR)
Story is ready to be pulled into sprint when:
- [ ] User story is clearly written (As a… I want… So that…)
- [ ] Acceptance criteria defined
- [ ] Sized/estimated
- [ ] Dependencies identified
- [ ] No blockers

### Definition of Done (DoD)
Story is done when:
- [ ] Code written and peer reviewed
- [ ] Unit tests written and passing
- [ ] Integration tests passing
- [ ] No new critical bugs
- [ ] Deployed to staging/test env
- [ ] Acceptance criteria verified
- [ ] Documentation updated
- [ ] Product Owner accepted

> 💡 **Interview Point:** DoD applies to every story. DoR is about sprint readiness. Both should be team agreements, not individual.

---

## 9. Scaling Agile (Brief Overview)

### SAFe (Scaled Agile Framework)
```
Team Level → Program Level (ART) → Large Solution → Portfolio

Key concepts:
- ART: Agile Release Train (50-125 people)
- PI Planning: Program Increment planning (8-12 weeks)
- Lean-Agile Principles
- Heavy framework, controversial in pure agile circles
```

### LeSS (Large Scale Scrum)
```
- Multiple teams working on ONE product backlog
- One Product Owner for entire product
- LeSS Huge: for 8+ teams (Area POs)
- Minimal extra roles — relies on Scrum + coordination events
```

### Nexus
```
- Scrum.org framework for 3-9 Scrum teams
- Nexus Integration Team (NIT): cross-team coordination
- Integrated Increment every sprint
- Nexus Sprint Backlog: identifies cross-team dependencies
```

---

## 10. Agile Anti-Patterns (Senior Dev Gold)

| Anti-Pattern | Symptom | Fix |
|---|---|---|
| **Zombie Scrum** | Ceremonies done, no real agility | Culture change, inspect & adapt |
| **Scrum-But** | "We do Scrum, but no retros…" | Address root cause |
| **Velocity as KPI** | Management pressures on velocity | Use velocity for forecasting only |
| **Fake urgency sprints** | Every sprint is critical | Proper prioritization |
| **No DoD** | "Done" means different things | Agree on explicit DoD |
| **One-man show PO** | PO absent, developer decides value | PO must be available |
| **Gold Plating** | Building beyond acceptance criteria | Focus on DoD, not more |
| **Waterfall in disguise** | Big upfront design in sprint 1 | Incremental architecture |

---

## Key Takeaways

- Agile is a **mindset first**, framework second
- The Manifesto has **4 values and 12 principles** — know them cold
- **Velocity is a planning tool**, not a performance metric
- **DoD is non-negotiable** — partial done is not done
- Scaling frameworks (SAFe, LeSS, Nexus) are band-aids if team-level agility is broken
- Agile works best with **self-organizing, cross-functional, co-located (or well-synced) teams**
