# COMP2270/6170 — Assessment 2 (Project) — Step-by-Step Instructions

A task-by-task walkthrough for the multi-site enterprise network build in Cisco Packet Tracer.

> **Personalise before you calculate.** Your address block and Site 1 host count are unique to you (iLearn → *Assessment 2 – Project*). Every worked number below except Site 1's is fixed by the brief and safe to use as-is — swap in your own block and Site 1 count wherever you see a placeholder.

> **Topology used throughout.** Router1/2/3 = Core (fully meshed by three P2P links). Router4 = Site 1, Router5 = Site 2 — both share one "connecting network" with Router2. Router6 = Data Centre, sharing a "connecting network" with Router3. Site 3 hangs off the Data Centre switch pair with no routed interface. Re-label to match your actual file if your device numbers differ.

---

## Before You Start

| Need | Details |
|---|---|
| Starter `.pkt` file | Downloaded from the *Assessment 2 – Project* section, unmodified |
| Personalised spec | Your address block + Site 1 host count, same page |
| File to save as | `StudentID_PT.pkt` — save early, save often |
| Interface labels | Options → Preferences → Interface tab, tick device name/port labels if hidden |

---

## Task 1 — Perform VLSM Subnetting (05 marks)

Break your assigned block into the smallest subnet that fits each requirement, largest requirement first. Nine requirements are listed in the brief (i–ix); Site 3's three VLANs count as three separate subnets, giving **eleven subnets in total**.

### 1.1 Requirements table (fixed values, pre-computed)

Everything here except Site 1 is the same for every student — the host counts come straight from the brief. Site 1 is yours to fill in and slot into the ordering by size.

| Order | Subnet | Hosts needed | 2ⁿ−2 ≥ hosts | Host bits (n) | Prefix | Mask | Usable |
|---|---|---|---|---|---|---|---|
| 1 | Data Centre | 50 | 2⁶−2 = 62 | 6 | /26 | 255.255.255.192 | 62 |
| 2 | Site 2 | 24 | 2⁵−2 = 30 | 5 | /27 | 255.255.255.224 | 30 |
| 3 | Site 1 | *your spec* | — | — | — | — | — |
| 4 | Site 3 — VLAN 10 | 5 | 2³−2 = 6 | 3 | /29 | 255.255.255.248 | 6 |
| 5 | Site 3 — VLAN 20 | 5 | 2³−2 = 6 | 3 | /29 | 255.255.255.248 | 6 |
| 6 | Site 3 — VLAN 30 | 5 | 2³−2 = 6 | 3 | /29 | 255.255.255.248 | 6 |
| 7 | R2–R4–R5 connecting net | 3 | 2³−2 = 6 | 3 | /29 | 255.255.255.248 | 6 |
| 8 | R3–R6 connecting net | ≤5 (scalability) | 2³−2 = 6 | 3 | /29 | 255.255.255.248 | 6 |
| 9 | P2P Router1–Router2 | 2 | 2²−2 = 2 | 2 | /30 | 255.255.255.252 | 2 |
| 10 | P2P Router1–Router3 | 2 | 2²−2 = 2 | 2 | /30 | 255.255.255.252 | 2 |
| 11 | P2P Router2–Router3 | 2 | 2²−2 = 2 | 2 | /30 | 255.255.255.252 | 2 |

**Why the Router2–Router4–Router5 network needs 3 hosts, not 2:** it's a shared multi-access segment (via the LAN Group 1 switches), not a point-to-point link — Router2, Router4 and Router5 each need an IP on it, so it needs at least a /29, not a /30.

### 1.2 Allocate addresses

1. Sort the eleven subnets from most hosts needed to fewest (use the table above; insert Site 1 wherever its size lands).
2. Start at the first address of *your* block. Assign the largest subnet: network address = block start, size = 2ⁿ addresses, broadcast = network + size − 1.
3. The next subnet starts at broadcast + 1 of the previous one. Repeat down the sorted list, always taking the smallest block that still fits the next requirement.
4. For each subnet, record: network address, CIDR/mask, broadcast, first usable, last usable, and what it's assigned to.
5. Pick one consistent addressing convention and note it in the report, e.g. *first usable → router interface*, *remaining usable → hosts*.
6. Check the last broadcast address doesn't exceed your block's upper boundary — if it does, you've mis-ordered or mis-sized a subnet.

**Worked example (illustrative numbers only — do not reuse these).** Block: `192.0.2.0/24`. Largest requirement first (50 hosts → /26):

| Subnet | Network | Mask | Broadcast | First usable | Last usable |
|---|---|---|---|---|---|
| Data Centre /26 | 192.0.2.0 | /26 | 192.0.2.63 | 192.0.2.1 | 192.0.2.62 |
| Site 2 /27 | 192.0.2.64 | /27 | 192.0.2.95 | 192.0.2.65 | 192.0.2.94 |
| next subnet… | 192.0.2.96 | … | … | … | … |

