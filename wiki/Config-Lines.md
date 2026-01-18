# Lines configuration

```py
from pepedd.pipeline.schema import Degradation
from pepedd_nodes.lines import LinesOptions

lines = Degradation(
    type="lines",
    options=LinesOptions(
        mode=["lines_random", "beziers_random", "rays_uniform"],
        n_lines=[1, 30],
        size0=[1, 3],
        size1=[1, 5],
        line_range0=[0.0, 0.2],
        line_range1=[0.8, 1.0],
        alpha=[0.2, 0.7],
        h=[0.0, 1.0],
        s=[0.2, 1.0],
        v=[0.2, 1.0],
        radius0=[0, 128],
        radius1=[256, 512],
        angle0=[-180, -1],
        angle1=[1, 180],
        probability=0.2,
        seed=None,
    ),
)
```

## Parameters

* **mode** — Line generators to sample from:
  `lines_random`, `beziers_random`, `circle_random`, `circle_uniform`, `rays_uniform`, `rays_random`.
* **n_lines** — Number of line primitives.
* **size0 / size1** — Line thickness range.
* **line_range0 / line_range1** — Normalized line length range (for ray generators).
* **alpha** — Opacity range for line overlay (0–1).
* **h / s / v** — HSV color ranges for line color.
* **radius0 / radius1** — Circle/ray radius ranges.
* **angle0 / angle1** — Angular range for rays/circles.
* **probability** — Activation probability.
* **seed** — Optional per-node seed.

## Behavior notes

* Lines are applied to **both** HQ and LQ so the pair remains aligned.
* If LQ size differs from HQ (e.g., after resize), the line mask is resized before application.

## HCL example

```hcl
degradation {
  type = "lines"
  options = {
    probability = 0.2
    mode = ["lines_random", "rays_uniform"]
    n_lines = [1, 25]
    size0 = [1, 3]
    size1 = [1, 5]
    alpha = [0.2, 0.7]
  }
}
```
