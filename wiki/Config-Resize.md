# Resize configuration

```py
from pepedd.pipeline.schema import Degradation
from pepedd_nodes.resize import ResizeOptions, DownUpOptions, UpDownOptions, DownDownOptions

resize = Degradation(
    type="resize",
    options=ResizeOptions(
        alg_lq=["down_up", "down_down", "s_catmullrom", "nearest"],
        alg_hq=["s_catmullrom", "dpid_0.5"],
        down_up=DownUpOptions(
            down=[2, 4],
            alg_up=["s_catmullrom", "nearest"],
            alg_down=["s_catmullrom"],
        ),
        up_down=UpDownOptions(
            up=[2, 4],
            alg_up=["s_catmullrom"],
            alg_down=["s_catmullrom", "nearest"],
        ),
        down_down=DownDownOptions(
            step=[2, 6],
            alg_down=["s_catmullrom"],
        ),
        spread=[0.5, 1.0, 0.05],
        scale=2,
        s_samplings=[2, 8],
        divider=2,
        olq=False,
        probability=1.0,
        seed=None,
    ),
)
```

## Parameters

* **alg_lq / alg_hq** — Lists of resize algorithms for LQ and HQ, respectively. Each image samples one from the list.
  * Supported values include any algorithm name from `pepedd_nodes.utils.constants.RESIZE_LIST`,
    plus composite methods: `down_up`, `up_down`, `down_down`, and any `dpid_<lambda>` value.
* **down_up / up_down / down_down** — Composite resize method configs:
  * **down_up** — downscale then upscale by `down` factor(s).
  * **up_down** — upscale then downscale by `up` factor(s).
  * **down_down** — repeated downscales in `step` stages.
* **spread** — Scale factor applied before resizing (range or range+step). This controls additional variability in target sizes.
* **scale** — Base downscale ratio between HQ and LQ sizes.
* **s_samplings** — Supersampling factors for `s_*` algorithms.
* **divider** — Ensures dimensions are divisible by this value.
* **olq** — If `True`, only LQ is resized; HQ stays at original resolution.
* **probability** — Activation probability.
* **seed** — Optional per-node seed.

## Algorithm naming (practical guide)

Resize algorithms are dynamically validated. Common patterns:

* `nearest` — nearest-neighbor resize.
* `c_*`, `i_*`, `s_*` — algorithms backed by `pepeline.ResizesAlg` with a filter suffix (`box`, `gaussian`, `catmullrom`, `mitchell`, `lanczos3`, etc.).
* `dpid_<lambda>` — deep priors (e.g., `dpid_0.5`).
* `down_up` / `up_down` / `down_down` — composite resizers (see above).

## Behavior notes

* HQ and LQ sizes are adjusted to stay divisible by `divider` after scaling.
* LQ is always resized; HQ is skipped when `olq=True`.
* `spread` is sampled per image, then HQ/LQ sizes are computed using `scale` and `divider`.

## HCL example

```hcl
degradation {
  type = "resize"
  options = {
    alg_lq = ["down_up", "s_catmullrom", "nearest"]
    alg_hq = ["dpid_0.5"]
    spread = [0.5, 1.0, 0.05]
    scale = 2
    down_up = {
      down = [2, 4]
      alg_up = ["s_catmullrom", "nearest"]
      alg_down = ["s_catmullrom"]
    }
  }
}
```
