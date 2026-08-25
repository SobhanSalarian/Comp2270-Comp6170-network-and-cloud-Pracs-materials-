# Week 5 Prac — Static Routes

Step-by-step instructions for completing the grey Task boxes in the Week 5 prac doc using Cisco Packet Tracer.

This lab **builds directly on the Week 4 file** — Router0's G0/1 and G0/2 interfaces and PC0–PC5 are already configured from last week. Open your saved Week 4 `.pkt` file to continue, rather than starting from scratch.

---

## Setup — Extend the topology

**Where:** Packet Tracer workspace (your Week 4 file).

1. Open your **Week 4** Packet Tracer file.
2. Add the new devices:
   - **Routers** → 1x **Router 2911** (this becomes **Router1**)
   - **Switches** → 1x **Switch 2960** (3rd switch, for the new LAN)
   - **End Devices** → 2x **PC** (**PC6**, **PC7**)
3. Wire it up:
   - **Router0 G0/0** ↔ **Router1 G0/0** — direct router-to-router link. Use a copper straight-through cable (or "Automatically Choose Connection Type") — the 2911's ports support Auto-MDIX so a straight-through cable works fine here, no crossover needed.
   - **Router1 G0/1** → new **Switch2** (right-hand switch).
   - **Switch2** → **PC6** and **PC7** via copper straight-through cables.
