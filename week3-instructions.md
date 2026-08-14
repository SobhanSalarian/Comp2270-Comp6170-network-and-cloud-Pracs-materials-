# Week 3 Prac — Switches, MAC Addresses and ARP

Step-by-step instructions for completing the blue Documentation Tasks in the Week 3 prac doc using Cisco Packet Tracer.

---

## Setup — Build the network (required before Task 1 & 2)

1. Open **Cisco Packet Tracer**.
2. In the bottom-left device palette, click **Switches** → drag one **Cisco 2960** onto the workspace.
3. Click **End Devices** → drag **two PCs** onto the workspace.
4. Rename the first PC (click its label, or right-click → rename) to something other than `PC0`. **Your setup:** the first PC was renamed to **`PC2`**. The second PC dragged in kept its default name, **`PC1`**.
   - ⚠️ Because the names ended up as PC2 and PC1 (not PC1/PC2 in drag order), the IP addressing below follows the **name**, not the order you added them — PC2 gets `.2`, PC1 gets `.1`. Double-check your own device names match before following the IPs below; rename here if not.
5. Click **Connections** → select the **copper straight-through cable** → click **PC2** → click a port on the switch → click **PC1** → click another port on the switch.
6. Wait for the link lights on the switch to turn **green** (may flash amber briefly first).

---

## Task 1 — Switch features

**Where:** Click the switch icon in the workspace → **Physical** tab.

### Identifying ports (FastEthernet vs uplink)

Don't try to read tiny labels off the image — get Packet Tracer to tell you directly:

**CLI method (most reliable):** Click the switch → **CLI** tab → run:
   ```
   show ip interface brief
   ```
   Prints every interface name and status as plain text.

### Checking LEDs (when they're too small to read visually)

The Physical-view image doesn't get sharper on zoom — that's a fixed-resolution limitation, not something you're doing wrong. Skip the visual LEDs and check status via CLI instead:
first enable it 

1. Switch → **CLI** tab.
2. Run:
   ```
   show interfaces status
   ```
   Shows every port's link status (connected/notconnect), speed, and duplex as text — equivalent to what the STAT/DUPLX/SPEED LEDs show, just readable.
3. For a single port's detail:
   ```
   show interface FastEthernet0/1
   ```

### What to write in your notebook

