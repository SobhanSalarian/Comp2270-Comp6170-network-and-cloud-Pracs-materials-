# Week 4 Prac — Inter-network & Basic Routing Connectivity

Step-by-step instructions for completing the grey Task boxes in the Week 4 prac doc using Cisco Packet Tracer.

---

## Setup — Build the topology

**Where:** Packet Tracer workspace.

1. Open **Cisco Packet Tracer**.
2. Drag in the devices you need:
   - **Routers** → 1x **Router 2911**
   - **Switches** → 2x **Switch 2960**
   - **End Devices** → 6x **PC**
3. Wire it up to match the topology diagram in the doc:
   - **PC0, PC1, PC2** → connect to **Switch0** (left switch) via copper straight-through cables.
   - **Switch0** → connect to **Router0 Gig0/2** via copper straight-through cable.
   - **Router0 Gig0/1** → connect to **Switch1** (right switch) via copper straight-through cable.
   - **PC3, PC4, PC5** → connect to **Switch1** via copper straight-through cables.
4. Wait for all link lights to go green.

Reference layout (from the doc):

| Left side (192.168.10.0/24) | Right side (192.168.20.0/24) |
|---|---|
| PC0 (.1), PC1 (.2), PC2 (.3) → Switch0 → Router0 Gig0/2 (.100) | Router0 Gig0/1 (.100) → Switch1 → PC3 (.3), PC4 (.2), PC5 (.1) |

---

## Step b — Assign hostnames

**Where:** Click each device (Router0, Switch0, Switch1) → **CLI** tab.

1. Click **Switch0** → **CLI** tab. Press Enter to get a prompt if blank.
2. Type, line by line:
   ```
   enable
   configure terminal
   hostname <name you want, e.g. Switch-Left>
   ```
3. Repeat on **Switch1** (e.g. `hostname Switch-Right`).
4. Repeat on **Router0** (e.g. `hostname Router0` or a name of your choice).

