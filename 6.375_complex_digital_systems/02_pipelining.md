# 02 — Pipelining: Inelastic vs Elastic (L05)

> **Big idea:** put registers between the stages of a combinational circuit so a new input can enter every cycle. Throughput becomes about 1 result per cycle and the clock is set by the **slowest stage**. The key design choice is how stages coordinate.

---

## 1. Two styles

| | **Inelastic** (synchronous) | **Elastic** |
|---|---|---|
| Between stages | Pipeline registers (+ valid bits) | FIFOs |
| Who decides movement | One global controller: all stages advance together | Each stage locally: advance if input valid **and** output ready |
| Pros | Designer controls exactly what happens each cycle | Small, independent stages; easy to change and extend |
| Cons | One big controller: easy to get wrong, hard to modify | Stages may not overlap if the FIFOs are wrong (see 03) |
| Stall behaviour | Whole pipe freezes | Back-pressure propagates stage by stage |

Real processor pipelines mix the two styles.

---

## 2. Inelastic pipeline and bubbles

The naive version only advances when the input is non-empty **and** the output is non-full. That is a bug: if the input queue runs dry, data already in the pipe is **stuck**, even though the output could accept it.

**Fix: valid bits.** Let the pipeline advance even with no new input and carry a *bubble* (valid = 0), and only write the output when the last stage is valid.

```systemverilog
// 3-stage inelastic pipeline: f0 -> reg -> f1 -> reg -> f2
always_ff @(posedge clk) begin
  if (!rst_n) begin
    v1 <= 1'b0;  v2 <= 1'b0;
  end else if (out_ready) begin          // whole pipe advances in lock-step
    v1 <= in_valid;  s1 <= f0(in_data);  // empty input -> bubble
    v2 <= v1;        s2 <= f1(s1);
  end
end

assign in_ready  = out_ready;            // stall everything if output is full
assign out_valid = v2;                   // never emit a bubble
assign out_data  = f2(s2);
```

A valid bit plus data is the hardware version of an optional value (`Maybe`/`std::optional`): **data is meaningless unless its valid bit is set.**

---

## 3. Elastic pipeline

Each stage is an independent piece of logic:

> **stage *k* fires when** `in_valid && out_ready` **→** it dequeues its input, applies f, and enqueues the result.

This is exactly the **valid/ready handshake** (AXI-Stream style). A transfer happens on a cycle where `valid && ready`.

**The catch:** for all stages to fire **in the same cycle**, every FIFO must support **enq and deq in the same cycle**. If a FIFO can only do one or the other, neighbouring stages alternate and throughput halves. This is the motivation for L06 (see [03](03_fifos_and_bypassing.md)).

---

## 4. The correctness model: atomic, guarded operations

The course models hardware as a set of **guarded atomic actions**. Each action is a block of state updates with a firing condition, much like one enabled branch of an `always_ff`.

- **Atomic:** if it fires, *all* its updates happen; otherwise *none* do. A stage must never dequeue its input without also enqueuing its output.
- **Guard** (e.g. FIFO not empty / not full): if it is false, the whole action is blocked.
- An **`if` inside the action** is different. It only switches off the updates inside the `if`, and the rest of the action still happens. Interview point: **"is this a stall condition or a select condition?"**
- **Implicit conditions** (such as "the FIFO I'm enqueuing into must not be full") propagate up and block the entire action. A conditional enqueue makes the stall condition conditional too: `!p || notFull`.

**One-at-a-time semantics.** The legal behaviours of the system are exactly those you get by firing one action at a time, in some order. The hardware's job is to fire **as many as possible in the same cycle** while giving a result equivalent to *some* serial order. How to decide that is covered in [05](05_concurrency.md).

> If pipeline stages never fire in the same cycle, it isn't really a pipeline. If every enabled stage fires blindly in the same cycle, you can get wrong answers. You need a model in between, and that model is serialisability.

---

## Interview takeaways

- Throughput = 1/(slowest stage), and latency ≈ number of stages × clock period. Pipelining usually **increases** latency a little (register overhead).
- Inelastic: one global stall signal, with valid bits for bubbles. Elastic: local valid/ready, and the FIFOs decide whether you get full throughput.
- Know the difference between a **guard/stall** (blocks the whole stage) and an **if/select** (affects only part of it).
