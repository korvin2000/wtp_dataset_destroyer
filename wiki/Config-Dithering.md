# Dithering configuration

```py
from pepedd.pipeline.schema import Degradation
from pepedd_nodes.dithering import DitheringOptions, TargetColorCh

options = DitheringOptions(
    types=["floydsteinberg", "order", "quantize"],
    palette=["oc_tree", "median_cut", "l_popular"],
    color_in_img=[32, 512],
    target_color=TargetColorCh(
        floydsteinberg=[32, 128],
        order=[64, 256],
        quantize=[64, 256],
    ),
    map_size=[2, 4, 8, 16],
    history=[10, 15],
    ratio=[0.1, 0.9],
    probability=0.5,
    seed=None,
)

DitheringNode = Degradation(type="dithering", options=options)
```

## Parameters

* **types** — Dithering algorithms to sample from:
  `floydsteinberg`, `jarvisjudiceninke`, `stucki`, `atkinson`, `burkes`, `sierra`,
  `tworowsierra`, `sierraLite`, `order`, `riemersma`, `quantize`.
* **palette** — Palette builders used for quantization (`oc_tree`, `median_cut`, `wu`, `l_*`, `uniform`, etc.).
* **color_in_img** — Default palette size range used by all algorithms.
* **target_color** — Per-algorithm palette size overrides (falls back to `color_in_img`).
* **map_size** — Ordered dither matrix sizes (`order` mode only).
* **history / ratio** — Riemersma parameters (memory length and feedback ratio).
* **probability** — Activation probability.
* **seed** — Optional per-node seed.

## Behavior notes

* One algorithm from `types` is selected per image.
* Grayscale inputs are temporarily converted to RGB, dithered, then converted back.

## HCL example

```hcl
degradation {
  type = "dithering"
  options = {
    types = ["floydsteinberg", "order", "quantize"]
    palette = ["oc_tree", "median_cut", "l_popular"]
    color_in_img = [32, 512]
    map_size = [2, 4, 8, 16]
    probability = 0.4
  }
}
```
