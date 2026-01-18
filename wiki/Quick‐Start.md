# Quick Start

## Installation

To install PepeDD in your project, run the following commands.

### Install the core:

```sh
uv pip install pepedd
```

### Install basic nodes (degradations):

```sh
uv pip install "pepedd[nodes]"
# or
uv pip install pepedd_nodes
```

### Install add-ons
Only after installing the `pepedd` core:

```sh
uv pip install your_addon
```

## Uninstallation

### Uninstall the core:

```sh
uv pip uninstall pepedd
```
### Uninstall basic nodes:

```sh
uv pip uninstall pepedd_nodes
```

## Example Usage

```py
from pepedd.core import PipeLine, PipelineOptions, Degradation
from pepedd.nodes.blur import BlurOptions

options = PipelineOptions(
    input="input_dir",
    output="output_dir",
    degradation=[
        Degradation(
            type="blur", options=BlurOptions(kernel=[0.2, 3.0], probability=0.2)
        )
    ],
)

# Example in dictionary format:
# dict_options = {
#     "input": "input_dir",
#     "output": "output_dir",
#     "degradation": {
#         "type": "blur",
#         "options": {
#             "kernel": [0.2, 3.0],
#         }
#     },
# }

pipeline = PipeLine(options)
pipeline()
```