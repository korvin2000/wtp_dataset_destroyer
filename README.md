# WTP Dataset Destroyer
A tool for creating paired image datasets with configurable degradation Degradations. Perfect for training image restoration and enhancement models.

## Documentation
### Install the core:
```bash
uv pip install pepedd
```

### Install basic nodes (degradations):
```bash
uv pip install "pepedd[nodes]"
# or
uv pip install pepedd_nodes
```

### Basic Setup
- TODO: Add "Basic Information" information
- TODO: Add "Common Parameters" information 

### Degradations
- TODO: add degradations: "Blur", "Compress","Resize", "Dithering", "Halo", "Lines", "logical operations", etc.

### Development
- TODO: how to add custom degradations and custom nodes

## TODO:
* Add more degradation Degradations

## Credits:
- Resize is implemented through [chainner rs](https://github.com/chaiNNer-org/chaiNNer-rs)
- Compression based on [Kim2091's implementation](https://github.com/Kim2091/helpful-scripts/blob/d413054eda3764fd04ec2c22fb3c3b6a5e61e31a/Dataset%20Destroyer/datasetDestroyer.py#L279)
