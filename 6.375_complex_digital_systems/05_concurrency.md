# 05 — Concurrency: When Can Two Operations Share a Cycle? (L08)

> **Big idea:** hardware runs many operations in the same clock cycle, but humans reason about them one at a time. Running a set of operations together is **legal only if the result equals running them in some sequential order**. That is **serialisability**, the same idea as database transactions. Each module's **conflict matrix** captures the ordering constraints, and they compose upwards.

---

## 1. The rule

- **Legal behaviour:** any state reachable by firing atomic operations **one at a time**.
- **Concurrent firing** of ops A and B in one cycle is allowed **iff** the result equals A-then-B **or** B-then-A, for every possible starting state.
- For *n* ops there must be **some permutation** that matches.

---

## 2. The swap example (great interview question)

```systemverilog
always_ff @(posedge clk) begin
  x <= y;
  y <= x;     // swap: perfectly legal as ONE operation
end
```

Now treat these as **two separate atomic operations**, `fr: x <= y` and `gr: y <= x`:

- fr then gr: both registers end up holding the old `y`.
- gr then fr: both end up holding the old `x`.
- Both in the same cycle, with registered semantics: they **swap**. That matches **neither** order.

So fr and gr **conflict** and must not fire in the same cycle, even though the hardware would happily do it. **Parallel hardware can produce results no sequential execution could.** This is the bug class that the analysis prevents.

---

## 3. The four relations between two operations f and g

| Relation | Meaning |
|---|---|
| `f < g` | Can share a cycle, and the effect is as if **f happened first** |
| `f > g` | Can share a cycle, as if **g happened first** |
| **CF** (conflict-free) | Can share a cycle in either order; they don't interact |
| **C** (conflict) | Cannot share a cycle |
| **ME** (mutually exclusive) | Never both enabled at once (e.g. `if (p)` / `if (!p)`), so ignore the pair |

**Primitives:**

- **Plain register:** **read < write**. A read in the same cycle sees the *old* value (non-blocking assignment semantics). Two writes **conflict**.
- **EHR / bypass register:** r0 < w0 < r1 < w1 … Each port fixes a position in the order (see [03](03_fifos_and_bypassing.md)).

**Deriving a module's matrix:** the relation between two methods is the **intersection** of the relations between every pair of lower-level calls they make (CF = {<, >}, C = {}). A compiler can build this bottom-up through the module hierarchy.

> Conflict relations are **not transitive**: A < B and B < C does not mean A < C. Check every pair.

**Scheduling theorem:** a set of operations can fire together if you can order them so that, for every pair in that order, the relation is `<`, CF or ME.

---

## 4. Applying it: FIFO choice decides whether two blocks can run together

Two blocks exchange data through two FIFOs: **A → B** (`aTob`) and **B → A** (`bToa`). Each block dequeues from one FIFO and enqueues into the other. Assume both FIFOs hold data.

- **Pipeline FIFO** (deq < enq): the *consumer* goes first.
- **Bypass FIFO** (enq < deq): the *producer* goes first.

| aTob | bToa | Required order | Same cycle? |
|---|---|---|---|
| CF | anything | at most one constraint | ✅ |
| pipeline | pipeline | B < A **and** A < B | ❌ cycle |
| bypass | bypass | A < B **and** B < A | ❌ cycle |
| pipeline | bypass | B < A, B < A | ✅ |
| bypass | pipeline | A < B, A < B | ✅ |

*(The slide poses this table as an exercise. The answers above are my own derivation.)*

**RTL intuition:** two pipeline FIFOs in a loop give a combinational **ready** loop, and two bypass FIFOs in a loop give a combinational **data** loop. In both cases there is no consistent "who goes first" (and in RTL, possibly a real combinational loop). **Feedback paths need at least one conflict-free (fully registered) FIFO, or a consistent ordering.**

---

## 5. How this shows up when writing SystemVerilog

SystemVerilog does not check any of this for you. The same reasoning appears as:

- "Does the consumer see this update **this cycle** (combinational/bypass) or **next cycle** (registered)?"
- "Do these two always-blocks or modules assume *opposite* orders?" If so, you get a combinational loop or a result that matches neither order.
- "Is this resource arbitrated on what is *actually* used, or on what *might* be used?" The latter causes dead cycles (see [04](04_ip_lookup_case_study.md)).

---

## Interview takeaways

- Correct concurrency = **equivalent to some serial order**.
- Register: read before write. Bypass: write before read. FIFO flavours are just choices of order.
- The swap example shows why "it all happens at the clock edge" isn't the end of the story.
- Feedback loops between modules need a consistent order, or a registered break.
