# Week 7 Prac — Dynamic Routing using RIP

Step-by-step instructions for completing the grey Task boxes in the Week 7 prac doc using Cisco Packet Tracer.

This lab uses the **same topology as Week 5** (Static Routes) — 2 routers, 3 switches, 8 PCs — but this time you'll replace static routes with **dynamic routing via RIP**.

---

## Setup — Get the topology ready

**Where:** Packet Tracer workspace.

1. **If a starter `.pkt` file was provided** for this week (check iLearn), open that — it already has: hostnames set on both routers, Router0's G0/1 and G0/2 interfaces configured (from Week 4), and PC0–PC5 configured.
2. **If no starter file was provided**, continue from your saved **Week 5** file instead — it's the identical topology and already has everything above done, plus PC6/PC7 and the Router0↔Router1 link cabled in.
3. Either way, confirm you have: **Router0**, **Router1**, 3 switches, and **PC0–PC7** all present and cabled per the diagram (LAN1 → Router0 G0/2, LAN2 → Router0 G0/1, Router0 G0/0 ↔ Router1 G0/0 point-to-point, Router1 G0/1 → LAN3 with PC6/PC7).

### If you see an initial configuration dialog

When opening the CLI on a router for the first time, you may see:
```
Would you like to enter the initial configuration dialog? [yes/no]
```
Type **`no`** and press Enter — this dialog is a guided setup wizard you don't need; it'll drop you at the normal `Router>` prompt instead.

---

## Step a — Configure router interfaces

**Where:** Router0 and Router1 → **CLI** tab.

### Confirm Router0's existing config

1. Click **Router0** → **CLI** tab.
2. Run:
   ```
   enable
   show ip interface brief
   ```
3. Confirm **G0/1** (192.168.20.100) and **G0/2** (192.168.10.100) already show as configured/up.

### Configure the Router0 ↔ Router1 link (Router0 side)

4. Still on Router0's CLI:
   ```
   configure terminal
   interface g0/0
   ip address 192.168.30.1 255.255.255.252
   no shutdown
   ```
5. Check `do show ip interface brief` — G0/0 will show `up`, but **line protocol will still show `down`**. That's expected — the other end isn't configured yet. **Ask your TA to explain why an interface can be administratively `up` while the line protocol is still `down`**, before continuing.

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
9. Recheck `show ip interface brief` on both routers — G0/0 should now show `up`/`up` on each end.

---

## Step b — Configure the PCs

**Where:** Click each PC → **Desktop** → **IP Configuration**.

PC0–PC5 are already done. Configure only the two new ones:

| PC | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|
| PC6 | 192.168.40.1 | 255.255.255.128 | 192.168.40.100 |
| PC7 | 192.168.40.2 | 255.255.255.128 | 192.168.40.100 |

---

## Step c — Test connectivity within each network

**Where:** Command Prompt on PC0, PC3, PC6.

1. **PC0** → `ping 192.168.10.2` (PC1, same LAN).
2. **PC3** → `ping 192.168.20.2` (PC4, same LAN).
3. **PC6** → `ping 192.168.40.2` (PC7, same LAN).

All three should succeed — this just confirms local switching/addressing before routing comes into play.

---

## Step d — Test connectivity across networks (before RIP)

**Where:** Command Prompt on PC0 and PC3.

1. **PC0** → `ping 192.168.20.3` (PC3).
2. **PC0** → `ping 192.168.40.2` (PC7).
3. **PC3** → `ping 192.168.40.1` (PC6).

### 📝 Task 1 — write this up

- Record whether each of the three pings above succeeded or failed.
- Write down **your own theory** for why you see that pattern before discussing it — think about which networks are directly connected to which router versus which require crossing to the other router, and what's actually in each router's routing table right now (nothing beyond directly-connected networks, since no static routes or dynamic routing protocol has been configured yet).
- Then **ask your TA** to confirm/correct your reasoning before proceeding.

---

## Step e — Check interface configuration and routing table

**Where:** CLI on both Router0 and Router1.

```
show ip interface brief
show ip route
```
Run both commands on each router. No formal task here — this is just a "before" baseline so Task 2 later has something to compare against.

---

## Step f — Enable RIP on both routers

**Where:** Router0 and Router1 → CLI.

On **Router0**:
```
enable
configure terminal
router rip
```
On **Router1**:
```
enable
configure terminal
router rip
```
Both should now sit at the `(config-router)#` prompt — this confirms a RIP process is running on each router (it doesn't advertise anything yet until Step g).

---

## Step g — Enable networks for RIP advertisement

**Where:** Still at the `(config-router)#` prompt on each router.

On **Router0**, advertise all three networks it's directly connected to:
```
network 192.168.10.0
network 192.168.20.0
network 192.168.30.0
```
On **Router1**, advertise its two directly-connected networks:
```
network 192.168.40.0
network 192.168.30.0
```

Note: you only ever type the **classful/major network address** here (not a subnet mask) — RIP's `network` command tells the router *which of its own interfaces* to run RIP on and advertise, by matching the interface's IP against the network you list.

---

## Step h — Check the routing table (again)

**Where:** CLI on both routers, back at the privileged prompt (`end` then `show ip route`, or `do show ip route` if still in router config mode).

### 📸 Task 2 — write this up

- Screenshot `show ip route` on **both** routers.
- Answer: are **all** 4 networks now present in Router0's table? In Router1's table? (Give RIP a few seconds — updates are periodic, not instant — and check again if something's still missing.)
- Answer: what does the **`R`** code mean in the output? (Check the legend Cisco IOS prints at the top of `show ip route` — compare it against the `C` for directly-connected routes you seen in earlier weeks; `R` marks routes the router *learned* rather than ones it worked out for itself.)

---

## Step i — Test connectivity across networks (again)

**Where:** Command Prompt on PC0 and PC3.

1. **PC0** → `ping 192.168.40.2` (PC7).
2. **PC3** → `ping 192.168.40.1` (PC6).

### 📸 Task 3 — write this up

- Ping **PC6 from PC3**. Screenshot the output. Record: total packets sent / received / lost.
- Ping **PC7 from PC0**. Screenshot the output. Record: total packets sent / received / lost.

---

## Step j — Save the configuration

**Where:** CLI on **both** Router0 and Router1.

```
copy running-config startup-config
```
or the shorthand `write`, on each router. Press Enter to confirm the filename prompt.
