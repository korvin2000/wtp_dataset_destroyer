# **What is PepeDD (WTP Dataset Destroyer)**
PepeDD is a Python library for building paired datasets by applying configurable degradations to high-quality (HQ) images to create low-quality (LQ) targets. The output is suitable for supervised restoration, enhancement, and super-resolution pipelines.

## **Project model (current implementation)**
The core pipeline is implemented in `pepedd.pipeline.PipeLine`. It:

* Reads images as `float32` (`ImgFormat.F32`) via `pepeline.read`/`read_tiler`.
* Builds a sequential list of *nodes* (degradations), each of which is a `Node` that can be probabilistically applied.
* Seeds per-image randomness using a stable mix of `seed` + `image path` (+ `tile index` for tiling).
* Writes LQ (and optionally HQ) PNGs into `output/lq` and `output/hq`.

Conceptual flow:

```
Input Image
   │
   ▼
[Degradation Pipeline]
   ├─ blur
   ├─ resize
   ├─ compress
   └─ ...
   │
   ▼
Output LQ (and HQ)
```

## **Key invariants**
* **Determinism:** global `seed` plus per-image mixing yields stable outputs given the same inputs and configuration.
* **Sequential semantics:** degradations are executed in order, left-to-right, without reordering.
* **Node-level probability:** each degradation node can be conditionally applied (`probability`).
* **Data model:** both HQ and LQ live in `LQHQState`; nodes can modify either or both.

## **Configuration philosophy (program of thought)**
Use the pipeline as a probabilistic program. A practical design loop:

1. **Define invariants** — target output size, HQ/LQ pairing rules, reproducibility requirements.
2. **Choose a minimal chain** — start with the smallest set of degradations that reproduce your target artifacts.
3. **Bound randomness** — for each node, set safe ranges and a probability that reflects how often you see the artifact.
4. **Validate edges** — ensure no node produces invalid shapes, invalid ranges, or type mismatches.
5. **Iterate** — add one node at a time; measure the delta.

This pipeline-first, invariant-driven approach avoids overly chaotic degradations that harm training stability.

## **Add-ons and modularity**
Nodes (degradations) are registered dynamically through decorators. Importing `pepedd_nodes` triggers registration of built-in nodes. Custom add-ons can register new node types without modifying the core pipeline.

See:
* **Quick Start** for getting running fast.
* **Config-Base** for pipeline options.
* **Config-* docs** for each degradation’s parameters.
