# 06 — Caches and Realistic Memories (L12)

> **Big idea:** real memory is slow and has variable latency, unlike the "magic" memories that answer combinationally. A **cache** keeps recently used data close to the processor. The design questions are **where a line can go** (mapping), **what to evict** (replacement), **what happens on a write** (write policy), and **what the cache does while a miss is outstanding** (blocking vs non-blocking). Sections 1–5 follow the lecture. Section 6 adds standard interview material that the lecture doesn't cover.

---

## 1. Why caches work

| | Register file | SRAM cache | DRAM |
|---|---|---|---|
| Size | tiny | small | huge |
| Latency | < 1 cycle | 1–few cycles | ~100+ cycles |
| Bandwidth | highest (on-chip) | high (on-chip) | lowest (off-chip) |

- **Temporal locality:** an address used recently is likely to be used again soon, so **keep it**.
- **Spatial locality:** a neighbouring address is likely to be used next, so **fetch whole lines** (e.g. 4–16 words). Transferring a whole line is also more efficient than transferring single words.
- **Hit:** fast. **Miss:** go to the next level, which is slow.

---

## 2. Organisation: tag / index / offset

```
| tag | index | offset |
        │        └─ which byte within the line
        └─ which set
```

- **Direct-mapped:** each address can go in exactly **one** line. Simple and fast, but two hot addresses that map to the same index keep evicting each other. The worst case is an access stride equal to the cache size.
- **N-way set-associative:** each address can go in any of **N** lines in its set. All N tags are compared in parallel, which reduces **conflict misses** at the cost of comparators, a way-select mux and slightly longer hit time.
- **Fully associative:** a single set containing every line. This is usually only used for small structures such as TLBs.