*(No formal Documentation Task for this step — just do it, it's required for later steps to make sense when reading `show` command output.)*

---

## Step c — Configure the PCs

**Where:** Click each PC → **Desktop** tab → **IP Configuration** (or **Config** tab → FastEthernet).

Using the addressing table below, set **IP Address** and **Subnet Mask** only — **do not set a default gateway yet**, the doc wants you to prove *why* it's needed later.

| PC | IP Address | Subnet Mask |
|---|---|---|
| PC0 | 192.168.10.1 | 255.255.255.0 |
| PC1 | 192.168.10.2 | 255.255.255.0 |
| PC2 | 192.168.10.3 | 255.255.255.0 |
| PC3 | 192.168.20.3 | 255.255.255.0 |
| PC4 | 192.168.20.2 | 255.255.255.0 |
| PC5 | 192.168.20.1 | 255.255.255.0 |

Do this for all 6 PCs.

---

## Step d — Test connectivity within each network

**Where:** PC → **Desktop** → **Command Prompt**.

1. Click **PC0** → **Desktop** → **Command Prompt**.
2. Run:
   ```
   ping 192.168.10.2
   ```
   (pings PC1 — same network, should succeed since no router is needed yet)
3. Click **PC3** → **Desktop** → **Command Prompt**.
4. Run:
   ```
   ping 192.168.20.2
   ```
   (pings PC4 — same network, should also succeed)

### 📸 Task 1 — write this up

- Screenshot **both** Command Prompt windows showing the ping output (PC0→PC1 and PC3→PC4).
- From the PC0→PC1 output, read the summary line (`Packets: Sent = X, Received = Y, Lost = Z`) and record:
  - Total packets **sent**
  - Total packets **received**
  - Total packets **lost**

---

## Step e — Test connectivity across networks (before gateway is set)

**Where:** Command Prompt on PC0.

1. Click **PC0** → **Desktop** → **Command Prompt**.
2. Run:
   ```
   ping 192.168.20.3
   ```
   (this is PC3, on the *other* network)
3. Observe the result — note whether it succeeds or fails.
4. **Ask your TA** to explain why you got that result before moving on (this is a deliberate discussion point in the lab, not something to just look up — come with your own theory about why it behaved that way, based on what a default gateway does).

### Now set default gateways

5. Go back to each PC's **IP Configuration** and now fill in the **Default Gateway** field:
   - PC0, PC1, PC2 → gateway `192.168.10.100`
   - PC3, PC4, PC5 → gateway `192.168.20.100`

### Re-test

6. On PC0's Command Prompt, run `ping 192.168.20.3` again.
7. Note the result again — likely still fails at this point (the router's interfaces aren't configured/active yet). **Ask your TA** to explain why, before proceeding to step f.

---

## Step f — Configure router interfaces

**Where:** Router0 → **CLI** tab.

1. Click **Router0** → **CLI** tab.
2. First, check the current (blank) state:
   ```
   enable
   show ip interface brief
   ```
   You should see all interfaces `unassigned` and `administratively down`.
3. Configure the interface facing PC0–PC2 (Gig0/2):
   ```
   configure terminal
   interface g0/2
   ip address 192.168.10.100 255.255.255.0
   no shutdown
   ```
4. Configure the interface facing PC3–PC5 (Gig0/1):
   ```
   interface g0/1
   ip address 192.168.20.100 255.255.255.0
   no shutdown
   ```

**Note:** router interfaces are shut down (disabled) by default — `no shutdown` is what turns them on. Without it, the interface stays `administratively down` even with an IP address assigned.

---

## Step g — Check interfaces and routing table

**Where:** Router0 CLI, back at the privileged `Router#` prompt (type `exit` if you're still in interface/config mode, or prefix commands with `do` if you stay in config mode).

1. Run:
   ```
   show ip interface brief
   ```
2. Run:
   ```
   show ip route
   ```

### 📸 Task 2 — write this up

- Screenshot **both** command outputs.
- Answer:
  - What does **up/up** mean in the `show ip interface brief` output? (Look at the two status columns — one is the physical/line-protocol layer, one is the administrative state — think about what each layer being "up" confirms is working.)
  - What does the **`C`** next to a route mean in the `show ip route` output? (Check the legend printed at the top of the `show ip route` output in Packet Tracer/Cisco IOS — it lists what each letter code stands for.)

---

## Step h — Save the configuration

**Where:** Router0 CLI (and do the same on both switches, good practice).

Run either:
```
copy running-config startup-config
```
or the shorthand:
```
write
```
Press Enter to confirm the filename prompt. This saves your config to NVRAM so it survives a reload — router/switch changes are otherwise only in running memory and are lost on restart.

---

## Step i — Test connectivity across networks (again)

**Where:** Command Prompt on PC0, then PC3.

1. Click **PC0** → **Desktop** → **Command Prompt**. Ping across the router:
   ```
   ping 192.168.20.3
   ping 192.168.20.2
   ping 192.168.20.1
   ```
   (PC3, PC4, PC5)
2. Click **PC3** → **Desktop** → **Command Prompt**. Ping back the other way:
   ```
   ping 192.168.10.1
   ping 192.168.10.2
   ping 192.168.10.3
   ```
   (PC0, PC1, PC2)
3. The doc notes pings "should succeed (eventually)" — the very first ping after everything comes up may show one timeout (ARP resolving) before the rest succeed; that's normal.

### 📸 Task 3 — write this up

- Ping **PC4 from PC0** (`ping 192.168.20.2` from PC0's Command Prompt).
- Screenshot the output.
- From the summary line, record:
  - Total packets **sent**
  - Total packets **received**
  - Total packets **lost**

---

## Step j — Troubleshoot failed pings (if anything above didn't work)

Work through this checklist in order:

1. **Router interfaces UP/UP?** On Router0 CLI: `show ip interface brief` — both Gig0/1 and Gig0/2 should show `up` / `up`. If not, re-check you ran `no shutdown` on the correct interface.
2. **PC gateways correct?** Check each PC's IP Configuration — gateway must be `192.168.10.100` for PC0–2, `192.168.20.100` for PC3–5, and must match the subnet the PC is actually in.
3. **Correct cable types?** Copper straight-through between PC↔Switch and Switch↔Router — Packet Tracer will show a link-down icon (not a solid green dot) on a bad connection.
4. **IP addresses match the addressing table?** Double-check each PC and router interface against the table in Step c/f — a typo'd octet is the most common cause of "same subnet but still fails."
5. **Router has both networks in its routing table?** Router0 CLI → `show ip route` — you should see both `192.168.10.0/24` and `192.168.20.0/24` listed as directly connected (`C`). If one is missing, the corresponding interface is either not configured or still administratively down (`no shutdown` not applied).
