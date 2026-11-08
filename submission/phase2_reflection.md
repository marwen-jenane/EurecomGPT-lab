# Phase 2 — Reflection: where does parallelism stop helping?

Copy this file to `submission/phase2_reflection.md`, fill in every answer slot, and commit it:

```bash
cp phase-2-tiny-gpt/reflection_template.md submission/phase2_reflection.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
 the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
 grader finds your answers. Anything you write outside the markers is ignored, so add extra
 prose, tables or your threading plot freely.

Numbers in section 1 must **match `submission/phase2_report.json`** — copy them across, do not
retype from memory; they are checked against it.

---

## 1. Your measurements

| | Value |
|---|---|
| Naive attention, ms (`attention.naive_ms`) | <!--answer:attn_naive_ms-->269.435<!--/answer--> |
| Vectorized attention, ms (`attention.vectorized_ms`) | <!--answer:attn_vectorized_ms-->1.12<!--/answer--> |
| CPU training, ms for 100 steps (`devices.cpu_ms`) | <!--answer:cpu_ms-->468916.8<!--/answer--> |
| GPU training, ms for 100 steps (`devices.gpu_ms`) | <!--answer:gpu_ms-->14491.9<!--/answer--> |
| Final training loss (`training.final_loss`) | <!--answer:final_loss-->1.449<!--/answer--> |

**The thread sweep.** Give your four timings from `threads` — 1, 2, 4 and 8 threads — and say
which count was fastest. (verbatim: the numbers, in order)

<!--answer:thread_sweep-->
1 thread: 2456.562 ms; 2 threads: 2710.899 ms; 4 threads: 3032.42 ms; 8 threads: 2598.07 ms. The fastest configuration was 1 thread (2456.562 ms per step).
<!--/answer-->

---

## 2. Intra-node parallelism, three ways

**Why vectorizing wins.** Both attention implementations do the same arithmetic and produce the
same result (the unit test checks that). Explain where the naive version's time actually goes,
and what the vectorized one does differently — in terms of the machine, not the Python.
(~70 words)

<!--answer:vectorization_why-->
The naive attention implementation performs many small operations through Python loops. Much of its execution time is spent on repeated instruction dispatch, indexing, and intermediate calculations rather than efficient numerical computation. The vectorized implementation groups these operations into matrix multiplications handled by optimized native libraries such as OpenBLAS. These routines use SIMD instructions, cache-aware memory access, and efficient low-level loops. In my experiment, execution dropped from 269.435 ms to 1.12 ms, a 240.57x speedup.
<!--/answer-->

**Where the thread curve flattens, and why.** Using your sweep: more threads stopped helping —
or started hurting — at some point. Say where, and give the reason. Your Colab runtime's vCPU
count is part of the answer; so is what the threads have to coordinate on. (~80 words)

<!--answer:threading_plateau-->
The curve did not improve beyond one thread: 1 thread took 2456.562 ms, while 2 took 2710.899 ms and 4 took 3032.42 ms. At 8 threads, the time decreased to 2598.07 ms but was still slower than one thread. The Colab runtime has limited CPU resources, so adding workers can exceed the useful parallel capacity of its available vCPUs. Threads also compete for memory bandwidth and cache space and incur scheduling and synchronization overhead. For this workload, those costs outweighed the benefit of extra threads.
<!--/answer-->

**What made the GPU faster.** The model, the data and the step count were identical; only the
 device changed. Say what about a transformer training step suits a GPU, and name one workload
 that would *not* see this speed-up. (~70 words)

<!--answer:gpu_why-->
Transformer training involves many large matrix multiplications in attention and feed-forward layers, as well as parallel gradient calculations. A GPU can execute many arithmetic operations simultaneously, giving it much higher throughput on these workloads. For the same 100 training steps, the CPU took 468916.8 ms and the GPU took 14491.9 ms, about 32.36x faster. A very small, sequential task with frequent CPU-GPU data transfers would not necessarily benefit, because launching GPU kernels and moving data introduce overhead.
<!--/answer-->

---

## 3. The question

**Where does parallelism stop helping, and why?** Pull the three experiments together. Your
answer should distinguish the *kinds* of limit you met — work that cannot be divided, hardware
 you do not have, coordination that costs more than it saves, data that has to move — and say
 which one bound each experiment. One concrete number from section 1 per claim. (~150 words)

<!--answer:parallelism_limit-->
Parallelism stops helping when the extra work of coordinating or moving data becomes greater than the computation it saves, or when the hardware cannot execute more work concurrently. In the attention experiment, vectorization reduced execution time from 269.435 ms to 1.12 ms because the calculation could be expressed as efficient matrix operations. However, Python overhead and any inherently sequential parts cannot be accelerated indefinitely, as described by Amdahl's law. In the CPU threading sweep, one thread was fastest at 2456.562 ms; four threads increased the time to 3032.42 ms. This suggests that limited vCPU capacity, contention for memory and caches, and thread coordination outweighed additional parallel computation. The GPU performed much better on the transformer's matrix-heavy workload: 14491.9 ms versus 468916.8 ms on the CPU for 100 steps. Even there, speedup is limited by kernel-launch overhead, host-to-device transfers, and work that is too small or sequential to keep the GPU busy.
<!--/answer-->

**What you would measure next.** One experiment this phase did not run that would sharpen the
answer, and what result would change your mind. (~50 words)

<!--answer:next_experiment-->
I would repeat the CPU-versus-GPU benchmark across several batch sizes, recording training time per step and GPU utilization. I expect small batches to show less GPU advantage because launch and transfer overhead become more important. If the GPU remained equally faster even at very small batch sizes, I would reconsider how significant those overheads are for this model.
<!--/answer-->
