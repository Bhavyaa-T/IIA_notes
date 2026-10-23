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
