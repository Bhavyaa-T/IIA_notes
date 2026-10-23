# Rapid-Fire Interview Questions

Cover the answer and say it out loud. Answers are deliberately short. Expand on them in the interview.

---

### Area, speed, folding (01)

**Q: What sets the maximum clock frequency of a design?**
The longest register-to-register combinational path (the critical path), plus setup time and clock-to-Q.

**Q: Combinational 32-bit multiplier vs folded: compare area, latency and throughput.**
Combinational: 31 adders, very long path, 1 result per long cycle. Folded: 1 adder + registers/muxes, short path, ~32 cycles latency and 1 result per 32 cycles.

**Q: How do you avoid a big mux when indexing `a[i]` with a counter?**
Shift `a` each cycle and always read `a[0]`. Shifting costs no logic.

**Q: You fold a 3-stage FFT. Why is the area saving much less than 3×?**
The twiddle constants become stage-dependent, so constant multipliers turn into general multipliers. You also add muxes, a stage register and control.

**Q: How do you pick between combinational, pipelined, folded and super-folded designs?**
Find the throughput requirement, then pick the smallest design that meets it at an acceptable clock.

### Pipelining (02)

**Q: Does pipelining reduce latency?**
No. It raises throughput. Latency usually rises slightly because of register overhead and uneven stages.

**Q: Inelastic vs elastic pipeline?**
Inelastic: global stall, all stages move together, valid bits mark bubbles. Elastic: FIFOs between stages, local valid/ready, stages move independently.

**Q: Why do you need valid bits in an inelastic pipeline?**
So the pipe can keep advancing when there is no new input. Otherwise data already in flight gets stuck, and the output can't tell real data from garbage.

**Q: Guard/stall vs `if`/select?**
A stall condition blocks the whole stage's update. An `if` only disables part of it.

### FIFOs and bypassing (03)

**Q: Why does a 1-entry "only-enq-when-empty" FIFO give 50% throughput?**
enq and deq are never both allowed, so neighbouring stages alternate.

**Q: Pipeline FIFO vs bypass FIFO vs conflict-free FIFO?**
Pipeline: can enq when full if deq happens in the same cycle, which creates a combinational path backwards on ready. Bypass: can deq when empty if enq happens in the same cycle, giving 0 latency and a forward combinational data path. CF: no combinational paths between sides, but needs 2 entries for full throughput.

**Q: What's a skid buffer and why use one?**
A 2-entry fully registered valid/ready slice (the CF FIFO idea). It breaks long combinational ready/data chains between modules, which helps timing closure.

**Q: How do you tell full from empty in a circular FIFO?**
Add an extra MSB to each pointer. Equal pointers mean empty. Pointers whose MSBs differ but whose lower bits match mean full.

**Q: What does a bypass register file do, and where is it used?**
A read of the register being written in the same cycle returns the new value. It's used for write-back → decode forwarding in CPU pipelines.

### IP lookup (04)

**Q: How do you keep outputs in order when operations have variable latency?**
Give each operation a tag/token on entry, and write results into a completion (reorder) buffer that drains in order. Running out of tokens provides back-pressure.

**Q: What's a dead cycle and how do you remove one?**
A cycle where a resource sits idle because of a false conflict. For example, "exit" and "enter" are treated as conflicting because exit is bundled with a branch that uses the RAM port. Fix it by splitting the operation by its condition, so the conflict only applies when that branch actually runs.

**Q: Why use a request/response memory interface?**
It makes the design latency-insensitive, so it keeps working when the memory latency changes.

**Q: Does the HDL matter for quality of results?**
Much less than the micro-architecture. Different HDLs came within ~10% of each other, while different architectures differed by up to 6×.

### Concurrency (05)

**Q: When is it safe for two operations to fire in the same cycle?**
When the combined effect equals running them one after the other in some order (serialisability).

**Q: `x <= y` and `y <= x` as two independent operations: can they share a cycle?**
No. Together they swap, which matches neither serial order.

**Q: In what order do a register's read and write happen within a cycle?**
Read before write: the read sees the old value.

**Q: Two modules exchange data through two FIFOs in a loop. When can both run every cycle?**
When the FIFO choices give a consistent order: one pipeline + one bypass, or at least one CF FIFO. Two pipelines or two bypasses create an ordering cycle (in RTL, potentially a combinational loop).

**Q: Is "A before B, B before C" enough to say "A before C"?**
No. Conflict relations aren't transitive, so check every pair.

### Caches (06)

**Q: 32 KiB, 4-way, 64 B lines, 32-bit addresses: how many tag, index and offset bits?**
Offset 6, sets = 32768 / (64 × 4) = 128 so index 7, tag = 32 − 7 − 6 = 19.

**Q: Why do caches work, and why use multi-word lines?**
Temporal locality (reuse) and spatial locality (neighbours). Lines exploit spatial locality, and moving a whole line at once is more efficient than moving single words.

**Q: Direct-mapped vs set-associative?**
Direct-mapped: one possible slot, simplest and fastest, but suffers conflict misses (worst case: stride equal to the cache size). N-way: N slots compared in parallel, giving fewer conflict misses but more comparators and muxing and a slightly slower hit.

**Q: What are the 3 Cs, and what fixes each?**
Compulsory (larger lines, prefetching), capacity (bigger cache), conflict (more associativity). Coherence misses are a fourth type in multicore systems.

**Q: L1 hit 1 cycle, 5% miss rate; L2 hit 10 cycles, 20% local miss rate; DRAM 100 cycles. What is the AMAT?**
1 + 0.05 × (10 + 0.2 × 100) = 2.5 cycles.

**Q: Write-back vs write-through?**
Write-back: update only the cache, set a dirty bit, write the line back on eviction, giving low traffic. Write-through: update the next level on every write, so it's always up to date but needs a write buffer.

**Q: What happens on a miss in a blocking write-back cache?**
Stop accepting requests. If the victim is dirty, write it back. Request the line, wait for the fill, install it and update the tag and valid bit. Then answer the load, or merge the store and set dirty, and return to ready.

**Q: What is a store buffer, and what must a load do with it?**
A FIFO of pending stores that drains into L1 in the background, so stores retire without waiting. A load searches it in parallel with L1 and takes the newest matching entry. The drain must be arbitrated against loads for the L1 arrays.

**Q: Blocking vs non-blocking cache? What is an MSHR?**
A non-blocking cache keeps serving hits and further misses while a miss is outstanding. An MSHR is a table entry tracking one outstanding line miss. Later misses to the same line merge into it. Responses can return out of order, so requests carry tags.

**Q: Why is a VIPT L1 often limited to page size × ways?**
So the index and offset bits come only from the page offset, which is the same in virtual and physical addresses. That lets the TLB lookup and cache indexing run in parallel without aliasing.

**Q: What are the MESI states?**
Modified (only copy, dirty), Exclusive (only copy, clean, can be written without asking), Shared (clean, other copies may exist), Invalid.
