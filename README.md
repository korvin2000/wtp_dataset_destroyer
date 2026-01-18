# WTP Dataset Destroyer (PepeDD)

PepeDD is a Python library for generating paired HQ/LQ image datasets by applying a configurable, probabilistic degradation pipeline. It is designed for training image restoration, enhancement, and super‑resolution models while keeping data generation deterministic and reproducible when seeded.

## At a glance
- **Paired outputs:** writes LQ (and optional HQ) PNGs to `output/lq` and `output/hq`.
- **Deterministic pipelines:** global seed + per‑image mixing for stable outputs.
- **Composable nodes:** sequential degradation nodes with per‑node probability controls.
- **Scalable execution:** simple, threaded, or process‑based mapping; optional tiling for large images.
- **Extensible:** register custom degradations as add‑ons without forking the core.

## Quick start

### Install
```bash
uv pip install pepedd
uv pip install "pepedd[nodes]"
# or
uv pip install pepedd-nodes
```

### Minimal pipeline (Python)
```py
from pepedd.pipeline.pipeline import PipeLine
from pepedd.pipeline.schema import PipelineOptions, Degradation
import pepedd_nodes  # registers built-in nodes
from pepedd_nodes.blur import BlurOptions

options = PipelineOptions(
    input="input_dir",
    output="output_dir",
    degradation=[
        Degradation(
            type="blur",
            options=BlurOptions(kernel=[0.2, 3.0], probability=0.2),
        )
    ],
)

PipeLine(options)()
```

More examples: [Quick Start](wiki/Quick‐Start.md).

## Core concepts (brief)
- **Pipeline model:** `PipeLine` reads HQ images as `float32`, applies nodes in sequence, and writes LQ (and optional HQ) outputs.
- **Data model:** each node receives an `LQHQState` containing both LQ/HQ images and a seeded RNG.
- **Probabilistic program:** nodes can be conditionally applied via `probability` with per‑node seeds if needed.

See [Home](wiki/Home.md) for the full model and invariants.

## Configuration overview

### Base options (common parameters)
Pipeline settings include `input`, `output`, `map_type`, `degradation`, `num_workers`, `tile`, `dataset_size`, `shuffle_dataset`, `gray`, `debug`, `only_lq`, `real_name`, `output_clear`, and `seed`. These control I/O, concurrency, tiling, determinism, and output layout.

Details and defaults: [Config‑Base](wiki/Config-Base.md).

### HCL configuration
The repository ships with example HCL configs (e.g., `digital_art.hcl`). Parse them externally into a `PipelineOptions`‑compatible dict. Example mapping is documented in [Config‑Base](wiki/Config-Base.md).

## Built‑in degradations (nodes)
Import `pepedd_nodes` to register these node types. Each section below links to parameter docs and HCL examples.

| Node | Purpose | Docs |
| --- | --- | --- |
| Blur | Kernel‑based blur variants | [Config‑Blur](wiki/Config-Blur.md) |
| Resize | Down/Up composite resizing | [Config‑Resize](wiki/Config-Resize.md) |
| Compress | JPEG/WebP/AVIF + video codecs | [Config‑Compress](wiki/Config-Compress.md) |
| Dithering | Error diffusion & ordered dithering | [Config‑Dithering](wiki/Config-Dithering.md) |
| Halo | Glow/edge halo effects | [Config‑Halo](wiki/Config-Halo.md) |
| Lines | Random lines, rays, circles | [Config‑Lines](wiki/Config-Lines.md) |

## Add‑ons (custom nodes)
Create and ship your own degradations with a small Pydantic options model + a `Node` implementation registered via `@register_class`. See [Creating add‑ons](wiki/Create-add‐ons.md).

## Project structure
- `pkgs/pepedd`: core pipeline, schemas, registration, and utilities.
- `pkgs/pepedd-nodes`: built‑in degradations (blur, resize, compress, dithering, halo, lines).
- `wiki/`: full documentation and configuration references.

## Credits
- Resize is implemented through [chaiNNer‑rs](https://github.com/chaiNNer-org/chaiNNer-rs).
- Compression is based on [Kim2091's implementation](https://github.com/Kim2091/helpful-scripts/blob/d413054eda3764fd04ec2c22fb3c3b6a5e61e31a/Dataset%20Destroyer/datasetDestroyer.py#L279).
