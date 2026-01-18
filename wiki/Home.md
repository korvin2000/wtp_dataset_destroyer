# **What is PepeDD (Wtp Dataset Destroyer):**
PepeDD is a library for generating paired datasets with diverse and complex degradations that mimic real-world distortions found in images and videos from various sources. In other words, you take high-quality (HQ) source data and use it to create either a paired dataset or only degraded data (LQ). As a result, you obtain a dataset where LQ serves as the neural network input, and HQ represents the expected output.

## **Key features of the library:**

* **Complex degradation pipelines:** combining multiple transformations with controllable probabilities and parameters, applied strictly in sequence.
* **Flexible configuration:** built on Pydantic; settings can be defined either as objects or dictionaries, simplifying integration into different projects.
* **Extensibility:** the ability to create and plug in custom degradations as modular add-ons.
* **Experiment reproducibility:** control over the random number generator seed to precisely reproduce results and experiments.

## **Conceptual diagram:**

```
Input Image
    │
    ▼
[Degradation Pipeline]
    ├─ Blur (motion, gaussian, defocus)
    ├─ Compression (JPEG, WebP)
    └─ ...
    │
    ▼
Output Image
```

Each block in the pipeline can be configured via a configuration that defines its parameters, application probability, and execution order.
