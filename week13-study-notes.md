# COMP2270 / COMP6170 – SGTA Week 13 Study Notes

Three topics: **Well-Architected pillars**, **cloud application architectures**, and **scaling types**.

## What each question is really asking

- **Q1** → Which **pillar** are they **ignoring**? (each finding is a problem / a broken rule)
- **Q2** → Which **architecture** would **fix** it? (finding = symptom, architecture = cure)
- **Q3** → Which **scaling type** is this, or which one do they **need**?

---

## Topic 1: Well-Architected pillars (Q1)

AWS says a good cloud system must be strong in **six areas** (pillars). Think of six things a good **restaurant** must get right.

| Pillar | Restaurant example | Cloud meaning |
|---|---|---|
| **Security** | Only the manager has the key to the safe | Protect data; only the right people get access |
| **Cost optimisation** | Turn off the lights and ovens when closed | Don't pay for things you're not using |
| **Reliability** | Have a backup oven so one breaking doesn't stop service | Keep working even when something fails |
| **Performance efficiency** | Right equipment and enough cooks to serve fast | Use the right resources so the system is fast |
| **Sustainability** | Cut food and energy waste | Reduce environmental impact (carbon, energy) |
| **Operational excellence** | Alarms, checklists, noticing problems early | Monitor, automate, and run things smoothly |

### Answers – which pillar is GlobalMart ignoring?

- **a)** Everyone shares one password for the payment database → **Security**
- **b)** Dev/test servers run 24/7 but are only used in business hours → **Cost optimisation**
- **c)** One server failure took checkout offline for 2 hours → **Reliability**
- **d)** Carbon goal, but always provisioned for peak demand → **Sustainability**
- **e)** Site slowed, no alerts, found out from customer complaints → **Operational excellence**

**Tricky one:** (e) mentions slow servers, which sounds like *performance efficiency*, but the main problem is that **nobody was monitoring**. Missing alerts and monitoring = **operational excellence**.

**Note:** The worksheet says "six findings" but lists only five, so performance efficiency isn't used. Still learn it, since it's one of the six pillars.

**Method:** read the problem → ask "what went wrong?" → match it to the pillar that would have prevented it.

---

## Topic 2: Cloud application architectures (Q2)

These are **different ways of building an app**, not rules. Pick the one that **solves** the problem.

| Architecture | Simple example | Cloud meaning |
|---|---|---|
| **API gateway** | One front reception desk in a hotel; every visitor checks in there | One single entry point that handles login checks and request limits for everyone |
| **Microservices** | A food court: separate stalls, each with its own kitchen and staff | App split into small separate services that run and update independently |
| **Event-driven** | A group chat announcement: "Order placed!" and everyone who cares reacts | Something happens (an event) and other parts react without being called directly |
| **Serverless** | Taking a taxi instead of owning a car; pay only per ride | Code runs only when needed; no servers to manage, no paying for idle time |

### Answers – which architecture fixes it?

- **a)** Website, app and partners each call different addresses and do their own login checks → **API gateway**
- **b)** One bug crashed everything; teams can't release separately → **Microservices**
- **c)** Payment, stock and shipping should start on their own when an order is placed → **Event-driven**
- **d)** Image resizing, irregular traffic, don't want idle servers → **Serverless**

(a) and (b) describe **problems**; (c) and (d) describe **what the team wants**. Either way, pick the architecture that solves it.

### Key words

- *single entry point, login checks, request limits* → API gateway
- *one bug crashes everything, teams can't deploy separately* → microservices
- *when X happens, Y starts on its own* → event-driven
- *no servers to manage, pay only when it runs, idle* → serverless

---

## Topic 3: Scaling types (Q3)

Scaling = **adding or removing servers** to match traffic. Think of a **café deciding how many staff to have working**.

| Scaling type | Café example | Cloud meaning |
|---|---|---|
| **Manual** | Manager notices a long queue and phones extra staff themselves | A person adds servers by hand |
| **Scheduled** | Fixed roster: "Every Friday 6 PM, 3 extra staff" | Servers added/removed at set times |
| **Dynamic** | Staff called in automatically when the queue gets long, sent home when it shrinks | Servers added/removed automatically based on live measurements like CPU |
| **Predictive** | Manager studies months of past sales to bring staff in before the rush | Uses past data to predict demand and add servers in advance |

### Answers

- **a)** Admin adds servers by hand; nobody noticed at 2 AM → **Manual scaling** (this describes what they do *now* – the problem)
- **b)** Same rush every Friday for years; raise at 6 PM, lower at midnight → **Scheduled scaling**
- **c)** Sudden unpredictable spikes; add servers when CPU climbs → **Dynamic scaling**
- **d)** Rush timing shifts, servers slow to boot, months of history → **Predictive scaling**

### Telling the tricky ones apart

- **Scheduled vs predictive** – both prepare ahead of time:
  - Traffic is **the same every time** (fixed day and time) → **scheduled**
  - Traffic **shifts** and there's **historical data** to forecast → **predictive**
- **Dynamic vs predictive:**
  - **Dynamic** *reacts* after traffic rises (follows CPU live)
  - **Predictive** acts *before* traffic rises – that's why (d) mentions slow boot times: reacting is too late

---

## Quick self-test

1. Data stored without encryption and open to anyone: which pillar?
2. A video app's upload, search and billing parts are built and updated separately: which architecture?
3. A university adds servers automatically when CPU hits 70%: which scaling type?
4. A shop adds servers every Monday at 9 AM because sales always start then: which scaling type?

**Answers:** 1. Security · 2. Microservices · 3. Dynamic scaling · 4. Scheduled scaling

---

## My notes

_(Add your own notes here.)_
