# Week 6 Prac — Static Single-Switch VLANs

Step-by-step instructions for completing the grey Task boxes in the Week 6 prac doc using Cisco Packet Tracer.

This is a **standalone lab** — one switch, three PCs, no router involved. Build it fresh.

---

## Setup — Build the topology

**Where:** Packet Tracer workspace.

1. Open **Cisco Packet Tracer**.
2. Drag in:
   - **Switches** → 1x **Cisco 2960** (this becomes **Switch_A**)
   - **End Devices** → 3x **PC** (PC0, PC1, PC2 — leave default names, they match the doc)
3. Wire it up with copper straight-through cables:
   - **PC0** → **Switch0, port Fa0/1**
   - **PC1** → **Switch0, port Fa0/3**
   - **PC2** → **Switch0, port Fa0/6**
4. Wait for all link lights to go green.

---

## Step 1 — Name the switch

**Where:** Click the switch → **CLI** tab.

```
enable
configure terminal
hostname Switch_A
```

Notice the prompt itself changes from `Switch(config)#` to `Switch_A(config)#` — that confirms it took effect.

---

## Step 2 — Verify VLAN 1 and its default ports

**Where:** Switch_A CLI, back at the privileged prompt (`end` or `exit` out of config mode first).

```
show vlan brief
```

### 📸 Task 1 — write this up

- Answer: how many VLANs are set up **by default** on the switch? Which ports belong to the default VLAN? (Read straight off the `show vlan brief` table — every port not yet reassigned sits in one specific VLAN out of the box.)

---

## Step 3 — Configure the PCs

**Where:** Click each PC → **Desktop** → **IP Configuration**.

| Device | Switchport | IP address | Subnet Mask | Default Gateway |
|---|---|---|---|---|
| PC0 | Fa0/1 | 192.168.1.3 | 255.255.255.0 | 192.168.1.100 |
| PC1 | Fa0/3 | 192.168.1.4 | 255.255.255.0 | 192.168.1.100 |
| PC2 | Fa0/6 | 192.168.1.5 | 255.255.255.0 | 192.168.1.100 |

(The gateway address won't actually be used for anything in this lab since there's no router — it's just there for consistency with the addressing scheme. Set it anyway as instructed.)

---

## Step 4 — Verify connectivity

**Where:** PC0 → **Desktop** → **Command Prompt**.

```
ping 192.168.1.5
```

### 📸 Task 2 — write this up

- Record whether the ping from PC0 to PC2 succeeded.
- If it **failed**: all three PCs are still on the same default VLAN at this point, so a failure here means a cabling or IP-config mistake, not a VLAN issue yet — re-check Setup and Step 3 before continuing. If you still can't get it working, this is the point the doc says to check with your TA before moving on.

---

## Step 5 — Create VLANs

**Where:** Switch_A CLI.

```
configure terminal
vlan 2
vlan 3
end
```

(Typing `vlan 3` while still inside `vlan 2`'s config context is fine — Cisco IOS lets you chain VLAN creation this way; it creates VLAN 2 first, then moves straight into creating VLAN 3.)

---

## Step 6 — Verify the VLANs were created

**Where:** Switch_A CLI.

```
show vlan brief
```

### 📸 Task 3 — write this up

- Answer: are VLAN 2 and VLAN 3 now present in the listing?
- Note the **default name** Packet Tracer/IOS auto-assigned to each (something like `VLAN0002`, `VLAN0003`) — you'll compare this against Step 8's output.

---

## Step 7 — Name the VLANs

**Where:** Switch_A CLI.

```
configure terminal
vlan 2
name Accounting
vlan 3
name HR
end
```

---

## Step 8 — Display VLAN information

**Where:** Switch_A CLI.

```
show vlan brief
```

### 📸 Task 4 — write this up

- Answer: have the VLAN names changed from Step 6's default names to `Accounting` / `HR`?
- Answer: do VLAN 2 and VLAN 3 have any ports assigned to them yet? (Check the Ports column for each — at this point they've only been *created and named*, not yet given any member ports.)

---

## Step 9 — Assign ports to VLAN 2 and VLAN 3

**Where:** Switch_A CLI.

```
configure terminal
interface range Fa0/1-5
switchport mode access
switchport access vlan 2

interface Fa0/6
switchport mode access
switchport access vlan 3
end
```

This puts **PC0 (Fa0/1)** and **PC1 (Fa0/3)** into VLAN 2 (Accounting), and **PC2 (Fa0/6)** into VLAN 3 (HR).

### 📸 Task 5 — write this up

- Answer: what does **access** mode mean for a switchport? (An access port carries traffic for exactly one VLAN and is used for connecting end devices like PCs — as opposed to carrying multiple VLANs at once.)
- Answer: what's the **other** switchport mode, and what does it do? (Look up **trunk** mode — it's used to carry traffic for *multiple* VLANs over a single link, typically between two switches or a switch and a router, by tagging frames with their VLAN ID as they cross the link.)

---

## Step 10 — Display VLAN information again

**Where:** Switch_A CLI.

```
show vlan brief
```

### 📸 Task 6 — write this up

- Screenshot the output.
- Answer: which ports are now listed under VLAN 2? Which under VLAN 3?
- Answer: are Fa0/1, Fa0/3, and Fa0/6 **still** listed under the default VLAN? (A port can only belong to one access VLAN at a time — check whether they moved out of the default VLAN's port list entirely.)

---

## Step 10 (continued) — Verify connectivity

**Where:** PC0 → **Desktop** → **Command Prompt**.

```
ping 192.168.1.4
```
(PC0 → PC1, both now in VLAN 2)

```
ping 192.168.1.5
```
(PC0 → PC2, now in different VLANs)

### 📸 Task 7 — write this up

- Screenshot both ping outputs.
- Record: did PC0 → PC1 succeed? Did PC0 → PC2 succeed?
- Answer **why**: PC0 and PC1 are both in VLAN 2, so they're in the same broadcast domain and can reach each other directly at Layer 2, same as before you created any VLANs. PC2 is now in a *different* VLAN (VLAN 3) — VLANs separate broadcast domains the same way separate physical LANs would, so traffic between VLAN 2 and VLAN 3 needs a Layer-3 device (a router, or a switch doing inter-VLAN routing) to get forwarded between them. Since this topology has no router and nothing configured for inter-VLAN routing, the ping from PC0 to PC2 fails even though both are still plugged into the same physical switch.

---

## Step 11 — Save the configuration

**Where:** Switch_A CLI.

```
copy running-config startup-config
```
or the shorthand `write`. Press Enter to confirm the filename prompt.
