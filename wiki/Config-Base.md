# Pipeline configuration (base)

## Python usage

```py
from pepedd.pipeline.pipeline import PipeLine
from pepedd.pipeline.schema import PipelineOptions, Degradation, TileOptions
import pepedd_nodes
from pepedd_nodes.blur import BlurOptions

options = PipelineOptions(
    input="input_dir",  # str
    output="output_dir",  # str
    map_type="thread",  # "simple" | "thread" | "process"
    degradation=[
        Degradation(
            type="blur",  # registered node name
            options=BlurOptions(),  # Pydantic model | dict
        )
    ],
    num_workers=16,
    tile=TileOptions(size=512, no_wb=True),  # Optional[TileOptions]
    dataset_size=1000,  # Optional[int]
    shuffle_dataset=True,
    gray=False,
    debug=False,
    only_lq=False,
    real_name=False,
    output_clear=True,
    seed=None,  # Optional[int]
)

PipeLine(options)()
```

## Parameter descriptions

* **input** — Directory of HQ images to process.
* **output** — Output directory. LQ images are stored in `output/lq`, HQ images in `output/hq` (unless `only_lq`).
* **map_type** — Execution mode:
  * `simple` — single-threaded loop.
  * `thread` — thread pool (good for I/O, limited by GIL for CPU-heavy nodes).
  * `process` — multi-process execution for CPU-bound workloads.
* **degradation** — Required list of `Degradation` blocks. Each block has a `type` and `options`.
* **num_workers** — Worker count for `thread`/`process` modes.
* **tile** — Optional tiling. When set, images are split into tiles of size `tile.size` and processed independently.
  * `no_wb` skips tiles that are pure white or black.
* **dataset_size** — Cap the number of processed input files.
* **shuffle_dataset** — Shuffle input files before processing.
* **gray** — Read images in grayscale (LQ/HQ are 2D arrays).
* **debug** — Forces deterministic `map_type="simple"` and records per-image parameters in `debug/*.log`.
* **only_lq** — Save only LQ outputs.
* **real_name** — Use original filenames instead of index-based names.
* **output_clear** — Delete `output` before creating new outputs.
* **seed** — Global seed. If unset, a random 64-bit seed is generated.

## Concurrency and entrypoint
`map_type="process"` uses `ProcessPoolExecutor`. On Windows (and some Linux configurations), wrap the entrypoint:

```py
if __name__ == "__main__":
    PipeLine(options)()
```

## HCL-style configuration (external parsing)

The repository contains example HCL config (e.g., `digital_art.hcl`). The core library does **not** ship with an HCL parser, so you must parse it externally and convert it into a `PipelineOptions`-compatible dict.

A minimal mental model:

```hcl
input = "input_dir"
output = "output_dir"

# Each degradation is a node with a type and options.
degradation {
  type = "blur"
  options = {
    filters = ["gauss"]
    kernel = [0.2, 1.5]
    probability = 0.3
  }
}
```

Equivalent Python dict:

```py
{
  "input": "input_dir",
  "output": "output_dir",
  "degradation": [
    {
      "type": "blur",
      "options": {
        "filters": ["gauss"],
        "kernel": [0.2, 1.5],
        "probability": 0.3,
      },
    },
  ],
}
```

## Defaults (implementation snapshot)

```py
class PipelineOptions(BaseModel):
    input: str = "input"
    output: str = "output"
    map_type: Literal["simple", "thread", "process"] = "thread"
    degradation: List[Degradation]

    num_workers: Optional[int] = None
    tile: Optional[TileOptions] = None

    dataset_size: Optional[int] = None
    shuffle_dataset: bool = True
    gray: bool = False

    debug: bool = False

    only_lq: bool = False
    real_name: bool = False

    output_clear: bool = True
    seed: Optional[int] = None

    @model_validator(mode="after")
    def apply_debug_and_seed(self):
        if self.debug:
            self.map_type = "simple"
            if self.seed is None:
                self.seed = 1234
        if self.seed is None:
            import secrets

            self.seed = secrets.randbits(64)
        return self
```