- A short list of switch features: port count/type (FastEthernet vs GigabitEthernet uplinks), LED indicators present (SYST, RPS, STAT, DUPLX, SPEED — meanings can be confirmed from Cisco's documentation online), power connector, console port.


---

## Task 2 — IP addresses & switch ports

**Where:** Click each PC → **Config** tab (or **Desktop → IP Configuration**).

1. Click **PC2** → **Config** tab → **FastEthernet** section.
2. Set:
   - **IP Address:** `192.168.1.2` (PC2 → x = 2)
   - **Subnet Mask:** `255.255.255.0`
   - Leave all other fields blank.
3. Click **PC1** → repeat with:
   - **IP Address:** `192.168.1.1` (PC1 → x = 1)
   - **Subnet Mask:** `255.255.255.0`
4. Back in the workspace, click each cable connection to confirm which switch port (e.g. Fa0/1, Fa0/2) each PC is plugged into.

### What to write in your notebook

- A small table: PC name | IP address | switch port connected to. Based on your setup:

  | PC name | IP address | Switch port |
  |---|---|---|
  | PC2 | 192.168.1.2 | *(fill in from step 4)* |
  | PC1 | 192.168.1.1 | *(fill in from step 4)* |

---

## Task 3 — Verify the network works

**Where:** PC → **Desktop** tab → **Command Prompt**.

1. Click **PC2** → **Desktop** → **Command Prompt**.
2. Type:
   ```
   ping 192.168.1.1
   ```
3. Confirm you get **Reply from...** messages (not "Request timed out").

### What to write in your notebook

- The tool used (`ping`, via Command Prompt) and the actual result/output observed.

---

## Task 4 — ipconfig /all

**Where:** Same Command Prompt window.

1. Type (note the space before `/all`):
   ```
   ipconfig /all
   ```
2. Scroll to the **Ethernet adapter** section.

### What to write in your notebook

- The useful fields: **IP Address**, **Subnet Mask**, **Physical Address (MAC)** — with a brief note on what each represents.

---

## Task 5 — ARP table

**Where:** Command Prompt on a PC.

1. Type:
   ```
   arp -a
   ```
2. Note what device(s) appear (likely little or nothing yet) — record this.
3. Ping the other PC (from PC2: `ping 192.168.1.1`; from PC1: `ping 192.168.1.2`).
4. Run `arp -a` again.
5. Also try `arp -?` to see the command's options.

# ARP Investigation in Packet Tracer
## Exercise 5 — ARP in Simulation Mode


### 1. Clear the ARP cache first (so you actually see ARP traffic)

If a PC already has the other PC's MAC cached, it won't send a new ARP request when you ping — you'll only see ICMP. Clear the cache first:

1. Click **PC2** → **Desktop** → **Command Prompt**.
2. Run:
   ```
   arp -d *
   ```
3. Do the same on **PC1** (Command Prompt → `arp -d *`).

### 2. Switch to Simulation Mode

1. Look at the bottom-right corner of the Packet Tracer window — there are two mode buttons: **Realtime** and **Simulation**.
2. Click **Simulation**. A new **Simulation Panel** opens on the right showing an **Event List** (empty for now) and playback controls (**Auto Capture / Play**, **Capture / Forward**, **Reset Simulation**).
3. Optional but recommended: click **Edit Filters** in the Simulation Panel → **Show All/None** → tick only **ARP** and **ICMP** so the Event List isn't cluttered with other protocols.

### 3. Generate traffic using ping

1. Click **PC2** → **Desktop** → **Command Prompt**.
2. Run:
   ```
   ping 192.168.1.1
   ```
3. Switch to the Simulation Panel and click **Auto Capture / Play** (or **Capture / Forward** to step through one event at a time).
4. Watch envelope icons animate between PC2 → Switch → PC1, and rows appear in the **Event List** below.

### 4. Observe ARP packets in the Event List

- The first envelopes you see should be **ARP** (Device = PC2, then the switch, then PC1) — this is the ARP request/reply happening *before* the ICMP ping packet, since PC2 needs PC1's MAC address first.
- After the ARP exchange completes, you'll then see **ICMP** envelopes for the actual ping.
- Each row in the Event List shows: **Time**, **Last Device**, **At Device**, **Type** (colour-coded — ARP is usually a different colour from ICMP).

### 5. Click on packets to inspect details

1. In the Event List, click the coloured square in the **Type** column for an **ARP** event (or click the envelope icon on the device in the topology). This opens the **PDU Information** window.
2. In that window:
   - **OSI Model tab** → look at **Layer 2** → shows the **Source MAC** and **Destination MAC** for that hop. For an ARP *request*, the destination MAC will be the broadcast address `FFFF.FFFF.FFFF`.
   - **Inbound PDU Details / Outbound PDU Details tabs** → scroll to the **ARP** section → shows whether it's an **ARP Request** (`Opcode: REQUEST`, asking "who has this IP?") or an **ARP Reply** (`Opcode: REPLY`, "this is my MAC").
3. Click through consecutive events at each device (PC2 → Switch → PC1 → Switch → PC2) to see the request go out and the reply come back, and note how the switch just forwards/floods it rather than processing IP.
4. Once you've stepped through the ARP exchange, keep clicking **Capture / Forward** to also inspect the following ICMP (ping) packets the same way, if you want to compare Layer 2 vs Layer 3 headers.

### What to notice (talk about with your demonstrator, no formal write-up required)

- The ARP Request is sent to the broadcast MAC `FFFF.FFFF.FFFF`, but the ARP Reply is sent directly (unicast) back to the requester's MAC.
- The switch does not generate or process ARP itself — it just forwards frames based on MAC address, flooding the broadcast request out all ports (except the one it came in on).
- If you ping again without clearing the ARP cache, you should see **no ARP packets** — only ICMP — since the MAC is now cached.