**Sizing (practise until it's automatic):**

$$
\text{offset bits} = \log_2(\text{line bytes}),\quad
\text{sets} = \frac{\text{cache bytes}}{\text{line bytes}\times \text{ways}},\quad
\text{index bits} = \log_2(\text{sets}),\quad
\text{tag} = \text{addr bits} - \text{index} - \text{offset}
$$

*Example:* 32 KiB, 4-way, 64 B lines, 32-bit addresses gives offset = 6, sets = 128, index = 7, tag = 19. Each line also stores a **valid** bit and (for write-back) a **dirty** bit.

```systemverilog
localparam OFF = $clog2(LINE_BYTES);
localparam IDX = $clog2(NUM_SETS);
assign offset = addr[OFF-1:0];
assign index  = addr[OFF +: IDX];
assign tag    = addr[31 : OFF+IDX];
assign hit    = valid[index] && (tag_array[index] == tag);   // direct-mapped
```

---

## 3. Policies

**Replacement** (set-associative only, because direct-mapped has no choice): LRU (true LRU is expensive above ~4 ways, so tree pseudo-LRU is common), random (cheap and surprisingly good), or prefer a **clean** victim to avoid a write-back. The lecture's point: **you can only judge a policy with benchmarks.**

**Write policy:**

| | **Write-back** (+ write-allocate) | **Write-through** (often no-write-allocate) |
|---|---|---|
| On a write hit | Update the cache only and set **dirty** | Update the cache **and** the next level |
| On eviction | Write back if dirty | Nothing (memory is already up to date) |
| On a write miss | Fetch the line, then write it | Usually write straight to the next level |
| Traffic | Low | High (needs a write buffer) |
| Next level always up to date? | No | Yes (simpler coherence/recovery) |

The lecture designs a write-back, write-allocate, direct-mapped cache. It notes that this is similar to Intel's L1, while IBM and ARM use write-through.

---

## 4. A blocking cache: the miss-handling state machine

A **blocking** cache handles **one miss at a time** and stops accepting requests until it is resolved. A hit is answered immediately. A miss steps through:

```
READY ──miss──► START_MISS ──► SEND_FILL ──► WAIT_FILL ──fill arrives──► READY
              (write back victim   (request    (install line, answer the load
               if valid & dirty)    the line)   or merge the store, set dirty)
```

```systemverilog
typedef enum logic [1:0] {READY, START_MISS, SEND_FILL, WAIT_FILL} state_t;
state_t state;

assign req_ready = (state == READY);   // blocking: no new requests during a miss

always_ff @(posedge clk) begin
  if (!rst_n) state <= READY;
  else case (state)
    READY:      if (req_valid && !hit) begin miss_req <= req; state <= START_MISS; end
    START_MISS: if (!victim_dirty || mem_req_ready) state <= SEND_FILL;  // write-back first
    SEND_FILL:  if (mem_req_ready)  state <= WAIT_FILL;                  // request the line
    WAIT_FILL:  if (mem_resp_valid) state <= READY;                      // fill, respond
  endcase
end
```

- Put **FIFOs** on the memory request/response sides and on the hit response. The lecture prefers **conflict-free** FIFOs here: the extra cycle of latency costs little, and the cache becomes properly decoupled from both neighbours (see [03](03_fifos_and_bypassing.md)).
- In the processor, a cache turns memory into a stage with a **variable (1 to n cycle) latency**. FIFOs between pipeline stages absorb the variation, the same latency-insensitive idea as the request/response memory interface in [04](04_ip_lookup_case_study.md).

> ⚠️ **Terminology:** the slides call this state register "mshr". In real designs an **MSHR** (miss status holding register) is one entry in a *table* of outstanding misses in a **non-blocking** cache (section 6). Use the standard meaning in interviews.

---

## 5. Store buffer

A store returns no data to the processor, so it doesn't need to wait for the cache:

- Stores go into a small **store buffer** FIFO of (address, data) pairs and retire straight away. A full buffer stalls new requests.
- In the background, the oldest entry drains into L1. A drain that misses triggers the normal miss handling.
- **Loads search the store buffer and L1 in parallel.** If the store buffer hits, it **wins**, and it must return the **newest** matching entry (*store-to-load forwarding*).
- **Structural hazard:** the drain and an incoming load both want the L1 arrays in the same cycle. Something must arbitrate. The lecture gives the load priority and blocks the drain for that cycle, even when the load hits in the store buffer.

```systemverilog
// Store-to-load forwarding: scan oldest -> newest so the newest match wins
logic [$clog2(SB_N)-1:0] e;

always_comb begin
  sb_hit  = 1'b0;
  sb_data = '0;
  e       = '0;
  for (int k = 0; k < SB_N; k++) begin
    e = sb_rptr + k;                                      // oldest first (wraps)
    if (k < sb_count && sb_addr[e] == ld_addr) begin
      sb_hit  = 1'b1;
      sb_data = sb_data_q[e];
    end
  end
end
assign ld_data = sb_hit ? sb_data : l1_word;
```

The slides assume whole-word stores. Real buffers must also handle **partial overlaps** (e.g. a byte store followed by a word load), which usually means stalling until the store has drained.

> ⚠️ Slide bugs: the store-buffer drain code writes `missReq <= r` when no `r` exists in that code (it should be the store just dequeued), and uses `wOffset` without computing it. The state is called `WaitFillReq` on one slide and `WaitFillResp` on the next.

---

## 6. Beyond the lecture: what interviewers also ask

*None of this is on the slides. These are standard results added for interview prep.*

**The 3 Cs of misses**

| Miss type | Cause | Fix |
|---|---|---|
| **Compulsory** | First access to a line | Bigger lines, prefetching |
| **Capacity** | Working set larger than the cache | Bigger cache |
| **Conflict** | Too many hot lines map to one set | More associativity, better index hashing |

(In multiprocessors there is a fourth: **coherence** misses.)

**AMAT (average memory access time)**

$$
\text{AMAT} = t_\text{hit} + \text{miss rate}\times\text{miss penalty}
$$

With two levels: $\text{AMAT} = t_{L1} + m_{L1}\,(t_{L2} + m_{L2}\,t_\text{mem})$. *Example:* 1 + 0.05 × (10 + 0.2 × 100) = **2.5 cycles**. Bigger and more associative caches lower the miss rate but raise the hit time, which is why caches come in levels.

**Line-size trade-off:** larger lines exploit spatial locality better but transfer more data per miss (longer miss penalty) and give fewer lines for a fixed capacity (more conflicts). Miss rate vs line size is U-shaped.

**Non-blocking caches**

- **Hit-under-miss / miss-under-miss:** keep serving requests while misses are outstanding.
- An **MSHR table** has one entry per outstanding line. A second miss to the same line is **merged** into the existing entry rather than sent to memory again.
- Responses can come back **out of order**, so each request carries a tag. This is the same idea as the completion buffer in [04](04_ip_lookup_case_study.md).
- The number of MSHRs limits memory-level parallelism.

**Cache coherence (multicore):** each core's private cache must agree on every line's value. **MESI** gives each line one of four states:

| State | Meaning |
|---|---|
| **M**odified | This cache owns the only copy, and it is dirty |
| **E**xclusive | This cache has the only copy, and it is clean, so it can be written without asking |
| **S**hared | Clean copies may exist in other caches |
| **I**nvalid | No valid copy here |

Coherence is implemented either by snooping a shared bus or with a directory. Store buffers also explain why x86 lets a load run ahead of an earlier store to a *different* address (TSO).

**Virtual memory interaction**

- A **TLB** caches virtual-to-physical translations.
- **VIPT** (virtually indexed, physically tagged) L1s do the TLB lookup and the cache indexing in parallel. To avoid aliasing, the index and offset bits must fall within the page offset, so **L1 size ≤ page size × ways** (e.g. 4 KiB × 8 = 32 KiB). This is a classic question.

**Bandwidth:** multiple ports are expensive, so caches are **banked** (interleaved by low address bits) to serve several accesses per cycle, at the risk of bank conflicts.

---

## Interview takeaways

- Be able to size the tag/index/offset bits and calculate AMAT on the spot.
- Know direct-mapped vs set-associative, write-back vs write-through, and blocking vs non-blocking (MSHRs).
- Store buffer = stores off the critical path. Loads forward from the **newest** match, and the drain must be arbitrated against loads.
- Caches create **variable latency**. Decouple them with FIFOs and request/response interfaces.
