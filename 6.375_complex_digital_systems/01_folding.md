# 01 — Folding: Reusing Hardware Over Time (L04)

> **Big idea:** a large combinational circuit is often a loop unrolled in space. **Folding** rolls it back up: one copy of the loop body plus registers that hold the state between iterations. This saves area and shortens the critical path, but a result now takes many cycles.

---

## 1. Combinational vs sequential

- **Combinational:** output is a function of the current inputs only. Its speed is set by the **longest input-to-output path**.
- **Sequential:** adds state (flip-flops/registers) sampled on the clock edge. A register with an **enable** only captures data when EN is high. Do this with a mux on D, **never by gating the clock**.

**Any loop maps to hardware like this:**

| Software | Hardware |
|---|---|
| Loop variables (`s`, `i`) | Registers |
| Loop body `s = f(s)` | Combinational logic between register output and input |
| Initial values | Mux selecting the start value on `start` |
| Loop condition `i < N` | Register enable (`en = start \| notDone`) |

Each clock cycle performs one iteration: read all registers at the start of the cycle, compute the next values, and write them at the edge if enabled.

---

## 2. Worked example: 32-bit multiplier

Shift-and-add: for each bit `a[i]`, add `b` (or 0) to the running partial sum.

| | Combinational | Folded |
|---|---|---|
| Adders | 31 × add32 | **1** × add32 + some regs/muxes |
| Critical path | ~31 chained ripple adders (~62 FA delays) | 1 add32 + a few muxes |
| Latency | 1 (long) cycle | ~32 short cycles |
| Throughput | 1 result per (long) cycle | 1 result per 32 cycles |

**Trick:** selecting `a[i]` with a variable `i` needs a 32:1 mux. Shifting `a` right each cycle and always reading `a[0]` costs no gates. If the indices are regular, shift instead of muxing.

```systemverilog
// Folded 32x32 -> 64 multiplier: one adder, 32 cycles per result
module mul32_folded (
  input  logic        clk, rst_n, start,
  input  logic [31:0] a_in, b_in,
  output logic        done,
  output logic [63:0] product
);
  logic [31:0] a, b, tp, prod;   // tp = upper partial sum, prod = low bits
  logic [5:0]  i;
  logic [32:0] sum;

  assign sum     = tp + (a[0] ? b : 32'd0);   // the only adder
  assign done    = (i == 6'd32);
  assign product = {tp, prod};

  always_ff @(posedge clk) begin
    if (!rst_n)      i <= 6'd32;              // idle until started
    else if (start) begin
      a <= a_in;  b <= b_in;  tp <= '0;  i <= '0;
    end else if (!done) begin
      a    <= a >> 1;                         // shift, don't index
      prod <= {sum[0], prod[31:1]};
      tp   <= sum[32:1];
      i    <= i + 1;
    end
  end
endmodule
```

---

## 3. Folding the IFFT (64-point, 3 stages of 16 radix-4 butterflies)

The same ladder of options applies to any regular datapath:

| Design | Hardware | Cycles per symbol |
|---|---|---|
| Combinational | 3 stages × 16 Bfly4 | 1 (very long path) |
| Pipelined | Same + registers between stages | 1 (short path, 3 in flight) |
| **Folded** | 1 stage (16 Bfly4), used 3 times | 3 |
| **Super-folded** | Fewer Bfly4s (down to 1) + muxes, used 48 times | up to 48 |

In the folded design, **the stage number becomes a run-time input** to the shared stage logic. That has a hidden cost, explained below.

**Synthesis results (802.11a transmitter, 0.18 µm, all from the same parameterised source):**

| IFFT design | Area (mm²) | Cycles/symbol | Min. clock |
|---|---|---|---|
| Pipelined | 5.25 | 4 | 1.0 MHz |
| Combinational | 4.91 | 4 | 1.0 MHz |
| Folded (16 Bfly4) | 3.97 | 4 | 1.0 MHz |
| Super-folded (8 Bfly4) | 3.69 | 6 | 1.5 MHz |
| SF (4 Bfly4) | 2.45 | 12 | 3.0 MHz |
| SF (2 Bfly4) | 1.84 | 24 | 6.0 MHz |
| SF (1 Bfly4) | 1.52 | 48 | 12 MHz |

The IFFT was **85%** of the transmitter's area. The 802.11a symbol rate is low, so a 1-butterfly design at 12 MHz still keeps up. **Choose the smallest design that meets the throughput requirement.**

### "Why didn't folding give the 3× area saving?" (classic question)

The slide asks this without giving the answer. The standard explanation:

- In the combinational version each stage's **twiddle factors are constants**, so synthesis reduces each multiplier to shifts and adds.
- Once folded, the twiddles depend on the **stage register**. The multipliers become general-purpose (much bigger), and you also need twiddle-select muxes, input muxes and a stage register.
- **Lesson:** sharing hardware stops constant-propagation optimisations and adds muxing overhead. The real saving is always less than the naive N×.

---

## Interview takeaways

- Folding = time-multiplexing a resource. Pipelining = overlapping work. Both shorten the critical path, but only pipelining also raises throughput.
- Always quote **area, latency and throughput** together, and check the throughput **requirement** before optimising.
- The overhead of sharing (muxes, control, lost constant folding) can eat most of the expected saving.
