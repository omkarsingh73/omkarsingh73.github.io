
- [[#Top 20 Agile Interview Questions|Top 20 Agile Interview Questions]]
- [[#Common Agile Interview Traps 🪤|Common Agile Interview Traps 🪤]]
- [[#Behavioral Discussion Points (Senior Dev)|Behavioral Discussion Points (Senior Dev)]]
- [[#Key Takeaways|Key Takeaways]]

## Top 20 Agile Interview Questions

### Conceptual / Principle-Based

**Q1. What's the difference between Agile and Scrum?**
> Agile is a **mindset/philosophy** (values + principles from the Manifesto). Scrum is a **framework** that implements Agile. You can be Agile without Scrum. Scrum is one way to practice Agile.

**Q2. Can Agile work without co-located teams?**
> Yes, but it requires extra effort. Distributed Agile relies on strong async communication, video ceremonies, shared tools (Jira, Confluence, Miro), overlapping time zones, and deliberate relationship-building. The Manifesto prefers face-to-face but acknowledges reality.

**Q3. How do you handle scope creep in Agile?**
> Agile welcomes change but through a process: (1) New items go to the backlog, not the current sprint. (2) If critical, negotiate with PO to swap an equal-sized item out. (3) Protect the sprint goal. (4) Educate stakeholders that "Agile means we can change anytime" is a misconception.

**Q4. What is the Agile "inspect and adapt" principle?**
> Empirical process control: make decisions based on observation, not assumption. In Scrum: Sprint Review (inspect product), Retrospective (inspect process). Don't just run ceremonies — act on findings.

**Q5. How do you measure success in an Agile project?**
> Primary: **working software delivered and customer satisfaction**. Supporting: cycle time, escaped defects, team happiness, sustainable velocity. NOT: lines of code, number of sprints completed, or hours worked.

---

### Scenario-Based Questions

**Q6. 🔴 Scenario: Stakeholders keep adding requirements mid-sprint. What do you do?**
> (1) Explain the sprint commitment and sprint goal. (2) Log the request in the backlog. (3) If truly urgent, escalate to PO — who decides if it's worth breaking the sprint. (4) If it happens repeatedly, address it in the retrospective and agree on rules. (5) Never silently accept mid-sprint scope — it hides the real cost.

**Q7. 🔴 Scenario: Your team's velocity dropped 40% this sprint. How do you respond?**
> (1) Investigate root cause: illness, dependencies, tech debt, unclear stories, tooling issues. (2) Don't panic or pressure the team. (3) Raise in the retrospective. (4) Protect next sprint forecast using the new data. (5) Velocity fluctuation is normal — look at trends, not single sprints. Never use velocity as a performance target.

**Q8. 🔴 Scenario: The team says "we're Agile, we don't need documentation." Respond.**
> Agile values working software over *comprehensive* documentation — not zero documentation. You still need: API contracts, architecture decisions (ADRs), runbooks, onboarding guides. The key is "just enough" documentation — useful, maintained, not exhaustive.

**Q9. 🔴 Scenario: A senior dev constantly estimates stories as "1 point" to inflate velocity. How do you handle it?**
> (1) Address it privately first — explain that velocity inflation hurts forecasting accuracy. (2) Use team estimation (planning poker) to prevent individual manipulation. (3) Reinforce that velocity is a team planning tool, not a performance metric. (4) If behavior continues, escalate to Scrum Master or Engineering Manager.

**Q10. 🔴 Scenario: Management wants a fixed delivery date for a feature that isn't estimated yet. How do you respond?**
> Apply the Agile Iron Triangle: fix scope OR fix time, but not both without flexibility in the other. Options: (1) Rough size the work now, give a confidence range. (2) Time-box: "Here's what we can deliver by X date." (3) Break into MVP — what's the minimum viable version for the date? Avoid committing to both fixed scope and fixed date without data.

---

### Estimation & Metrics Questions

**Q11. How do you explain story points to a non-technical stakeholder?**
> "Think of it like T-shirt sizes. A size-S task is simpler and faster than a size-XL. We size based on complexity, not time — because different developers have different speeds, but complexity stays the same. Over time, our history tells us how many points we complete per sprint, so we can forecast."

**Q12. When should you stop using velocity for forecasting?**
> When: (1) Team composition changed significantly. (2) Tech stack changed. (3) Scope of stories changed (e.g., started including test automation). (4) Team just formed (first 3-4 sprints are calibration). Reset baseline when any of these change.

**Q13. What's the difference between a burndown chart and a burnup chart?**
> **Burndown**: starts high, shows remaining work decreasing → good for sprint tracking. **Burnup**: starts at zero, shows completed work increasing → better for showing scope creep (total line moves up when scope is added). Burnup is more honest in long-running projects.

**Q14. How do you estimate an epic with high uncertainty?**
> (1) T-shirt size it (S/M/L/XL) for roadmap planning. (2) Break into features → break into stories. (3) Only estimate stories that will be done in the next 1-2 sprints in detail. (4) Use a "spike" to reduce uncertainty before estimating. (5) Accept that early estimates are rough — refine as you learn.

---

### Senior / Tech Lead Behavioral Questions

**Q15. How have you improved your team's agile process?**
> Strong answer includes: identifying a specific pain point (e.g., too many bugs in production, sprint goals not met), proposing a measurable change (e.g., introduced DoD with mandatory code review + integration tests), measuring the outcome (escaped defects reduced by X%), and sustaining it (added to team norms).

**Q16. How do you balance technical debt with feature delivery in Agile?**
> (1) Make technical debt visible — add it to the backlog with business impact. (2) Reserve 10-20% of sprint capacity for tech debt reduction. (3) Use the "boy scout rule" — improve code you touch. (4) Frame debt as risk to PO: "If we don't address this, the next feature in this area will take 3x longer." (5) Track with metrics (code complexity, bug rate in legacy modules).

**Q17. How do you handle disagreement with the Product Owner on priorities?**
> (1) Understand their reasoning — what business driver is behind the priority? (2) Share technical perspective: "This dependency means X must come before Y or we'll re-do it." (3) Propose alternatives: "If we do A first, we can deliver B+C in the same sprint as your original plan." (4) Ultimately, PO owns the backlog — respect the decision after making your case.

**Q18. What is "sustainable pace" and why does it matter?**
> Teams should work at a pace they can maintain indefinitely. Sprint after sprint of overtime leads to burnout, quality degradation, and attrition. A team that consistently delivers at 80% capacity outperforms a team that sprints at 120% capacity for 3 months then crashes. A senior dev should protect the team from unsustainable pressure.

**Q19. What's the most common reason Agile transformations fail?**
> Strongest answer: (1) **Management doesn't change** — Agile requires management to shift from command-and-control to servant leadership. (2) **Teams "do Agile" mechanically** without understanding why (Zombie Scrum). (3) **No psychological safety** — teams can't raise impediments honestly. (4) **Short-term thinking** — expecting Agile to fix everything in one quarter.

**Q20. How do you coach a team member who resists Agile?**
> (1) Understand their concern — is it process overhead? Loss of technical autonomy? Lack of trust? (2) Acknowledge valid points (some Agile implementations do create overhead). (3) Focus on what's in it for them: fewer surprises, clearer expectations, protected time for tech work. (4) Involve them in shaping the process — people support what they help create. (5) Let results speak — demonstrate improvement, not just principles.

---

## Common Agile Interview Traps 🪤

| Trap Statement | Correct Response |
|---|---|
| "Agile means no planning" | Agile plans continuously — just not all upfront |
| "Agile means no documentation" | "Just enough" documentation — useful not exhaustive |
| "In Agile, requirements can change anytime" | Changes go through PO and backlog, not mid-sprint |
| "Velocity tells you how productive the team is" | Velocity is a forecasting tool, not a performance metric |
| "Agile doesn't work for large projects" | Scaling frameworks exist; small Agile teams can coordinate |
| "Done means all user stories completed" | Done means sprint goal met + DoD criteria satisfied |

---

## Behavioral Discussion Points (Senior Dev)

Use STAR format (Situation → Task → Action → Result):

- "Tell me about a time you improved your team's delivery process"
- "Describe a situation where you pushed back on a deadline"
- "How did you handle a sprint where the team was struggling to meet the goal?"
- "Give an example of introducing a technical practice that improved agility"
- "Tell me about a failed sprint and what you learned"

---

## Key Takeaways

- Know the **4 values and 12 principles** — interviewers test nuance, not memorization
- Scenario answers should show **judgment + collaboration**, not just process knowledge
- Senior devs are expected to **coach, advocate, and improve** the process — not just follow it
- Velocity is **never a performance metric** — this comes up in every senior interview
- Technical excellence (TDD, CI/CD, clean code) is **part of Agile**, not separate from it
