# 04 — Case Study: IP Lookup in a Router (L07)

> **Big idea:** a realistic block with **variable-latency, memory-bound** work. Getting it functionally right is easy. Getting full throughput depends on subtle same-cycle concurrency (**dead cycles**). And the choice of micro-architecture matters far more than the choice of HDL.

---

## 1. The problem

- A router forwards each packet based on the **longest prefix match (LPM)** of its destination IP against a routing table.
- Line rate for 10 GbE is about **15 Mpackets/s**, i.e. one lookup every **~67 ns**. Packet **order must be preserved**.
- The table is stored as a **multi-level trie** in SRAM, indexed by 16 bits, then 8 bits, then 8 bits of the address. Each level either returns a result (a leaf) or a pointer to the next level.
- So each lookup takes **1 to 3 dependent memory reads**, and each read has a multi-cycle latency.

```c
p = RAM[ip[31:16]];               if (isLeaf(p)) return value(p);
p = RAM[ptr(p) + ip[15:8]];        if (isLeaf(p)) return value(p);
p = RAM[ptr(p) + ip[7:0]];         return value(p);
```

> ⚠️ Slide typo: it says "1/15 s or 67 ns". It means 1/15 **µs**.

---

## 2. Three architectures

| Architecture | Idea | Trade-off |
|---|---|---|
| **Rigid / static pipeline** | A fixed slot per level, whether or not it is used | Simple, but **wastes memory bandwidth** (~64% utilisation) |
| **Linear pipeline** | One stage per level, with a memory-port replicator to share the RAM | ~100% memory utilisation, but **largest area** |
| **Circular pipeline** | One RAM; unfinished lookups **recirculate** through it | ~100% utilisation at moderate area, but **the most complex control** |

**Synthesis results (0.18 µm):** static ≈ 2.3k gates, linear ≈ 15k, circular ≈ 8k. Hand-written Verilog and the higher-level HDL came within ±8% in area and speed for every architecture.

> **"Architecture differences are much more important than language differences."** This is a good line for interviews.

---

## 3. The circular pipeline

```
 enter ──► RAM ──► leaf? ──yes──► completion buffer ──► getResult (in order)
            ▲        │no
            └─ FIFO ◄┘   (FIFO holds token + remaining IP bits while RAM is busy)
```

- **Completion buffer:** lookups finish **out of order**, because each one needs 1–3 passes. On entry each lookup gets a **token** (a slot in a circular buffer). Results are written into their slot when done, and leave only from the head, **in order**. This is the same idea as a CPU's **reorder buffer**.
- Running out of tokens is the natural **back-pressure**: entry stops when the buffer is full.
- The memory has a **request/response interface** (`req`, then later `resp valid` / `deq`). This makes the design **latency-insensitive**: it still works if the RAM latency changes.

---

## 4. Dead cycles: a performance bug, not a functional one

The **enter** operation and the **recirculate** operation both want the RAM request port and the FIFO enqueue port.

- As first written, "recirculate" is one block that either sends a result to the completion buffer (**exit**) *or* re-issues a RAM request. Because it *might* use the RAM port, it is treated as **always** conflicting with `enter`.
- So on a cycle where one lookup **exits**, a new one cannot **enter**. That cycle is dead: the RAM port sits idle even though nothing actually needed it.
- **Fix: split the operation by its condition.** Separate *exit* (leaf: write to the completion buffer) from *recirculate* (not a leaf: re-issue). Now *exit* and *enter* touch different resources and can share a cycle.
- They also need FIFOs that allow enq and deq in the same cycle, and completion-buffer operations (get token / put / get result) that can all happen in the same cycle (see [03](03_fifos_and_bypassing.md)).

**General lesson:** when one block has an if/else in which only one branch uses a shared resource, **split it**, so the conflict only applies when that branch actually runs. In RTL this means making the arbitration/grant condition depend on the real branch, not on "might use".

---

## Interview takeaways

- Variable-latency work + in-order output → **tags/tokens + reorder (completion) buffer**.
- Memory-bound design → measure **memory-port utilisation** and pick the architecture that keeps the port busy.
- Watch for **false conflicts** on shared resources. They cost throughput silently and no functional test will catch them.
- Decoupled request/response interfaces make designs **latency-insensitive** and composable.
