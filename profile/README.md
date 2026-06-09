<div align="center">

<img src="banner.png" alt="LoopGain — cost control for AI agent loops" width="100%" />

AI agent loops waste time and money when they don't know when to stop. LoopGain measures the loop in real time and stops it the moment it has actually converged — and rolls back before it degrades — instead of running to a fixed `max_iterations` cap.

**[loopgain.ai](https://loopgain.ai)** · **[Benchmarks](https://loopgain.ai/benchmarks)** · **[Dashboard](https://dashboard.loopgain.ai)** · **[PyPI](https://pypi.org/project/loopgain/)**

</div>

---

## The problem

Verify-revise loops — agentic coding, self-refinement, ReAct — have no real stopping signal, so they run to a guessed `max_iterations`. Set it too low and you cut the loop off before it's done. Set it too high and you burn tokens and wall-clock on iterations that aren't improving anything — or, worse, that quietly degrade a correct answer back into a broken one. `max_iterations=N` is just a guess, and the loop has no idea which iteration was its best.

## What LoopGain does

LoopGain watches each loop's error trajectory and classifies it live into five named states — `FAST_CONVERGE`, `CONVERGING`, `STALLING`, `OSCILLATING`, `DIVERGING` — then acts on that signal:

- **Stops** the loop once it has converged, instead of running out the cap.
- **Rolls back** to the best-so-far iteration before a loop degrades a good result.
- **Reports** the iterations it saved versus a fixed cap, exposed as `result.savings_vs_fixed_cap`.

Under the hood it's a **Barkhausen-criterion (`Aβ`) stability classifier** — the same loop-gain test that decides whether any feedback system converges or oscillates, applied to an LLM agent loop instead of an amplifier. That's the *how*; the outcome is less spend and faster loops.

```bash
pip install loopgain
```

```python
from loopgain import LoopGain

lg = LoopGain(target_error=0.1)          # wrap your existing loop — framework-agnostic

while lg.should_continue():              # stops on convergence, not a fixed cap
    errors = verifier.verify(output)
    lg.observe(errors, output=output)
    output = reviser.revise(output, errors)

result = lg.result
print(result.outcome)        # "converged" | "stalled" | "oscillating" | "diverged" | "max_iterations"
print(result.best_output)    # best-so-far iteration, automatically recovered
```

## The numbers

Measured across a public benchmark of **2,000 paired real-API trials** (8,000 loop runs) against a fixed `max_iterations=20` baseline, on `loopgain` v0.4.0:

| Metric | Result |
|---|---|
| Cost | **92.8% reduction** in total API spend ($27.05 → $1.94 across the run) |
| Latency | **~15× faster** median wall-clock (the ratio is the stable claim; absolute latency is environment-dependent) |
| Quality | preserved on natural-distribution workloads (W1–W4); *improved* on engineered-failure workloads (W5) |

Weighted judge preference 0.678 across 1,800 pairwise comparisons, and **zero of six pre-registered kill criteria fired.** Full protocol, raw data, and the cases where it *doesn't* help are public: **[loopgain-bench](https://github.com/loopgain-ai/loopgain-bench)**.

> **Scope, honestly:** LoopGain proves a loop *stopped moving* and recovers the best iteration it saw — it does not by itself prove the loop stopped at the *correct* answer. It's a cost-and-stability control on the loop, not a correctness oracle.

## Works with your stack

Six first-class adapters plus the raw API — framework- and model-agnostic, never tied to one provider:

**LangGraph** · **CrewAI** · **AutoGen** · **LangChain** · **OpenAI Agents SDK** · **Claude Agent SDK**

## Open-core

- **`loopgain`** — the library. **Apache-2.0**, free, self-hostable. [PyPI](https://pypi.org/project/loopgain/) · [source](https://github.com/loopgain-ai/loopgain)
- **Hosted dashboard + telemetry** — paid SaaS for fleet-wide loop observability and alerting. [dashboard.loopgain.ai](https://dashboard.loopgain.ai)

Source is open; hosting and ops are paid. Run it entirely yourself, or let us run the dashboard.

---

<div align="center">

**[loopgain.ai](https://loopgain.ai)** · [@LoopGainAI](https://x.com/LoopGainAI) · [LinkedIn](https://www.linkedin.com/company/loopgain-ai) · [hello@loopgain.ai](mailto:hello@loopgain.ai)

</div>
