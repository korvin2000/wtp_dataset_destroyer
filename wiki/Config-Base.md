```py
from pepedd.core import PipeLine, PipelineOptions, Degradation
from pepedd.core.pipeline.schema import TileOptions
from pepedd.nodes.blur import BlurOptions
options = PipelineOptions(
    input="input_dir",  # str
    output="output_dir",  # str
    map_type="simple",  # Literal["simple", "thread", "process"]
    degradation=[
        Degradation(
            type="blur", #str
            options=BlurOptions() #BaseModel | Dict
        )
    ],  # List[Degradation]
    num_workers=16,
    tile=TileOptions(size=512, no_wb=True), # Optional[TileOptions]
    dataset_size=1000, # Optional[int]
    shuffle_dataset=True, # bool
    gray=False, # bool
    debug=True, # bool
    only_lq=False, # bool
    real_name=False, # bool
    output_clear=True, # bool
    seed=None # Optional[int]
)

```

## Parameter Descriptions:

* **input** — Directory containing the source images (HQ).
* **output** — Directory where the generated dataset will be saved.
* **map_type** — Execution mode for tasks:
  * `simple` — Sequential processing in a single thread.
  * `thread` — Multi-threaded mode. Suitable for I/O-bound tasks, but computational operations may block each other due to the GIL (Global Interpreter Lock).
  * `process` — Multi-processing mode. Each process runs in its own environment with its own GIL, providing true parallelism. **Note:** Requires the script to be executed strictly within an `if __name__ == "__main__":` block.


* **degradation** — A list of applied degradations. The `type` field specifies the method name, and `options` contains its configuration.
* **num_workers** — The number of worker threads or processes. This is ignored if `map_type="simple"`.
* **tile** — Tiling settings (`TileOptions`). If this parameter is not set, images are processed in their entirety.
* **dataset_size** — The maximum number of images in the final dataset. If not specified, all available files are processed.
* **shuffle_dataset** — If `True`, the file list is shuffled before processing begins.
* **gray** — If `True`, images will be forcibly read in grayscale.
* **debug** — Debug mode. It forces `map_type="simple"` and creates a log file containing pipeline and node parameters.
* **only_lq** — If `True`, only the augmented (LQ) images are saved, without their corresponding HQ originals.
* **real_name** — If `True`, saves files using their original names instead of sequential indices. The format is forcibly changed to `.png`.
* **output_clear** — If `True`, the `output` directory will be completely cleared before the pipeline starts.
* **seed** — Global value for the random number generator. If not specified, it is generated randomly (in `debug` mode, the value is printed to the log).

---

> **Note:** All parameters are optional and have default values, with the exception of **degradation**, which is the only required field.

### Default Values (Implementation):

```py
class Degradation(BaseModel):
    type: str
    options: Any


class TileOptions(BaseModel):
    size: int = 512
    no_wb: bool = True


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
