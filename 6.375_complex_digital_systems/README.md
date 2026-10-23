# MIT 6.375 Complex Digital Systems: Interview Prep Notes

MIT 6.375 (Arvind, Spring 2016), lectures L04–L08. The ideas covered are area/throughput trade-offs, pipelining, valid/ready handshakes, FIFOs, bypassing and same-cycle concurrency, all of which come up in RTL and micro-architecture interviews.

These notes stay **high-level**: concepts, trade-offs and likely interview questions. Code is in **SystemVerilog** and is only there to make an idea concrete. The course itself uses a different HDL (Bluespec), which these notes leave out. Source PDFs are in `source_slides/` (gitignored).

## Contents

| File | Lecture | Big idea |
|---|---|---|
| [01_folding.md](01_folding.md) | L04 | Trade time for area: reuse one block over many cycles |
| [02_pipelining.md](02_pipelining.md) | L05 | Trade registers for throughput: inelastic vs elastic pipelines |
| [03_fifos_and_bypassing.md](03_fifos_and_bypassing.md) | L06 | Pipeline / bypass / conflict-free FIFOs, bypass register file |
| [04_ip_lookup_case_study.md](04_ip_lookup_case_study.md) | L07 | Circular pipeline, completion buffer, dead cycles |
| [05_concurrency.md](05_concurrency.md) | L08 | When can two operations share a cycle? Conflict matrices |
| [interview_questions.md](interview_questions.md) | All | Rapid-fire Q&A for revision |

## The one-paragraph summary

A big combinational circuit is large and has a long critical path. **Folding** reuses one copy of the hardware over several cycles: less area, lower throughput. **Pipelining** puts registers between stages so several inputs are in flight at once: higher throughput for a small area cost. Pipelines are either **inelastic** (all stages advance together, and valid bits mark the bubbles) or **elastic** (stages are separated by FIFOs and each advances when its input is valid and its output is ready). An elastic pipeline only overlaps its stages if each FIFO can **accept and release data in the same cycle**, so FIFO design matters a lot. Pipeline, bypass and conflict-free FIFOs trade latency against combinational paths in different ways. Beneath all of this is one correctness rule: whatever happens in parallel in one cycle must have the same effect as doing the operations **one at a time in some order**.

## Recurring themes to bring up in interviews

1. **Area vs throughput vs latency.** Compare designs on all three. The IFFT table in 01 is a good concrete example.
2. **The critical path sets the clock.** Folding and pipelining both shorten it.
3. **Valid/ready handshakes** are just "FIFO not empty / FIFO not full".
4. **Bypassing buys latency with combinational path length.** A pipeline FIFO adds a backward path (ready), a bypass FIFO adds a forward path (data), and a conflict-free FIFO has neither but needs an extra entry.
5. **Parallel must equal some serial order** (serialisability, the same idea as in databases).
6. **Micro-architecture matters more than the HDL.** Different HDLs came within about 10% of each other, while different architectures differed by up to 6x in area (04).