Notice each new subnet begins immediately after the previous broadcast — that's the whole VLSM mechanic, repeated eleven times with your real block and real order.

Put the full table (all 11 rows, your real numbers) in the design report — this is the "VLSM subnetting process" section the report structure asks for.

---

## Task 2 — Assign Hostnames (no marks, but do it)

Best-practice naming, applied consistently across every router and switch. No marks attached, but it makes Tasks 3–7 far easier to follow.

### 2.1 Suggested convention

| Device in topology | Suggested hostname |
|---|---|
| Router1 (Core) | `Core-Router1` |
| Router2 (Core) | `Core-Router2` |
| Router3 (Core) | `Core-Router3` |
| Router4 (Site 1) | `Site1-Router` |
| Router5 (Site 2) | `Site2-Router` |
| Router6 (Data Centre) | `DC-Router` |
| Switch, LAN Group 1 distribution | `Switch-LAN1` |
| Switch, Site 1 access | `Switch-Site1` |
| Switch, Data Centre | `Switch-DC` |
| Switch, Site 3 (left) | `Switch-Site3A` |
| Switch, Site 3 (right) | `Switch-Site3B` |

Match this against your actual file's device count/labels — adjust names if your topology has a different number of switches.

### 2.2 Apply it (per device)

```
-- press Enter to wake the prompt --
Router> enable
Router# configure terminal
Router(config)# hostname Core-Router1
Core-Router1(config)# end
```

Same pattern for switches — `Switch> enable`, `configure terminal`, `hostname Switch-Site1`.

---

## Task 3 — Configure Router Interfaces (03 marks)

Every active router interface gets the correct address and mask from your Task 1 table. Skip the interface facing Site 3 — no IP required there.

1. Open the router's CLI tab, `enable`, `configure terminal`.
2. Enter the interface: `interface GigabitEthernet0/0` (check the port label on the topology diagram — it may be Gig, Fa, or Serial depending on the link).
3. Assign the address: `ip address <network-or-host-address> <subnet-mask>`, using dotted-decimal mask (e.g. `255.255.255.252` for a /30).
4. Bring it up: `no shutdown`.
5. `exit`, then repeat for every other active interface on that router.
6. Leave the Site-3-facing interface with no `ip address` line — connectivity to Site 3 is intentionally not routed.
7. Verify with `show ip interface brief` — every configured interface should read `up / up`.

```
Site1-Router(config)# interface GigabitEthernet0/1
Site1-Router(config-if)# ip address 10.10.1.1 255.255.255.248
Site1-Router(config-if)# no shutdown
Site1-Router(config-if)# exit
Site1-Router(config)# do show ip interface brief
```

| Router | Interfaces needing an IP |
|---|---|
| Router1 (Core) | Link to Router2, link to Router3 |
| Router2 (Core) | Link to Router1, link to Router3, interface on the R2–R4–R5 connecting network |
| Router3 (Core) | Link to Router1, link to Router2, interface on the R3–R6 connecting network |
| Router4 (Site 1) | Site 1 LAN interface, interface on the R2–R4–R5 connecting network |
| Router5 (Site 2) | Site 2 LAN interface, interface on the R2–R4–R5 connecting network |
| Router6 (Data Centre) | Data Centre LAN interface, interface on the R3–R6 connecting network — **not** the Site 3 interface |

---

## Task 4 — Configure PCs and Servers (02 marks)

Static IP, mask, and default gateway on every end device — except Site 3 PCs, which get no gateway (no inter-VLAN routing in this design).

