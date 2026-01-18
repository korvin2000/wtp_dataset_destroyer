# Creating add-ons (custom nodes)

This guide shows how to create a new degradation node and ship it as a reusable add-on package.

## 1) Define a Pydantic options model

```py
from typing import Optional
from pydantic import BaseModel, Field

class MyNoiseOptions(BaseModel):
    strength: float = Field(0.2, ge=0.0, le=1.0)
    probability: float = 1.0
    seed: Optional[int] = None
```

## 2) Implement a Node

```py
import numpy as np
from pepedd.node_register import register_class
from pepedd.objects.node_base import Node
from pepedd.objects.lq_hq_state import LQHQState

@register_class("my_noise")
class MyNoise(Node):
    def __init__(self, options: MyNoiseOptions | dict):
        opts = MyNoiseOptions(**options) if isinstance(options, dict) else options
        super().__init__(opts.probability, opts.seed)
        self.strength = opts.strength

    def forward(self, data: LQHQState) -> LQHQState:
        noise = data.rng.normal(0.0, self.strength, size=data.lq.shape)
        data.lq = (data.lq + noise).clip(0, 1)
        return data
```

## 3) Register in your package

Your package must import the module containing the node so the decorator runs.
A simple `__init__.py` approach:

```py
# your_addon/__init__.py
from .my_noise import MyNoise, MyNoiseOptions

__all__ = ["MyNoise", "MyNoiseOptions"]
```

## 4) Use the add-on

```py
import pepedd_nodes  # optional: built-in nodes
import your_addon    # registers custom nodes
from pepedd.pipeline.pipeline import PipeLine

options = {
    "input": "input_dir",
    "output": "output_dir",
    "degradation": [
        {"type": "my_noise", "options": {"strength": 0.3}},
    ],
}

PipeLine(options)()
```

## Design notes (program of thought)

1. **Invariant:** Nodes must accept/return `LQHQState` with `float32` images.
2. **Safety:** Clip values to `[0, 1]` after adding noise or artifacts.
3. **Reproducibility:** Respect `probability` and `seed` from `Node` base.
4. **Isolation:** Do not modify global RNG; use `data.rng` only.
