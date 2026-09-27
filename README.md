# EndoClock

**Did the Grid Erase the Event? EndoClock for Auditing Medical World-Model Pipelines**

Yarin Udi · Tom Sharon-Shahak · Roee Masad · Dan Pri-Tal — Diastole Medical R&D

[![arXiv](https://img.shields.io/badge/arXiv-2608.09266-b31b1b.svg)](https://arxiv.org/abs/2608.09266)
[![Workshop](https://img.shields.io/badge/MWM4MICCAI-2026-1f4e79.svg)](https://mwm2026.github.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

- 📄 **Paper:** <https://arxiv.org/abs/2608.09266> · [PDF](https://arxiv.org/pdf/2608.09266)
- 🏆 **Accepted at** the 1st MICCAI Workshop on Medical World Models
  ([MWM4MICCAI 2026](https://mwm2026.github.io/)), co-located with MICCAI 2026,
  Strasbourg, France.

---

## The problem in one paragraph

Medical world models commonly learn from multimodal recordings synchronized onto a
fixed-rate grid, which resamples each native stream onto a shared time axis. Each stream
has an **observation clock** governing when observations are emitted or updated. When that
clock depends on the latent or acquisition state, it is **endogenous**, and synchronization
may not be a neutral representation change: it can erase task-relevant evidence before the
model sees the data.

The evidence needed to distinguish a target event or state is what the paper calls a
**witness**. In the limiting case, every modeled stream is censored or invariant while the
distinguishing event is retained only outside the modeled record — a **no-witness
interval**, in which the modeled record has the same in-interval law under both latent
histories, *including both event times and recorded values*. Because every fixed-grid
representation is a deterministic function of the modeled record, a finer grid gives more
identical tiles, not more information. Raising the grid rate, or appending update masks,
validity flags, Δt features or timestamp channels built from that record, can **preserve**
a witness that is already present, but cannot **recreate** one that is absent.

## What EndoClock does

EndoClock is a conservative **pretraining audit**. Given a target distinction, a candidate
interval, the modeled modalities, the resampling operator and any available acquisition
metadata, it reports the **lowest witness-bearing representation** supported by the
available evidence. Each regime requires every coarser quantity to agree and the next one
to differ:

| Regime | Witness-bearing level | Recommended policy |
|---|---|---|
| **(1) Value-witnessed** (clock-safe) | fixed-grid values | standard grid resampling is safe for this distinction |
| **(2) Cell-witnessed** | values + update or validity summary | retain the update mask or validity flags alongside values |
| **(3) Sub-cell-witnessed** | values + native timestamps or event ordering | retain native timestamps, or model the stream in continuous time |
| **(4) External-witness-only** | external acquisition channel | ingest the external channel, or redefine the target distinction |

Regime 1 covers every case in which the clocks are exogenous, or their state dependence is
irrelevant to the distinction. The four regimes are mutually exclusive but **not
exhaustive**: when the evidence establishes none of them the audit returns `unresolved`,
whose policy is to preserve every native channel until the regime is resolved.

**Decision rules.** `external_witness_only` is returned only when regime 4 is certified
*structurally* — zero write-outs on the interval together with a device-level rule that
accounts for them, an invariant remaining modality, and a verified external-only marker.
Two symmetric guards keep the audit from reading an absence of evidence as a certificate:

- A merely statistical failure to reject equality of the competing observation kernels is
  not a proof of equality, and returns `unresolved`. The same holds for a non-detection of
  clock-state dependence, which never certifies exogeneity.
- A modality that merely continues to update on the interval is not a witness unless a
  cell-level difference is actually exhibited, so it does not license a `cell_witnessed`
  verdict.

Being conservative, the audit can flag an interval whose censoring is state-correlated even
though the target distinction does not depend on its in-interval content. Such a false
positive costs retained metadata, not lost evidence. `unresolved` is the default whenever
certification fails, directing the practitioner to preserve the native channels rather than
to conclude that resampling was safe.

## What EndoClock is *not*

- It **does not infer the target distinction** — you must specify what is being audited.
- It **does not prove exogeneity** from a failure to reject dependence.
- It **may require device and acquisition knowledge** to certify structural conditions.
- It **does not establish prevalence** of this failure mode, and does not validate the
  audit across devices, institutions, or acquisition workflows.
- It **does not guarantee finite-sample learnability** even when a witness is retained.
- It is a **preliminary failure alert and executable audit**, not a production tool.

One further caution from the paper: an external witness should **not** automatically be
added as a model input, because it may be unavailable at inference, non-causal, or a source
of target leakage.

## Evidence and its limits

- **Worked echocardiography example.** When an echocardiography console enters pulsed-wave
  Doppler (PWD) mode, B-mode write-outs may cease while the operator holds the Doppler
  gate. In the examined pipeline the modeled image stream is fully censored across the
  candidate interval, leaving no cell-level or sub-cell image witness, while five PWD
  measurement events remain recorded in an external acquisition log. EndoClock returns
  `external_witness_only` for this single de-identified acquisition. The example
  illustrates how a task-relevant witness can be retained outside the modeled record; it is
  not a prevalence estimate and not a corpus-level result.
- **Controlled synthetic check.** All four regimes are exercised on a parameterized
  generator with known ground truth. Within a constructed no-witness interval the modeled
  values are label-independent by construction, so the evidential result is not that a
  stream-only learner fails, but that **the failure is selective**: from the same flattened
  stream, values-only recovers a coarse target at 0.915 AUC while the fine in-interval
  target stays at chance (0.524, 95% CI [0.482, 0.564]), and only the external marker fully
  recovers it. The collapse persists across a 48× grid-rate range (5–240 Hz) and for every
  probe class tried, from logistic regression to a recurrent network.

The underlying cardiac-ultrasound corpus is private and is not released. The measured trace
is a de-identified, derived timing summary containing no DICOM, no pixel data, and no
identifiers.

## Citation

Please cite the peer-reviewed workshop version. The paper is on arXiv at
**<https://arxiv.org/abs/2608.09266>** ([PDF](https://arxiv.org/pdf/2608.09266)).

```bibtex
@inproceedings{udi2026endoclock,
  title         = {Did the Grid Erase the Event? {EndoClock} for Auditing Medical
                   World-Model Pipelines},
  author        = {Udi, Yarin and Sharon-Shahak, Tom and Masad, Roee and Pri-Tal, Dan},
  booktitle     = {Proceedings of the 1st MICCAI Workshop on Medical World Models
                   (MWM4MICCAI)},
  year          = {2026},
  eprint        = {2608.09266},
  archivePrefix = {arXiv},
  primaryClass  = {cs.CV},
  url           = {https://arxiv.org/abs/2608.09266},
  note          = {To appear}
}
```

## License

Released under the [MIT License](LICENSE).

## Contact

Yarin Udi — <yarin@diastole.io> · Diastole Medical R&D
