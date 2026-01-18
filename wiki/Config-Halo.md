# Halo configuration

```py
from pepedd.pipeline.schema import Degradation
from pepedd_nodes.halo import HaloOptions
from pepedd_nodes.blur import BlurOptions

halo = Degradation(
    type="halo",
    options=HaloOptions(
        type_halo=["rgb", "y"],
        blur=BlurOptions(
            filters=["gauss", "lens"],
            kernel=[0.1, 2.0],
        ),
        threshold=[0.0, 0.2],
        amount=[1.0, 3.0],
        probability=0.4,
        seed=None,
    ),
)
```

## Parameters

* **type_halo** — Channels for halo generation:
  * `"rgb"` — operate directly in RGB.
  * `"y"` — operate on luminance (Y channel) and recompose into RGB.
* **blur** — Nested `BlurOptions` controlling the blur used to compute halo edges.
* **threshold** — Per-pixel threshold for edge strength (values below threshold are ignored).
* **amount** — Scale factor for halo intensity.
* **probability** — Activation probability.
* **seed** — Optional per-node seed.

## Behavior notes

* Halo is computed as `diff = original - blurred`, followed by thresholding and scaling.
* Both LQ and HQ are preserved, but only LQ is modified.

## HCL example

```hcl
degradation {
  type = "halo"
  options = {
    type_halo = ["rgb"]
    blur = {
      filters = ["gauss", "lens"]
      kernel = [0.1, 2.0]
    }
    threshold = [0.0, 0.2]
    amount = [1.0, 3.0]
    probability = 0.4
  }
}
```
