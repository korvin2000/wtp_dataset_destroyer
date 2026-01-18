```py
from pepedd.core import Degradation
from pepedd.nodes.blur import BlurOptions,TargetKernels

degratation = Degradation(
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
        ], #LiteralBlur = Literal[ "box", "gauss", "median", "lens", "motion", "random", "airy", "ring", "triangle"]
        kernel=[1.0, 2.0],  # List max_min_length=2 min_value 0.0 float
        target_kernels = TargetKernels(
            box = [2.0,10.0], # Optional List max_min_length=2 min_value 0.0 float
            gauss = None, # Optional List max_min_length=2 min_value 0.0 float
            median = None, # Optional List max_min_length=2 min_value 0.0 float
            lens = None, # Optional List max_min_length=2 min_value 0.0 float
            random = None, # Optional List max_min_length=2 min_value 0.0 float
            airy = None, # Optional List max_min_length=2 min_value 0.0 float
            triangle = None, # Optional List max_min_length=2 min_value 0.0 float
            ring = None, # Optional List max_min_length=2 min_value 0.0 float
        ),
        motion_size = [0, 10], # List max_min_length=2 int
        motion_angle = [-30.0, 30.0], # List max_min_length=2 float
        ring_thickness =  [1, 5], # List max_min_length=2 int
        probability = 1.0, # Float 0-1
        seed = None # Optional[int]
    ),
)
```
## Parameter Descriptions:
* **filters** — a list of blur filters to be applied.
* **kernel** — the base kernel size. For most filters, the final kernel size is calculated as `ceil(kernel) * 2 + 1`. For example, if `1.1` is provided for the `box` filter, the resulting kernel size will be `5`, where the three central values are equal to `1` and the remaining values are `0.1`.
* **target_kernels** — a list of kernel sizes for each filter. If not specified, the value from `kernel` is used. In practice, this parameter defines the actual kernel sizes, while `kernel` serves as the base value. The kernel size for `motion` cannot be set here and is configured via a separate parameter.
* **motion_size** — a dedicated parameter for `motion` blur, as it operates differently from other filters. It defines the pixel displacement along the specified rotation angle.
* **motion_angle** — the rotation angle for `motion` blur.
* **ring_thickness** — the ring thickness for the `ring` filter.
* **probability** — the probability of applying the degradation. For example, `0.5` corresponds to 50% and `0.75` to 75%.
* **seed** — the degradation seed. When set, it overrides the global seed, freezing the degradation output and ensuring reproducibility.

## Default Values (Implementation):
https://github.com/umzi2/wtp_dataset_destroyer/blob/rework/pepedd/nodes/blur/schemas.py