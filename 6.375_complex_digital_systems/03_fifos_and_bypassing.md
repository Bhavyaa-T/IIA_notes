# 03 — FIFOs, Bypassing and Same-Cycle Communication (L06)

> **Big idea:** with plain registers, a write is only visible **next cycle**. To let a dequeue free a slot for an enqueue in the *same* cycle (or let data pass straight through), you need **bypass paths**. Where you put them decides latency, throughput and the combinational paths that cross module boundaries.

---

## 1. Why a naive FIFO halves throughput

A 1-element FIFO built from a data register plus a full flag:

- `enq` is allowed only if **empty**, and `deq` only if **full**.
- These two can never both be true, so a stage cannot enq and deq in the same cycle.
- In an elastic pipeline, neighbouring stages alternate: **50% throughput**.

A 2-element version allows simultaneous enq/deq when it holds exactly one item. That fixes throughput but costs an extra entry.

---

## 2. EHR: a register with ordered internal bypass ports

The course's fix is the **Ephemeral History Register (EHR)**, a register with ports 0, 1, 2…, ordered within a cycle:

- Read port *i* sees the value written by any **lower-numbered** write port in the same cycle.
- A higher-numbered write wins.
- The value stored at the clock edge is the last write.

In RTL this is **just a chain of bypass muxes in front of a flip-flop**:

```systemverilog
// 2-port EHR: port 0 happens before port 1 within a cycle
assign r0   = q;                    // sees the value from the start of the cycle
assign r1   = w0_en ? w0_data : q;  // sees port 0's write this cycle
assign next = w1_en ? w1_data : r1; // port 1 wins
always_ff @(posedge clk) q <= next;
```

"Port 0 before port 1" is the whole concept: it fixes an **order** for operations that happen in the same cycle.

---

## 3. The three FIFO flavours (very common interview topic)

| FIFO | Special case allowed | Order in the cycle | Combinational path created | Min. entries for full throughput |
|---|---|---|---|---|
| **Pipeline** | enq when **full**, if a deq happens in the same cycle | deq → enq | **Backwards**: consumer's `ready` → producer's `ready` | 1 |
| **Bypass** | deq when **empty**, if an enq happens in the same cycle | enq → deq | **Forwards**: producer's `data/valid` → consumer | 1 (0-cycle latency) |
| **Conflict-free** | enq and deq together only if neither empty nor full | independent | **None**: both sides are registered | 2 |

```systemverilog
// 1-entry PIPELINE FIFO: full throughput, ready path is combinational
assign deq_valid = full;
assign deq_data  = d;
assign enq_ready = !full || deq_ready;       // deq_ready -> enq_ready (backwards)

always_ff @(posedge clk) begin
  if (!rst_n)                     full <= 1'b0;
  else if (enq_valid && enq_ready) full <= 1'b1;   // also covers enq+deq together
  else if (deq_valid && deq_ready) full <= 1'b0;
  if (enq_valid && enq_ready) d <= enq_data;
end
```

```systemverilog
// 1-entry BYPASS FIFO: zero latency, data path is combinational
assign enq_ready = !full;
assign deq_valid = full || enq_valid;        // enq -> deq (forwards)
assign deq_data  = full ? d : enq_data;

wire enq_fire = enq_valid && enq_ready;
wire deq_fire = deq_valid && deq_ready;
always_ff @(posedge clk) begin
  if (!rst_n)                    full <= 1'b0;
  else if (enq_fire != deq_fire) full <= enq_fire; // pass-through leaves it empty
  if (enq_fire && !deq_fire) d <= enq_data;
end
```

**Trade-off to articulate.** Chaining many pipeline FIFOs builds a long combinational `ready` chain from the back of the pipe to the front. Chaining bypass FIFOs builds a long forward data path. Conflict-free FIFOs (the same idea as a **skid buffer** / fully registered valid-ready slice) break both chains, which is why they are used to **close timing between modules**, at the cost of one extra entry.

---

## 4. Register files: normal vs bypass

| | Write visible | Order in the cycle | Use |
|---|---|---|---|
| **Normal** | Next cycle | reads → write (read the old value) | Default |
| **Bypass** | Same cycle | write → reads (read the new value) | Removes one stall in a CPU pipeline (write-back → decode forwarding) |

```systemverilog
// Bypass register file read port
assign rd1_data = (wr_en && wr_idx == rd1_idx) ? wr_data : rf[rd1_idx];
```

The same effect can be built **outside** a normal register file by putting a small bypass FIFO in front of it and searching that FIFO on every read.

---

## 5. N-entry circular FIFO: the pointer trick

Use `enqP`/`deqP` pointers into an array of N entries. **Empty** and **full** both look like `enqP == deqP`, so add **one extra pointer bit** (count modulo 2N and index the array modulo N):

```systemverilog
localparam AW = $clog2(N);              // N a power of 2
logic [AW:0] wptr, rptr;                // one extra bit
assign empty = (wptr == rptr);
assign full  = (wptr[AW] != rptr[AW]) && (wptr[AW-1:0] == rptr[AW-1:0]);
// mem[wptr[AW-1:0]] <= enq_data;   deq_data = mem[rptr[AW-1:0]];
```

To make an N-entry FIFO conflict-free, the course's design lets each side do at most one operation per cycle and recomputes the full and empty flags from both pointers at the **end** of the cycle.

---

## Interview takeaways

- Same-cycle communication between modules needs a combinational path. Decide deliberately which direction it goes.
- **Pipeline FIFO = deq before enq**, **bypass FIFO = enq before deq**, **CF FIFO = independent, 2 entries, no comb paths**.
- Bypass register file = forwarding. The extra-MSB pointer trick is a classic question.
