# Compress configuration

```py
from pepedd.pipeline.schema import Degradation
from pepedd_nodes.compress import CompressOptions, TargetCompress

compress = Degradation(
    type="compress",
    options=CompressOptions(
        algorithm=["jpeg", "webp", "avif"],
        samplings=["444", "422", "420"],
        ffmpeg_samplings=["yuv420p_g", "yuv444p_g"],
        compress=[40, 95],
        quantize_table=["default", "mssim"],
        target_compress=TargetCompress(
            jpeg=[45, 80],
            avif=[20, 35],
        ),
        preset=["faster"],
        tune=["grain"],
        probability=1.0,
        seed=None,
    ),
)
```

## Parameters

* **algorithm** — List of compression algorithms to pick from. Supported:
  `jpeg`, `webp`, `h264`, `hevc`, `mpeg2`, `mpeg4`, `vp9`, `avif`, `j2000`, `bd`.
* **samplings** — JPEG subsampling list (`"444"`, `"440"`, `"441"`, `"422"`, `"420"`, `"411"`, `"410"`).
* **ffmpeg_samplings** — For video codecs (H.264/HEVC/VP9/AVIF), a list of ffmpeg pixel formats with optional interpolation suffixes (e.g., `yuv420p_g`).
* **compress** — Default quality range. Used when a specific algorithm is not specified in `target_compress`.
* **quantize_table** — JPEG quantization table presets.
* **target_compress** — Per-algorithm overrides (e.g., `jpeg=[40, 80]`).
* **preset / tune** — Codec-specific settings (validated per codec).
* **probability** — Activation probability.
* **seed** — Optional per-node seed.

## Behavior notes

* One algorithm from `algorithm` is chosen per image.
* For video codecs, invalid pixel formats are filtered to known-good ones; a fallback format is used if none remain.
* Compression operates on the LQ image only.

## HCL example

```hcl
degradation {
  type = "compress"
  options = {
    algorithm = ["jpeg", "webp", "avif"]
    compress = [40, 95]
    samplings = ["444", "422", "420"]
    quantize_table = ["default", "mssim"]
    target_compress = {
      jpeg = [45, 80]
      avif = [20, 35]
    }
    probability = 0.6
  }
}
```
