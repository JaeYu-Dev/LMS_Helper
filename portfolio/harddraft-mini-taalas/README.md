# HardDraft / Mini-Taalas LLM Accelerator

> A staged hardware–software project for building a practical, low-latency LLM inference accelerator, inspired by Taalas-style model specialization.

## Status

**Planned portfolio project** — research and implementation roadmap saved for future development.

## Core idea

Instead of trying to manufacture a GPT-3-scale model on one personal ASIC, this project targets a realistic hybrid architecture:

- a small quantized Llama/Qwen-family draft model,
- FPGA-based full-system prototyping with external DDR,
- dedicated acceleration for INT4 matrix multiplication, RMSNorm, RoPE and quantization,
- speculative decoding with a larger target LLM,
- eventual tape-out of the highest-value accelerator blocks,
- a custom PCB combining the host SoC/FPGA, memory and custom ASIC.

The primary optimization target is not raw `tokens/s`, but **accepted tokens/s**, energy per accepted token and total inference cost reduction.

## Proposed architecture

```text
User request
    |
    v
+-----------------------------+
| Host CPU / FPGA runtime     |
| - tokenizer                 |
| - scheduler                 |
| - KV-cache control          |
| - target-model integration  |
+---------------+-------------+
                |
                v
+-----------------------------+
| HardDraft accelerator       |
| - quantized draft model     |
| - INT4 x INT8 MAC array     |
| - RMSNorm / RoPE            |
| - quantization engine       |
| - optional fixed backbone   |
| - programmable LoRA/delta   |
+---------------+-------------+
                |
                v
+-----------------------------+
| Large target LLM            |
| - verifies draft tokens     |
| - preserves output quality  |
+-----------------------------+
```

## Long-term Mini-Taalas direction

The longer-term design combines:

1. **Fixed backbone** — stable quantized weights mapped into ROM or constant-coefficient hardware.
2. **Programmable delta** — LoRA/adapters stored in writable memory.
3. **State engine** — KV-cache movement, compression and scheduling.
4. **Speculative interface** — optimizes the number of accepted draft tokens delivered to the target model.

This preserves some of the efficiency of a hard-wired model while reducing the model-obsolescence problem of a completely fixed ASIC.

## Development roadmap

### Phase 0 — Modeling and feasibility

- Benchmark candidate draft models such as small Qwen/Llama derivatives.
- Quantize weights to INT4, INT2 and experimental binary/ternary formats.
- Measure draft acceptance rate against larger target models.
- Build a roofline model for memory bandwidth, compute and KV-cache traffic.

### Phase 1 — Software reference implementation

- Implement speculative decoding end to end.
- Add telemetry for acceptance rate, accepted tokens/s and latency.
- Compare standard GEMM against fixed-weight and fused-operation kernels.
- Establish exact numerical reference tests for hardware validation.

### Phase 2 — FPGA prototype

- Use external DDR for model weights and KV state.
- Implement an INT4/INT8 matrix engine.
- Add fused RMSNorm, RoPE, activation and requantization blocks.
- Integrate the accelerator with a host CPU or embedded SoC.
- Measure throughput, board power and deterministic latency.

### Phase 3 — Fixed-backbone experiment

Compare three implementations of the same small layer or model:

1. weights read from writable memory,
2. weights stored in ROM,
3. weights synthesized as constants or fixed interconnect.

Measure:

- LUT/gate area,
- SRAM/BRAM usage,
- routing congestion,
- maximum clock frequency,
- energy per inference,
- model update cost.

### Phase 4 — ASIC block tape-out

Tape out a verified subset rather than the full language model:

- INT4 x INT8 MAC array,
- accumulation and requantization,
- fused normalization/rotation unit,
- compact command and DMA interface.

The first silicon objective is a functional LLM inference accelerator block, not a standalone GPT-class chip.

### Phase 5 — Custom PCB

Create a board containing:

- host FPGA or SoC,
- external DDR/LPDDR where practical,
- custom accelerator ASIC,
- clock and power-management circuits,
- USB, PCIe or another host interface,
- measurement points for power and timing analysis.

## Evaluation metrics

| Category | Metric |
|---|---|
| Speculative decoding | Acceptance rate |
| Effective speed | Accepted tokens per second |
| Latency | Time to first token and inter-token latency |
| Efficiency | Joules per accepted token |
| Economics | Cost per accepted million tokens |
| Hardware | Area, clock, bandwidth and utilization |
| Quality | Equality or bounded divergence from target-model output |
| Flexibility | Adapter swap time and supported model families |

## Key technical risks

- Low draft-token acceptance can erase the speedup.
- External memory bandwidth can dominate compute performance.
- Fixed-weight logic may lose density due to routing congestion.
- Small draft models can be fast but poorly aligned with the target model.
- ASIC memory interfaces and packaging can be harder than the arithmetic core.
- Full-model hard-wiring creates rapid obsolescence when model architectures change.

## Definition of success

A successful first-generation result should demonstrate:

- a working quantized draft model on FPGA,
- measurable speculative-decoding acceleration against a target LLM,
- reproducible accepted-tokens/s and energy results,
- one verified custom accelerator block manufactured as real silicon,
- a custom PCB integrating the silicon into the full inference pipeline.

## Possible repository structure

```text
harddraft-mini-taalas/
├── README.md
├── docs/
│   ├── architecture.md
│   ├── feasibility.md
│   └── experiments.md
├── models/
│   ├── training/
│   ├── quantization/
│   └── export/
├── software/
│   ├── reference/
│   └── speculative-runtime/
├── rtl/
│   ├── mac-array/
│   ├── norm-rope/
│   └── control/
├── fpga/
├── asic/
├── pcb/
└── benchmarks/
```

## Current scope boundary

This project does **not** claim that an individual can reproduce a Taalas HC1-class, single-die Llama 8B accelerator. The portfolio goal is to reproduce and test the architectural principle at a realistic scale, then build a useful LLM-system accelerator whose value comes from low latency, low energy and speculative-decoding throughput.
