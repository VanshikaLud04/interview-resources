# HR & Behavioral — Q&A

## STAR Format
**S**ituation → **T**ask → **A**ction (say "I", not "we") → **R**esult (quantified).

---

## Q. "Why do you want to work here?"
**A:** "I've spent the last year on backend systems and AI infrastructure—LLM Cost Guard handles 7,000+ concurrent requests. I know [Company] is solving [their scaling challenge]. I want to bring my async systems and distributed architecture experience to a team where I build products impacting thousands of users."

## Q. "Strengths?"
**A:** 1) **System-level thinking** — EE + backend means I think about CPU, memory, DB locks. In Focus Lock I designed entropy-based adaptive sampling to reduce how often heavy inference runs. 2) **Diving into large codebases** — I submitted a LangChain policy-governance callback to Cordum and learned to work within an established integration layer.

## Q. "Weakness?"
**A:** "Over-engineering. During early hackathons I'd try full microservices + Kafka for a simple MVP. Now I build robust monolith first, only add complexity like RabbitMQ when traffic demands it."

## Q. "Time you failed?"
**A:** "During ArduPilot research, I spent weeks writing a custom vision script ignoring existing MAVLink protocol standards. At integration, nothing communicated. Scrapped a month of work. Taught me: read interface documentation and understand the broader system before writing code. Applied this to LLM Cost Guard—architected Redis interfaces first."

## Q. "Conflict with a teammate?"
**A:** "HackOdisha—teammate wanted a heavy state management library with 12 hours left. I proposed React context for the MVP and offered to help wire endpoints. We agreed, shipped on time, placed top 30/441 teams."

## Q. "5 years?"
**A:** "Senior Backend Engineer or Distributed Systems Architect. Go-to person for scaling AI infrastructure. Mentoring junior engineers."

---

## Hackathon Stories

**MeitY SWAYAAN (National Finalist — Drone CV)** → Use for: pressure, hardware/software integration, leadership. Key: constraint-aware programming (limited battery/compute).

**HackOdisha (VeriChain — Top 30/441)** → Use for: teamwork, rapid prototyping, learning new stack quickly.

**ElectroHack'25 (2nd Place)** → Use for: cross-disciplinary problem solving, polished delivery under deadline.

---

## Tricky Questions

### Q. "Why EE if you want CS?"
**A:** "Complementary. EE gave me math, signal processing, low-level system constraints. When I write backend Python/C++, I understand what the hardware is doing. EE makes me a more hardware-aware software engineer."

### Q. "CGPA is 7.57?"
**A:** "I balanced a rigorous EE curriculum with project work, research, open-source contribution, and competitions. The useful signal is the work I can walk through clearly—design choices, code, and evidence—not an inflated claim about a PR that was not merged."

### Q. "No corporate internship?"
**A:** "I pursued hands-on work: a public open-source contribution with a real design iteration, plus a research internship across NIT Jamshedpur and IIT Guwahati building simulation pipelines. I can explain exactly what I owned and what I learned from each."

---

## Questions to Ask
1. "Biggest technical challenge your team faces right now?"
2. "How do you handle tech debt vs shipping features?"
3. "Engineering culture—are deployments painful or automated?"
4. "I saw you launched [X feature]. How did backend handle the scaling?"

**Never ask:** "What does your company do?" / "Do I have to work long hours?" / "Did I get the job?"