1. Click the PC or server icon in the topology.
2. Open the **Desktop** tab → **IP Configuration**.
3. Select **Static**.
4. Enter **IP Address** and **Subnet Mask** from the subnet that device belongs to (Task 1 table).
5. Enter **Default Gateway** = the local router interface's address on that same subnet — **except** for PC2–PC7 on Site 3, leave the gateway field blank.
6. For servers, the same panel exists under Desktop → IP Configuration; Data Centre servers do get a gateway (they're routed).
7. Close the window — Packet Tracer applies immediately, no save/apply button needed.

> Command-line equivalent on a PC, if you prefer: Desktop → **Command Prompt** won't set the IP, but once static IP is configured via the panel above you can use that prompt to run `ipconfig` and `ping` for verification in Task 7.

---

## Task 5 — Configure Static Routes (03 marks)

Site 1 and Site 2 must reach each other and the Data Centre; Site 3 stays unreachable by design (no gateway on its PCs, no IP on the router interface facing it — no static route needed to enforce that).

### 5.1 The shape of it

Router4, Router5 and Router6 are each "leaf" routers with exactly one way out to the rest of the network — a single default route covers everything they need. Router1, Router2 and Router3 sit in the core and branch in more than one direction, so give them specific routes.

| Router | Route | Next hop |
|---|---|---|
| Router4 (Site 1) | `ip route 0.0.0.0 0.0.0.0` | Router2's IP on the shared connecting network |
| Router5 (Site 2) | `ip route 0.0.0.0 0.0.0.0` | Router2's IP on the shared connecting network |
| Router6 (Data Centre) | `ip route 0.0.0.0 0.0.0.0` | Router3's IP on the R3–R6 connecting network |
| Router2 (Core) | Route to Data Centre LAN | via Router3 |
| Router2 (Core) | Route to the Router1–Router3 P2P subnet | via Router3 |
| Router3 (Core) | Route to Site 1 LAN | via Router2 |
| Router3 (Core) | Route to Site 2 LAN | via Router2 |
| Router3 (Core) | Route to Data Centre LAN | via Router6 |
| Router1 (Core) | Route to Site 1 LAN | via Router2 |
| Router1 (Core) | Route to Site 2 LAN | via Router2 |

Router1 needs those two routes only so it can send the ICMP *reply* back to PC0/PC8 in the Task 7 ping tests — it isn't a source of traffic itself. Site 1 and Site 2 LANs are directly reachable from each other and from Router2 across the shared connecting network, so no extra route is needed there.

```
-- Example: Router4 (Site 1) --
Site1-Router(config)# ip route 0.0.0.0 0.0.0.0 10.10.2.1
Site1-Router(config)# do show ip route static
```

```
-- Example: Router3 (Core, explicit routes) --
Core-Router3(config)# ip route 10.10.1.0 255.255.255.248 10.10.2.2   ! Site 1 LAN, via Router2
Core-Router3(config)# ip route 10.10.3.0 255.255.255.224 10.10.2.2   ! Site 2 LAN, via Router2
Core-Router3(config)# ip route 10.10.4.0 255.255.255.192 10.10.5.2   ! DC LAN, via Router6
```

> **Note:** Addresses above are placeholders — substitute your own Task 1 networks and next-hop interface addresses. If your marker specifically wants explicit per-network routes rather than a default route on the leaf routers, swap Router4/5/6 to explicit routes using the same pattern as Router3's example — check your unit outline or ask your tutor if unsure which is expected.

---

## Task 6 — Configure VLANs (03 marks)

Site 3 gets three VLANs across two switches, with fixed port ranges from the brief — these numbers are the same for everyone.

| VLAN | Left switch ports | Right switch ports | Total hosts |
|---|---|---|---|
| VLAN 10 | Fa0/1–2 | Fa0/1–3 | 5 |
| VLAN 20 | Fa0/6–7 | Fa0/6–8 | 5 |
| VLAN 30 | Fa0/11–12 | Fa0/11–13 | 5 |

### 6.1 Create the VLANs (both switches)

```
Switch-Site3A(config)# vlan 10
Switch-Site3A(config-vlan)# name VLAN10
Switch-Site3A(config-vlan)# exit
Switch-Site3A(config)# vlan 20
Switch-Site3A(config-vlan)# name VLAN20
Switch-Site3A(config-vlan)# exit
Switch-Site3A(config)# vlan 30
Switch-Site3A(config-vlan)# name VLAN30
Switch-Site3A(config-vlan)# exit
```

### 6.2 Assign access ports (left switch shown — repeat with right switch's own ranges)

```
Switch-Site3A(config)# interface range FastEthernet0/1-2
Switch-Site3A(config-if-range)# switchport mode access
Switch-Site3A(config-if-range)# switchport access vlan 10
Switch-Site3A(config-if-range)# exit

Switch-Site3A(config)# interface range FastEthernet0/6-7
Switch-Site3A(config-if-range)# switchport mode access
Switch-Site3A(config-if-range)# switchport access vlan 20
Switch-Site3A(config-if-range)# exit

Switch-Site3A(config)# interface range FastEthernet0/11-12
Switch-Site3A(config-if-range)# switchport mode access
Switch-Site3A(config-if-range)# switchport access vlan 30
Switch-Site3A(config-if-range)# exit
```

On the right switch, same commands with `0/1-3`, `0/6-8`, `0/11-13`.

### 6.3 Trunk the link between the two Site 3 switches

This is the brief's hint: a single link must carry all three VLANs between the switches, so that link — and only that link — needs trunk mode.

```
Switch-Site3A(config)# interface FastEthernet0/24
Switch-Site3A(config-if)# switchport mode trunk
Switch-Site3A(config-if)# exit
```

Check your file for the actual port number joining the two switches — it may not be Fa0/24. Any port not explicitly assigned to a VLAN (including the router-facing port, which needs no IP per Task 3) can stay at its default settings.

Verify with `show vlan brief` on each switch — ports should list under the correct VLAN, and the inter-switch port should show as a trunk in `show interfaces trunk`.

---

## Task 7 — Test and Verify Connectivity (05 marks)

Ping every pair below, record the IPs and result, and screenshot each one for the report. Device labels are the ones from the packet tracer file, not your hostnames.

| Source | Source IP | Destination | Destination IP | Expect |
|---|---|---|---|---|
| PC0 | *fill in* | PC1 | *fill in* | PASS |
| PC0 | *fill in* | PC8 | *fill in* | PASS |
| PC0 | *fill in* | Router 1 Gig0/0 | *fill in* | PASS |
| PC8 | *fill in* | Router 1 Gig0/2 | *fill in* | PASS |
| PC0 | *fill in* | Server1 | *fill in* | PASS |
| PC9 | *fill in* | Server0 | *fill in* | PASS |
| PC2 | *fill in* | PC5 | *fill in* | PASS if same VLAN |
| PC3 | *fill in* | PC6 | *fill in* | PASS if same VLAN |
| PC4 | *fill in* | PC7 | *fill in* | PASS if same VLAN |
| PC2 | *fill in* | PC3 | *fill in* | FAIL expected |

**Why the last four rows are expected to split PASS / FAIL:** based on the VLAN port layout in Task 6, PC2/PC5, PC3/PC6 and PC4/PC7 sit on *matching* VLANs across the two switches (reachable through the trunk), so those three should PASS. PC2 and PC3 sit on the *same* switch but different VLANs — with no inter-VLAN routing configured, that ping should FAIL. Confirm this against your own switch's `show vlan brief` output rather than assuming it — port-to-PC wiring can vary between starter files.

1. Fill in Source/Destination IP columns from your Task 1 & 3 addressing.
2. Test from a PC: Desktop → Command Prompt → `ping <destination-ip>`.
3. Test from a router (for the Router 1 rows): CLI tab → `ping <destination-ip>`.
4. Screenshot each result showing the command and the reply/timeout output.
5. Record Pass/Fail in the table and label each screenshot clearly (e.g. "Figure 4: PC0 → PC1").

---

## Task 8 — Answer the Questions (04 marks)

Four questions, 50 words max each, written in your own words for the report. The points below are a starting frame, not a drop-in answer.

**i. Expanding with two new sites — physical topology options?**
- The R3–R6 connecting network was already sized with scalability headroom — new site routers could attach there the same way Router6 does.
- Alternatively, extend the hierarchical pattern: give each new site its own link into a core router, mirroring how Router4/Router5 attach to Router2.
- Include a simple diagram: two new site boxes connecting into the existing core layer.
*(≤ 50 words · needs a diagram)*

**ii. Why VLSM over fixed-size subnets here?**
- Host counts across this network span from 2 (P2P links) to 62 (Data Centre) — a huge range.
- One fixed subnet size would either waste addresses on small links or be too small for the Data Centre.
- VLSM matches each subnet to its actual need, conserving address space in a finite block.
*(≤ 50 words)*

**iii. Role of the default gateway on a PC?**
- PC compares the destination IP against its own subnet.
- If it's outside the local subnet, the PC hands the frame to the default gateway (the local router interface) instead of ARPing for the destination directly.
- The gateway then routes the packet on toward its destination network.
*(≤ 50 words)*

**iv. Virtualization and AWS — the relationship?**
- AWS's compute services (e.g. EC2) run on a hypervisor layer that partitions physical hardware into isolated virtual machines.
- Virtualization is the enabling technology; AWS is a provider that sells access to it as elastic, multi-tenant, on-demand cloud infrastructure.
*(≤ 50 words)*

---

## Deliverables

Two files, submitted via iLearn: the configured Packet Tracer file, and the design report.

### Packet Tracer file
Save as `StudentID_PT.pkt`. Should reflect Tasks 1–6 fully configured, with Task 7 connectivity already tested in the file.

### Design report — StudentID_Report.pdf (max 15 pages)

- [ ] **Title page** — unit code, your name, student ID.
- [ ] **Client requirements summary** — restate the brief in your own words.
- [ ] **VLSM subnetting process** — the full Task 1 table, all 11 subnets, your real numbers.
- [ ] **Configuration summary** — key CLI commands for *one* router and *one* Site 3 switch only.
- [ ] **Testing table + screenshots** — Task 7 table with IPs, results, and labelled screenshots.
- [ ] **Task 8 answers** — all four, within the word limit.
- [ ] **Appendix** — full addressing table: every router interface, every PC, every server.

> Use *your* assigned address block and Site 1 host count throughout — the brief is explicit that using incorrect data or another student's specification can fail the assessment outright.

---

*Reference guide for COMP2270/6170 Assessment 2 — Project. Cross-check every device label, port number, and interface type against your actual starter `.pkt` file before configuring, since starter files can vary in small ways.*
