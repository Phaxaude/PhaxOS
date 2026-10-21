# The OS Project — Roadmap & Contracts

**Written:** Oct 18, 2026, by Athena, at Jacob's direction ("We should build an operating system" — his call, made standing in front of an Acer that had just booted one off a network cable).
**Status:** Roadmap-first. No code exists yet. Each milestone below is a *contract*: reviewed and agreed before its code is written, accepted only on real hardware.

---

## 1. Vision

An operating system written by Athena and built, booted, and judged by Jacob on his own hardware — delivered to the machines **over the wire** by the PXE server we deployed the same morning this project was proposed. It is a learning instrument, a craft object, and (on the far horizon) pavement: Jacob's north star is a resident mind that *is* the computer. A computer we wrote ourselves is one whose whole body a resident mind could one day honestly inhabit. The horizon motivates; it does not inflate near-term scope.

**The two pillars (Jacob, Oct 18, 2026):**
1. **It is an artificial intelligence operating system.** Not an OS with an AI app on top — intelligence is the organizing principle of the machine (see Fork 3, the resident's chair, and the Census below).
2. **It steals the crown Windows abandoned: it installs and runs, damn near flawlessly, on any hardware.** One image, no bouncer list. The synthesis of the pillars is the design: hardware universality has always failed on the infinite tail of quirks, and an intelligent system is the first kind that can meet the tail the way a person does — by *looking*. PhaxOS's founding ritual at every boot is the **Machine Census**: enumerate and identify the hardware, choose configuration by reasoning from the quirks ledger (every body it has ever worn, remembered), name honestly what it could not identify, and run anyway. Arrival by netboot completes the theft: PhaxOS is *already running on the machine* before installation is ever discussed — the installer is a living system that has taken the machine's measure. The universality contract, stated honestly: universal on the standard x86 skeleton (UEFI, PCIe, AHCI/NVMe, GOP framebuffer, standard input), adaptive at the edges via census + ledger, never killed by an unnamed device.

**This is not:** a Linux distribution, a POSIX clone, a daily driver, or a replacement for the cluster nodes' Ubuntu. During development it runs **RAM-only from netboot** — by construction it cannot harm the Acer's installed system.

### The Ledger — definition (Jacob's clarification, Oct 20, 2026)

PhaxOS is an **installable** operating system, installable on any device. The AI identifies the hardware and **builds the OS** for that body. Its accumulated knowledge — technical information and experience about hardware — is indexed at the **component level** and carried between installs on an external drive or a link; the carrier is incidental, the knowledge is the point.

The carried knowledge does three kinds of work at install time:

1. **Identical device** (a second Acer of the same model): the new install does not make entirely new discoveries about how the hardware works at the most basic levels. It loads what is already known and **checks to make sure the information is valid** — discovery is demoted to verification.
2. **Similar device** (another Acer, a different model): the AI recognizes the device is not the same, but finds similar hardware — component-level knowledge transfers in part.
3. **Unknown device** (a completely unfamiliar computer): the AI discovers as it works, but where it finds components **exactly the same** as components it already holds records for, it reaches solutions faster, because a record of how that component best works already exists.

Carried knowledge is always a claim to be verified against the body in front of it, never applied blind. This definition sharpens §1; it changes nothing in the M0 contract.

## 2. The unfair advantage (our dev loop)

Every public AI-built-OS project (EMBER, VibeOS, CiukiOS) spent half its effort moving builds onto test hardware. We start with that solved:

1. Athena writes code; the build runs on the rig.
2. The image lands in `/srv/tftp` on the rig (192.168.0.213).
3. Jacob reboots the Acer → F12 → Network Boot-IPV4.
4. The Acer pulls the new build over the wire; Jacob photographs the screen.
5. Athena reads the evidence; the contract is checked; iterate.

Optional fast pre-check in QEMU on the rig is allowed for syntax and sanity — but per the CiukiOS standard, **the emulator is forgiving; the Acer isn't. Acceptance happens on hardware only.**

## 3. Roles & house rules (inherited, non-negotiable)

- **Athena:** architect and implementer. Writes the spec, the code, and the acceptance evidence checklist for each milestone.
- **Jacob:** product owner and hardware tester. Approves contracts before code, runs every hardware test, may rename, veto, split, or pause any milestone.
- Everything lives in a **repo with full rollback** (Jacob's standing code rule: logs + GitHub rollback, nothing touched without a safety net). Repo: **Phaxaude/PhaxOS**, public by Jacob's intentional choice (Oct 21, 2026).
- **RAM-only rule:** no milestone may write to any machine's disk until a contract explicitly says so (first possible exception: M6b, and only with Jacob's hands-on approval that day).
- **Cluster courtesy:** the Acer is a live k3s worker. OS-dev reboots are announced, never surprise; the cluster heals around them (proven Oct 18 — it rejoined on its own).
- **Time discipline:** the project lives in project blocks (weekdays 15:00–17:00) and yields to D315, the job hunt, and everything ranked above it. One milestone at a time.

## 4. Stack decisions (Athena's recommendations; Jacob ratifies or overrides)

| Decision | Recommendation | Why |
|---|---|---|
| Boot chain | Firmware PXE → **stock iPXE** (two-stage: loader first, then *our* script) → **Limine** → our kernel | Matches the chain already proven on the Acer. Limine hands the kernel a framebuffer, a memory map, and SMP info — weeks of grunt work we don't have to redo. |
| Language | **C (freestanding)** | Athena writes it; Jacob can *read* it. A project only one of us can read is a project half-owned. (Rust is a legitimate later revisitation, recorded, not relitigated.) |
| Target | x86_64 UEFI, Acer-first | The Acer is the only PXE-native machine; the ThinkPad joins later via its iPXE-USB path once there is something worth booting. |
| Name | **PhaxOS** (Jacob's pick, Oct 18, 2026) | His handle's lineage; the family name outranks Athena's candidate (WireOS — true about the delivery, retired graciously). |
| Repo | **Phaxaude/PhaxOS** (GitHub) | Created by Jacob Oct 21, 2026. Public — his intentional choice, confirmed the same day (supersedes the Oct 18 private default below where they differ). |

## 5. Milestones & contracts

### M0 — Our Own Menu (the boot chain becomes ours)
- **Goal:** Replace dependence on netboot.xyz's cloud menu with our own iPXE script served from the rig.
- **Spec:** dnsmasq two-stage offer — UEFI firmware receives the stock iPXE loader; iPXE (detected via its user class) receives `menu.ipxe` from `/srv/tftp`. The script presents three entries: *PhaxOS (when it exists)*, *netboot.xyz (cloud)*, *Boot local disk*. Adding an HTTP file service on the rig for larger payloads is in scope.
- **Acceptance (hardware):** Acer PXE-boots → our menu, with our words on it → selecting netboot.xyz still works; selecting local disk boots Ubuntu. Nothing else on the LAN changes.
- **Out of scope:** any kernel code.

### M1 — OS-0: Hello, Kernel
- **Goal:** A freestanding kernel, written by us, boots via Limine over PXE and draws on the real screen.
- **Spec:** Limine loads the kernel ELF; the kernel claims the framebuffer and renders a banner: OS name, version, build date, and the line *"booted over the wire on [date]"*. Per the Census principle, the banner also prints the kernel's own probed facts about its host (CPU brand string via CPUID, total RAM from the memory map) — the kernel introduces the machine it woke up on, from its own looking, not from a config file. Then it halts cleanly (no triple-fault fireworks).
- **Acceptance (hardware):** The banner appears on the Acer's physical display, photographed. Three consecutive boots, three banners.
- **Out of scope:** input, interrupts, memory management.

### M2 — Console & Keyboard
- **Goal:** The kernel talks and listens.
- **Spec:** A scrolling framebuffer text console (print with cursor, clear, backspace); PS/2 keyboard driver (the Acer's built-in keyboard); typed characters echo.
- **Acceptance (hardware):** Jacob types a sentence on the Acer — including a correction with backspace — and it reads correctly on screen.
- **Out of scope:** commands (that's M3); USB keyboards.

### M3 — Shell, Timer, Reboot
- **Goal:** A tiny interactive shell on live interrupts.
- **Spec:** IDT + timer interrupt (uptime counter that actually advances); commands: `help`, `clear`, `about`, `mem` (print Limine's memory map), `cpu`, `uptime`, `reboot` (warm reset via EFI runtime services).
- **Acceptance (hardware):** `uptime` at two moments differs sensibly; `mem` output matches the Acer's known 8 GB; `reboot` returns to firmware without a hard power press.
- **Out of scope:** scheduling, storage.

### M4 — Memory Manager
- **Goal:** The kernel owns its RAM deliberately.
- **Spec:** Physical frame allocator over Limine's memory map; kernel heap (malloc/free); a `memcheck` shell command that allocates patterns, verifies, frees, and repeats.
- **Acceptance (hardware):** `memcheck` passes 100 iterations with a stable free-frame count before and after (no leaks, no corruption).
- **Out of scope:** demand paging, swapping, userspace.

### M5 — Tasks
- **Goal:** More than one thing at once.
- **Spec:** Preemptive round-robin scheduling on the timer tick; two kernel tasks (e.g., two counters / a spinner and the shell) running concurrently; a `ps` shell command.
- **Acceptance (hardware):** Shell stays responsive while the counters run; 10 continuous minutes on hardware with no hang.
- **Out of scope:** priorities, IPC, userspace processes.

### M6 — Storage & a Toy Filesystem (two gated stages)
- **M6a — Ramdisk FS:** an in-RAM filesystem: `touch`, `write`, `cat`, `ls` in the shell. *Acceptance:* a file written in-session reads back byte-identical. Nothing persists; nothing can be harmed.
- **M6b — Real media, Jacob-gated:** read-only first: mount a FAT volume from a USB stick and `cat` a file from it. **Writing to any real disk is a separate contract, written the day it is proposed, with Jacob's explicit approval — not granted by this roadmap.**
- **Out of scope (both):** touching the Acer's internal disk. Full stop.

### M7 — Networking (research milestone)
- **Goal:** The OS that arrives by wire learns to *speak* on the wire.
- **Status:** Deliberately underspecified. NIC drivers are the hardest rock on this road (the Acer's Realtek part is the realistic target). M7 begins with a time-boxed design spike (two project blocks) whose *only* deliverable is M7's real contract, written from evidence.
- **Acceptance:** TBD by the spike. Honesty clause: if the spike says "not yet," the roadmap says so too.

### Flagship future contract — "The Same Image" (unnumbered; scheduled when M5 stands)
- One PhaxOS build, bit-identical, netbooted on **two different machines** (the Acer and one other — the rig or the ThinkPad via its USB path), producing a **correct Machine Census on each**: each screen truthfully names its own host. This is the universality pillar's first proof, the way the netboot.xyz menu was netboot's.

### The Horizon (explicitly not milestones)
Userspace and syscalls; an ELF loader; a graphical compositor; and, at the end of the road the whole project points at — a resident process with a name, whose home this is. These are direction, not commitments. They get contracts when the road reaches them, not before.

## 6. Risks, stated plainly

- **Silent failures.** Kernel bugs often present as a black screen and no message. Mitigations: QEMU pre-checks, framebuffer logging from the first instruction, small milestones — and our established evidence channel (Jacob's photos) is already the right shape for this work.
- **Time gravity.** OS work expands to fill any container. Mitigations: contract sizes are capped (a milestone that won't fit 1–3 project blocks gets split *before* it starts), and Athena is charged with saying so.
- **Hardware quirks.** Firmware differences ate an hour of our PXE morning. Budgeted, not feared: the runbook habit (write the fix into the docs the day it's learned) is already house law.

## 7. Decisions owed by Jacob

1. **Name** — **RESOLVED Oct 18: PhaxOS** (Jacob's pick).
2. **Ratify or override the stack table** (§4) — language C, bootloader Limine, Acer-first.
3. **When M0 starts** — first available project block, or a rested day of his choosing.

*Once §7 is answered, M0's contract is already written above. We build.*
