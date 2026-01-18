# Quick Start

## Installation

### Install the core

```sh
uv pip install pepedd
```

### Install basic nodes (degradations)

```sh
uv pip install "pepedd[nodes]"
# or
uv pip install pepedd-nodes
```

### Install add-ons
Only after installing the core:

```sh
uv pip install your_addon
```

## Example Usage (Python)

```py
from pepedd.pipeline.pipeline import PipeLine
from pepedd.pipeline.schema import PipelineOptions, Degradation
import pepedd_nodes  # registers built-in nodes via side effects
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

pipeline = PipeLine(options)
pipeline()
```

### Dictionary configuration

```py
import pepedd_nodes
from pepedd.pipeline.pipeline import PipeLine

options = {
    "input": "input_dir",
    "output": "output_dir",
    "degradation": [
        {
            "type": "blur",
            "options": {
                "filters": ["box", "gauss"],
                "kernel": [0.2, 3.0],
                "probability": 0.2,
            },
        }
    ],
}

PipeLine(options)()
```

## Uninstallation

```sh
uv pip uninstall pepedd
uv pip uninstall pepedd-nodes
```
