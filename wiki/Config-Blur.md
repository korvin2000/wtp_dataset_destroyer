# Blur configuration

```py
from pepedd.pipeline.schema import Degradation
from pepedd_nodes.blur import BlurOptions, TargetKernels

blur = Degradation(
    type="blur",
    options=BlurOptions(
        filters=[
            "box",
            "gauss",
            "median",
            "lens",
            "motion",
            "random",
            "airy",
            "ring",
            "triangle",
        ],
        kernel=[0.1, 2.0],
        target_kernels=TargetKernels(
            box=[0.5, 1.5],
            gauss=None,  # defaults to kernel
            median=None,
            lens=None,
            random=None,
            airy=None,
            triangle=None,
            ring=None,
        ),
        motion_size=[0, 10],
        motion_angle=[-30.0, 30.0],
        ring_thickness=[1, 5],
        probability=1.0,
        seed=None,
    ),
)
```

## Parameters

* **filters** — List of blur kernels to sample from per image.
* **kernel** — Base kernel range for all filters (used when `target_kernels` is not set for a specific filter).
* **target_kernels** — Per-filter kernel ranges. Any `None` entry is filled with `kernel`.
* **motion_size** — Motion blur kernel size range (integers).
* **motion_angle** — Motion blur angle range in degrees.
* **ring_thickness** — Ring blur thickness range (integers).
* **probability** — Per-node activation probability.
* **seed** — Optional per-node RNG seed to override the global seed.

## Behavior notes

* Each image chooses **one** blur filter from `filters` at random.
* All blur filters operate on the LQ image only (HQ is untouched).
* Motion blur uses both `motion_size` and `motion_angle` ranges.

## HCL example

```hcl
degradation {
  type = "blur"
  options = {
    filters = ["box", "gauss", "ring"]
    kernel = [0.1, 1.5]
    ring_thickness = [1, 3]
    probability = 0.4
  }
}
```