4. Wait for all link lights to go green (the router-to-router link will only go green *after* you configure both ends in Step b — that's expected, not a mistake).

Reference layout (from the doc):

| LAN 1 (192.168.10.0/24) | LAN 2 (192.168.20.0/24) | P2P link (192.168.30.0/30) | LAN 3 (192.168.40.0/25) |
|---|---|---|---|
| PC0(.1), PC1(.2), PC2(.3) → Switch0 → **Router0 G0/2** (.100) | PC3(.3), PC4(.2), PC5(.1) → Switch1 → **Router0 G0/1** (.100) | **Router0 G0/0** (.1) ↔ **Router1 G0/0** (.2) | **Router1 G0/1** (.100) → Switch2 → PC6(.1), PC7(.2) |

---

## Step b — Configure router interfaces

**Where:** Router0 and Router1 → **CLI** tab.

### Confirm Router0's existing config

1. Click **Router0** → **CLI** tab.
2. Run:
   ```
   enable
   show ip interface brief
   ```
3. Confirm **G0/1** (192.168.20.100) and **G0/2** (192.168.10.100) already show as configured/up from last week. If they don't, go back and re-do Week 4's Step f before continuing.

### Configure the new Router0 ↔ Router1 link (Router0 side)

4. Still on Router0's CLI:
   ```
   configure terminal
   interface g0/0
   ip address 192.168.30.1 255.255.255.252
   no shutdown
   ```
5. Check `show ip interface brief` (or `do show ip interface brief` if you're still inside config mode) — G0/0 should now show `up`, but the **line protocol column will still show `down`**. That's expected at this point — the other end isn't configured yet.**WHY ?**

### Configure Router1

6. Click **Router1** → **CLI** tab.
7. Configure G0/0 (the link back to Router0):
   ```
   enable
   configure terminal
   interface g0/0
   ip address 192.168.30.2 255.255.255.252
   no shutdown
   ```
8. Configure G0/1 (the interface facing PC6/PC7):
   ```
   interface g0/1
   ip address 192.168.40.100 255.255.255.128
   no shutdown
   ```
9. Go back and check `show ip interface brief` on **both** routers — G0/0 on each end should now show `up`/`up`, since both sides of the link are configured.

---

## Step c — Configure the PCs

**Where:** Click each PC → **Desktop** → **IP Configuration**.

PC0–PC5 are already done (carried over from Week 4). Just configure the two new PCs:

| PC | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|
| PC6 | 192.168.40.1 | 255.255.255.128 | 192.168.40.100 |
| PC7 | 192.168.40.2 | 255.255.255.128 | 192.168.40.100 |

Note the subnet mask here is **255.255.255.128 (/25)**, not /24 like the other two LANs — don't copy-paste the wrong mask.

---

## Step d — Test connectivity within each network

**Where:** Command Prompt on PC0, PC3, and PC6.

1. **PC0** → Desktop → Command Prompt → `ping 192.168.10.2` (pings PC1, same LAN).
2. **PC3** → Desktop → Command Prompt → `ping 192.168.20.2` (pings PC4, same LAN).
3. **PC6** → Desktop → Command Prompt → `ping 192.168.40.2` (pings PC7, same LAN).

All three should succeed — this just confirms each LAN's switching/addressing is fine before you touch routing.

---

## Step e — Test connectivity across networks (before static routes)

**Where:** Command Prompt on PC0 and PC3.

1. **PC0** → Command Prompt → `ping 192.168.20.3` (PC3 — reachable via Router0 directly, no static route needed yet since both LANs are directly connected to Router0).
2. **PC0** → Command Prompt → `ping 192.168.40.2` (PC7 — this crosses through Router1, likely to fail).
3. **PC3** → Command Prompt → `ping 192.168.40.1` (PC6 — same idea, likely to fail).
4. Note which ones succeed and which fail. **Why ??**

---

## Step f — Check interfaces and routing tables

**Where:** CLI on both Router0 and Router1.

1. On **Router0**: `show ip interface brief`, then `show ip route`.
2. On **Router1**: `show ip interface brief`, then `show ip route`.

### 📸 Task 1 — write this up

- Screenshot the `show ip route` output on **both** routers (2 screenshots).
- Answer: which network(s) are **missing** from Router0's routing table? Which are missing from Router1's? (A router only auto-knows networks directly connected to its own interfaces — compare each router's interface list against all 4 networks in the topology to spot the gap.)

---

## Step g — Configure static routes on Router0

**Where:** Router0 → CLI.

1. Enter config mode and add a route for the one network Router0 doesn't know about yet (PC6/PC7's LAN):
   ```
   configure terminal
   ip route 192.168.40.0 255.255.255.128 G0/0
   ```
   or, equivalently, using the next-hop IP instead of the exit interface:
   ```
   ip route 192.168.40.0 255.255.255.128 192.168.30.2
   ```
   (Pick one — don't add both for the same route. The doc's note: exit-interface style can generate extra ARP traffic; next-hop style requires an extra recursive lookup. Either is fine for this lab.)
2. Check the routing table again:
   ```
   do show ip route
   ```

### 📸 Task 2 — write this up

- Screenshot the `show ip route` output on Router0.
- Answer: is the previously-missing network now listed? What does the **`S`** code mean in the output? (Check the legend Cisco IOS prints at the top of `show ip route` — it lists what each route-source letter stands for.)

---

## Step h — Test connectivity across networks (again)

**Where:** Command Prompt on PC0 and PC3.

1. **PC0** → `ping 192.168.40.2` (PC7).
2. **PC3** → `ping 192.168.40.1` (PC6).
3. Note the results. 

---

## Step i — Configure static routes on Router1

**Where:** Router1 → CLI.

1. Add a route back to PC0/PC1/PC2's LAN:
   ```
   configure terminal
   ip route 192.168.10.0 255.255.255.0 G0/0
   ```
2. Now test: **PC0** → `ping 192.168.40.2` (PC7). Note the result. before continuing.
3. Test: **PC3** → `ping 192.168.40.1` (PC6). Note the result. before continuing (one route alone won't fix every path — think about which networks Router1 still doesn't know about).
4. Add the second static route on Router1, for PC3/PC4/PC5's LAN:
   ```
   ip route 192.168.20.0 255.255.255.0 G0/0
   ```
5. Check `do show ip route` on Router1 and confirm both new routes appear.
6. Test **PC3** → `ping 192.168.40.1` (PC6) one more time — it should succeed now.

### 📸 Task 3 — write this up

- Ping **PC6 from PC3**. Screenshot the output. Record: total packets sent / received / lost (from the summary line).
- Ping **PC7 from PC0**. Screenshot the output. Record: total packets sent / received / lost.

---

## Step j — Configure a default route

**Where:** Router0 → CLI.

A default route matches *any* destination not otherwise found in the routing table — useful when a router has only one way "out" (here, Router0's only route off its local LANs is via Router1).

1. On Router0:
   ```
   configure terminal
   ip route 0.0.0.0 0.0.0.0 G0/0
   ```
2. Check the routing table:
   ```
   do show ip route
   ```

### 📸 Task 4 — write this up

- Screenshot the `show ip route` output on Router0.
- Answer: what new route entry appears? What does **`S*`** mean specifically (as opposed to plain `S`)? (Same legend at the top of the `show ip route` output — look for how it distinguishes a default route from a regular static route.)

---

## Step k — Save the configuration

**Where:** CLI on **both** Router0 and Router1.

Run on each:
```
copy running-config startup-config
```
or the shorthand `write`. Press Enter to confirm the filename prompt.

---

