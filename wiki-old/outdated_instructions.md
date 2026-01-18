# WTP Dataset Destroyer = DATA PREPARATION for Training
# author: Umzi
A tool for creating paired image datasets with configurable degradation Degradations. Perfect for training image restoration and enhancement models.

## Documentation
### Requirements install
```bash
pip install -r requirements.txt
# or
python3 -m pip install -r requirements.txt
```
### Basic Setup
- [Basic Information](https://github.com/umzi2/wtp_dataset_destroyer/blob/master/instructions/basic_info.md)
- [Common Parameters](https://github.com/umzi2/wtp_dataset_destroyer/blob/master/instructions/common.md)

### Degradations
- [Blur](https://github.com/umzi2/wtp_dataset_destroyer/blob/master/instructions/blur.md)
- [Canny](https://github.com/umzi2/wtp_dataset_destroyer/blob/master/instructions/canny.md)
- [Color](https://github.com/umzi2/wtp_dataset_destroyer/blob/master/instructions/color.md)
- [Compress](https://github.com/umzi2/wtp_dataset_destroyer/blob/master/instructions/compress.md)
- [Dithering](https://github.com/umzi2/wtp_dataset_destroyer/blob/master/instructions/dithering.md)
- [Halo](https://github.com/umzi2/wtp_dataset_destroyer/blob/master/instructions/halo.md)
- [Noise](https://github.com/umzi2/wtp_dataset_destroyer/blob/master/instructions/noise.md)
- [Pixelate](https://github.com/umzi2/wtp_dataset_destroyer/blob/master/instructions/pixelate.md)
- [Resize](https://github.com/umzi2/wtp_dataset_destroyer/blob/master/instructions/resize.md)
- [Saturation](https://github.com/umzi2/wtp_dataset_destroyer/blob/master/instructions/saturation.md)
- [Screentone](https://github.com/umzi2/wtp_dataset_destroyer/blob/master/instructions/screentone.md)
- [Shift](https://github.com/umzi2/wtp_dataset_destroyer/blob/master/instructions/shift.md)
- [Sin](https://github.com/umzi2/wtp_dataset_destroyer/blob/master/instructions/sin.md)
- [Subsampling](https://github.com/umzi2/wtp_dataset_destroyer/blob/master/instructions/subsampling.md)
- [And Or operations](https://github.com/umzi2/wtp_dataset_destroyer/blob/master/instructions/logical_operations
)
### Development
- [Add Degradation](https://github.com/umzi2/wtp_dataset_destroyer/blob/master/instructions/creators/add_degradation.md)

## TODO:
* Add more degradation Degradations

## Credits:
- Resize is implemented through [chainner rs](https://github.com/chaiNNer-org/chaiNNer-rs)
- Compression based on [Kim2091's implementation](https://github.com/Kim2091/helpful-scripts/blob/d413054eda3764fd04ec2c22fb3c3b6a5e61e31a/Dataset%20Destroyer/datasetDestroyer.py#L279)
This document covers basic information on how to configure `wtp_dataset_destroyer`.

### Common Parameters
- All Degradations support these basic parameters:
  - `probability`: Chance to apply the effect (0.0-1.0)
  - `type`: Name of the effect (required)
  - Parameters marked with `*` are optional

### Available Degradations
   - `blur`: Gaussian and motion blur
   - `noise`: Various noise patterns
   - `compress`: JPEG/HEVC compression artifacts
   - `pixelate`: Mosaic/pixel art effect
   - `color`: Brightness and contrast
   - `saturation`: Color intensity
   - `dithering`: Color reduction with patterns
   - `subsampling`: Color channel degradation
   - `screentone`: Manga-style patterns
   - `halo`: Light/dark edge artifacts
   - `sin`: Sinusoidal patterns
   - `shift`: Color channel misalignment
   - `resize`: Resolution changes
   - `and`:  If the first degradation group triggers, the second one will also be executed.
   - `or`: If the first degradation group does not trigger, the second one will be executed.
### Disabling degradations
To disable degradations, simply delete the section from your config file. This is what a section looks like:
```hcl
degradation {
   type = "sin"
   shape = [100,1000,100]
   alpha = [0.1,0.5]
   bias = [-1,1]
   vertical = 0.5
   probability = 0.5
}
```
Notice the { } enclosing the section. This defines a valid section that you can remove.

### Rearranging degradation order
To change the order that degradations are applied in, simply re-arrange sections in the config. Refer to the section example described above.

### Using config
```bash 
python destroyer.py -f configs/default.py
```
It is not necessary to specify the path to the config; by default it uses the path configs/default.py

### Tips
1. Start with low probabilities (0.2-0.5) when combining multiple Degradations
2. Test Degradations individually before combining
3. Some Degradations work better in specific orders
4. Check example images in each effect's documentation
### common / base configuration block
```hcl
input = "path/to/input"
output = "path/to/output"
degradation {}...
tile = {
  size = 512
  no_wb = true
}
num_workers = 16
map_type = "thread"
laplace_filter = 0.02
size = 1000
shuffle_dataset = true
gray = false
gray_or_color = true
debug = false
only_lq = false
real_name = false
out_clear = true
```

# Common Parameters and Concepts

## Parameter Formats

### Ranges
Many effects use range formats for randomization:
- `[min, max]`: Random value between min and max
- `[min, max, step]`: Random value between min and max in steps
  - Example: [1, 5, 2] means choose from [1, 3, 5]

### Probabilities
- All effects support `probability`
- Format: 0.0 to 1.0
  - 0.0 = Never apply
  - 0.5 = 50% chance
  - 1.0 = Always apply
- Default: 1.0 if not specified

### Color Spaces
Several effects work in different color spaces:
1. RGB (Red, Green, Blue)
   - Standard color space
   - Each channel 0-255
   - Used by most effects

2. YUV/YCbCr
   - Y: Brightness
   - U/Cb: Blue difference
   - V/Cr: Red difference
   - Used by: compress, shift, subsampling

3. CMYK
   - C: Cyan
   - M: Magenta
   - Y: Yellow
   - K: Black
   - Used by: screentone, shift

### Image Types
Effects handle different image types:
- Color (RGB/BGR): 3 channels
- Grayscale: 1 channel
- Alpha channel: Preserved if present

### Optional Parameters
- Marked with `*` in documentation
- Have default values
- Can be omitted from configuration

### Effect Order
- Effects are applied in sequence
- Order can affect final result
- Some effects work better early/late in chain

### Examples:
- `input` - input folder
- `output` - output folder (saves new HQ and LQ images)
- `degradation` - The degradations that will be applied to your images
- `num_workers`* - The number of processes for parallel processing. It's best to use your CPU core count
- `map_type`* - Valid processing types: `process`, `thread` and `for`
  - If you run into issues on Windows, swap to `thread`
- `size`* - How many images to process from the input folder 
- `laplace_filter`* - It filters out images based on low saturation using the laplace operator. Accepts a float value. (experiment with it)
- `shuffle_dataset`* - Determines whether or not the images will be shuffled
- `tile`* - This section enables automatic tiling of your images. For example, if you choose `size` 512, it'll split a single image into multiple 512x512 images.  
  - `size` - tile size
  - `no_wb`* - Ignore pure white and pure black images (bool)
- `gray`* - All images are read only in grayscale mode
- `gray_or_color`* - If an image has shades transitioning smoothly from light to dark and all RGB values for each pixel are almost the same, it gets converted to grayscale.
  - This is very performance intensive.
- `debug`* - Creates a `debug` folder if it doesn't exist, and in it creates a `debug.log` file where all random values during degradation processes will be logged. When enabled, it ignores map_type by setting it to `for`.
- `out_clear`* - Cleans the output directory out_path/lq|hq if it exists and contains files. Just to make the tests easier

Doesn't work with tile:
- `only_lq`* - Saves only lq files without hq. `spread` in resize causes discrepancies, so turn it off
- `real_name`* - When saving, the names do not change
### logical operations
```hcl
  degradation {
    type = "or" #and
    probability_one = 0.5
    probability_two = 0.5
    one_degradation { ... }
    two_degradation { ... }
  }
```
### Type
- `or` - if the first degradation group does not trigger, the second one will be executed.
- `and` - if the first degradation group triggers, the second one will also be executed.
### Probability
- `probability_one` - the chance of triggering the first degradation group (e.g., 0.5 = 50%).
- `probability_two` - depending on the type, if the process moves to the second degradation group, it will trigger with this probability (e.g., 0.5 = 50%).
### Degradatio
- `one_degradation` - a group of degradations that execute if probability_one is triggered. It works the same way as regular degradations but uses a different name.
- `two_degradation` - the second group of degradations, working on the same principle.
- Both `one_degradation` and `two_degradation` can be defined multiple times, for example:
```hcl
one_degradation { ... }
one_degradation { ... }
two_degradation { ... }
two_degradation { ... }
```


### "resize" degradation operations
```hcl
degradation {
  type = "resize"
  alg_lq = ["box", "hermite", "linear", "lagrange", "cubic_catrom", "cubic_mitchell", "cubic_bspline",
    "lanczos", "gauss", "down_up", "down_down", "up_down"]
  alg_hq = ["lagrange"]
  down_up = {
    down = [1, 2]
    alg_up = ["nearest", "box", "hermite", "linear", "lagrange", "cubic_catrom", "cubic_mitchell",
      "cubic_bspline", "lanczos", "gauss", "down_down"]
    alg_down = [ "hermite", "linear",  "lagrange", "cubic_catrom", "cubic_mitchell", "cubic_bspline",
      "lanczos", "gauss"]
  }
  up_down = {
    up = [1, 2]
    alg_up = ["nearest", "box", "hermite", "linear", "lagrange", "cubic_catrom", "cubic_mitchell",
      "cubic_bspline", "lanczos", "gauss"]
    alg_down = [ "hermite", "linear",  "lagrange", "cubic_catrom", "cubic_mitchell", "cubic_bspline",
      "lanczos", "gauss","down_down"]
  }
  down_down = {
    step = [1, 6]
    alg_down = [ "linear", "lagrange", "cubic_catrom", "cubic_mitchell", "cubic_bspline"]
  }
```

### Basic Parameters
- `spread`* - Controls the resolution reduction range
  - Format: [min, max, step]
  - Default: [1, 1, 1]
  - Example: [1, 4, 1] means:
    - For HQ image of size 512x512 and scale=4:
    - Original LQ would be 128x128
    - With spread, LQ size can vary from 128x128 to 32x32
    - Step of 1 means it can be 128, 96, 64, or 32
  - Larger spread values create more aggressive downscaling

- `probability`* - Controls how often resize is applied
  - Default: 1.0 (always apply)
  - Range: 0.0 to 1.0
  - Example: 0.5 means 50% chance to apply resize


### Algorithm Selection
- `alg_lq` - Algorithms for low quality image. One is randomly picked per image
- `alg_hq` - Algorithms for high quality image. One is randomly picked per image
- `scale` - Base scaling factor
### Up-Down Resize
- `up_down` - This algorithm increases the resolution of the image through resizing, then downsizes it again.
- `up` - Scale range for up-down
- `alg_up` - Upscaling algorithms
- `alg_down` - Downscaling algorithms
### Down-Up Resize
- `down_up` - This algorithm reduces the resolution of an image by resizing it and then increases its size again.
- `down` - Scale range for down-up
- `alg_up` - Upscaling algorithms
- `alg_down` - Downscaling algorithms

### Down-Down Resize
- `down_down` - This algorithm downsizes an image through resizing, then downsizes it again.
- `step` - Scale step size
- `alg` - Downscaling algorithms

### Options
- `color_fix`* - Fix colors after resize
- `gamma_correction`* - Apply gamma correction
- `olq`* - Only resize low quality image

## Examples:
<div> Raw</div>
<img src="images/resize/raw.png" title="raw_img">
<div> Box scale = 4</div>
<img src="images/resize/box_4.png" title="box_img">
### Subsampling operations
```hcl
{
degradation {
  type = "subsampling"
  down = ["box", "nearest", "linear",  "lagrange", "cubic_catrom", "cubic_mitchell", "cubic_bspline",
    "lanczos", "gauss"]
  up = ["box",  "linear", "nearest", "lagrange", "cubic_catrom", "cubic_mitchell", "cubic_bspline",
    "lanczos", "gauss"]
  sampling = ["4:4:4", "4:2:2", "4:2:1", "4:1:1", "4:2:0", "4:1:0", "3:1:1"]
  yuv = ["601","709","2020"]
  blur = [0.0,4]
  probability = 0.5
}
```

### Basic Parameters
- `down`* - Downscaling algorithms for color channels
  - Default: ["nearest"]
  - Options:
    - "nearest": Fast but blocky
    - "bilinear": Smooth but can blur
    - "bicubic": Best quality but slower
  - One algorithm randomly selected per image

- `up`* - Upscaling algorithms for color channels
  - Default: ["nearest"]
  - Same options as `down`
  - Different up/down combinations create unique artifacts

### Color Settings
- `sampling`* - Chroma subsampling formats
  - Default: ["4:4:4"]
  - Options:
    - "4:4:4": No subsampling (full quality)
    - "4:2:2": Half horizontal chroma resolution
    - "4:2:0": Quarter chroma resolution
    - "4:1:1": Quarter horizontal chroma resolution
  - Lower ratios = more color artifacts

- `yuv`* - YUV color space standard
  - Default: ["601"]
  - Options:
    - "601": Standard for SD content (BT.601)
    - "709": Standard for HD content (BT.709)
    - "2020": Standard for UHD content (BT.2020)
  - Affects color conversion accuracy

### Enhancement
- `blur`* - Optional blur kernel size
  - Format: [min, max]
  - Default: None (no blur)
  - Must be odd numbers
  - Larger values create more blur
  - Helps reduce subsampling artifacts

- `probability`* - Chance of applying effect
  - Default: 1.0
  - Range: 0.0 to 1.0
### blur / blur-effect operations / degradations
```hcl
degradation {
  type = "blur"
  filter = ["box", "gauss", "median","lens","motion","random"]
  kernel = [0, 1]
  target_kernel = {
    box = [0,1]
    gauss = [0,1]
    median = [0,1]
    lens =[1,2]
    random = [0,1]
  }
  motion_size = [0,10]
  motion_angle = [-30,30]
  probability = 0.5
}
```

`*` = optional parameters

### Basic Parameters
- `filter` - List of blur algorithms to choose from. One is randomly selected per image
- `probability`* - Chance of applying the blur effect (e.g., 0.5 = 50% chance)

### Kernel Settings
- `kernel`* - Default kernel size range for all blur types: [min, max]
   - Used only when `target_kernel` is not specified for a blur type
   - All types except median blur support float numbers
   - Median blur requires odd integers

- `target_kernel`* - Specific kernel size ranges for each blur type:
  - `box`: Box blur kernel range
  - `gauss`: Gaussian blur kernel range
  - `median`: Median blur kernel range (must be odd integers)
  - `lens`: Lens blur kernel range
  - `random`: Random kernel blur range

### Motion Blur Settings
- `motion_size`* - Length of motion blur effect: [min, max]
  - Default: [1, 2]
- `motion_angle`* - Angle of motion blur in degrees: [min, max]
  - Default: [0, 1]

### Available Blur Types
1. `box` - Simple averaging blur
2. `gauss` - Gaussian blur for smooth, natural-looking effects
3. `median` - Removes noise while preserving edges
4. `lens` - Simulates camera lens blur
5. `motion` - Simulates movement blur
6. `random` - Uses randomly generated kernel

## Examples:


  <div> Raw</div>
  <img src="images/blur/raw.png" alt="raw" title="raw_img">
  <div> Box kernel: 2</div>
  <img src="images/blur/box_2.png" alt="box_2" title="box_img">
  <div> Gauss kernel: 2</div>
  <img src="images/blur/gauss_2.png" alt="gauss_2" title="gauss_img">
  <div> Lens kernel: 2</div>
  <img src="images/blur/lens_2.png" alt="lens_2" title="lens_img">
  <div> Median kernel: 3</div>
  <img src="images/blur/median_3.png" alt="median_3" title="median_img">
  <div> Motion size: 10, angle 30</div>
  <img src="images/blur/motion_10_30.png" alt="motion_10_30" title="motion_img">
### "pixelate" degradation operations
```hcl
degradation {
  type = "pixelate"
  size = [1, 10]
  probability = 0.5
}
```
`*` = optional parameters

### Parameters
- `size`* - Controls pixel block size
  - Format: [min, max]
  - Default: [1, 1]
  - Example: [1,10] means:
    - Original pixels are grouped into blocks
    - Block size varies from 1x1 (no effect) to 10x10
    - Each block becomes a single color (average)
    - Larger values create more obvious "mosaic" effect
  - Values ≤ 1 have no effect
  - Integer values work best

- `probability`* - Chance of applying effect
  - Default: 1.0
  - Range: 0.0 to 1.0

## Examples:
<div> Raw</div>
<img src="images/pixelate/raw.png" title="raw_img">
<div> Size 10 </div>
<img src="images/pixelate/size_10.png" title="size_img">
### "dithering" degradation operations
```hcl
degradation {
  type = "dithering"
  dithering_type = ["floydsteinberg", "jarvisjudiceninke", "stucki", "atkinson", "burkes", "sierra",
        "tworowsierra", "sierraLite","order","riemersma","quantize"]
  color_ch = [2,10]
  map_size = [2,4,8,16]
  history = [10, 15]
  ratio = [0.1,0.9]
  probability = 0.5
}

```

### Parameters
- `type_dither`* - Controls the dithering algorithm
  - Default: ["quantize"]

- `color_ch`* - Controls color depth reduction
  - Format: [min, max]
  - Default: [2, 10]
  - Range: 2-255
  - Example: [1,4] means:
    - 2 ch = Black & White only (2 colors)
    - 4 ch = 12 colors
    - 8 ch = 24 colors
    - 16 ch = 48 colors
  - Lower values create more obvious dithering

- `probability`* - Chance of applying effect
  - Default: 1.0
  - Range: 0.0 to 1.0

## Examples:
all color_ch = 8
<div> Raw</div>
<img src="images/dithering/raw.png" title="raw_img">
<div> Quantize</div>
<img src="images/dithering/quantize.png" title="quantize_img">
<div> SierraLite</div>
<img src="images/dithering/sierraLite.png" title="sierraLite_img">
<div> Jarvisjudiceninke</div>
<img src="images/dithering/jarvisjudiceninke.png" title="jarvisjudiceninke_img">
<div> sierra</div>
<img src="images/dithering/sierra.png" title="sierra_img">
<div> Stucki</div>
<img src="images/dithering/stucki.png" title="stucki_img">
<div> Tworowsierra</div>
<img src="images/dithering/tworowsierra.png" title="tworowsierra_img">
<div> Atkinson</div>
<img src="images/dithering/atkinson.png" title="atkinson_img">
<div> Floydsteinberg</div>
<img src="images/dithering/floydsteinberg.png" title="floydsteinberg_img">
<div> Burkes</div>
<img src="images/dithering/burkes.png" title="burkes_img">
<div> Order map_size: 8</div>
<img src="images/dithering/order.png" title="order_img">
<div> Riemersma history: 10 ratio: 0.5</div>
<img src="images/dithering/riemersma.png" title="riemersma_img">
### "saturation" degradation operations
```hcl
degradation {
  type = "saturation"
  rand = [0.5,1.0]
  probability = 0.5
}
```
`*` = optional parameters

### Parameters
- `rand`* - Controls color saturation level
  - Format: [min, max]
  - Default: [0.5, 1.0]
  - Range: 0.0-1.0
  - Example: [0.1,0.9] means:
    - Values randomly chosen between 0.1-0.9
    - 0.0 = Complete grayscale
    - 0.5 = Half saturation
    - 1.0 = Full original saturation
  - Lower values create faded/washed-out look

- `probability`* - Chance of applying effect
  - Default: 1.0
  - Range: 0.0 to 1.0
  - No effect on grayscale images

## Examples:
<div> Raw</div>
<img src="images/saturation/raw.png" title="raw_img">
<div> Rand 2</div>
<img src="images/saturation/rand_2.png" title="rand_2_img">
<div> Rand 0.5</div>
<img src="images/saturation/rand_0_5.png" title="rand_0_5_img">
### "color" degradation operations
```hcl
degradation {
  type = "color"
  high = [240,255]
  low = [0,15]
  gamma = [0.9,1.1]
  probability = 0.5
}
```

### Parameters
- `high`* - Controls how bright colors can get
  - Format: [min, max]
  - Default: [255, 255]
  - Range: 0-255
  - Example: [200,255] means:
    - Brightest pixels will be randomly mapped to values between 200-255
    - Creates washed-out or faded appearance
  - Lower values reduce contrast in bright areas

- `low`* - Controls how dark colors can get
  - Format: [min, max]
  - Default: [0, 0]
  - Range: 0-255
  - Example: [0,50] means:
    - Darkest pixels will be randomly mapped to values between 0-50
    - Creates lifted blacks or faded shadows
  - Higher values reduce contrast in dark areas

- `gamma`* - Controls overall image brightness curve
  - Format: [min, max]
  - Default: [1.0, 1.0]
  - Values < 1.0: Makes mid-tones darker
  - Values > 1.0: Makes mid-tones brighter
  - Affects middle brightness more than extremes

- `probability`* - Chance of applying effect
  - Default: 1.0
  - Range: 0.0 to 1.0

## Examples:


<div> Raw</div>
<img src="images/color/raw.png" title="raw_img">
<div> Low 100</div>
<img src="images/color/out_low_100.png" title="low_img">
<div> High 150</div>
<img src="images/color/out_high_150.png" title="high_img">
<div> Gamma 3</div>
<img src="images/color/gamma_3.png" title="gamma_img">
### "noise" degradation operations
```hcl
degradation {
  type = "noise"
  type_noise = ["perlinsuflet", "perlin", "opensimplex", "simplex",
    "supersimplex","uniform","salt","salt_and_pepper","pepper","gauss"]
  clip = [0.3,0.5]
  normalize = true
  y_noise = 0.3
  uv_noise = 0.3
  alpha = [0.01,0.5,0.01]
  scale {
    size = [1, 2]
    sigma = [1, 2]
    amount = [1, 3]
    probability = 0.4
  }
  motion {
    size = [1, 3]
    angle = [1, 360]
    sigma = [0,1]
    amount = [0,1]
    probability = 0.4
  }
  octaves = [1,10,1]
  frequency = [0.1,0.9,0.1]
  lacunarity = [0.01,0.5,0.01]
  probability_salt_or_pepper = [0,0.02]

  bias =  [-0.5, 0.5]
  probability =  1
}
```
`*` = optional parameters

### Basic Parameters
- `type` - The type of noise to apply. One is randomly selected from the list for each image
- `alpha`* - Controls noise intensity. Format: [min, max, step]
- `probability`* - Chance of applying the noise effect (e.g., 0.5 = 50% chance)
- `clip`* - Restricts noise to areas where image values are between [min, max]
- `bias`* - Offset added to noise values. Range: [-1, 1]
### Scale*
- `size`* - degree of increase in gauss or uniform noise.
- `sigma`* - Sharping size
- `amoiunt`* - Sharpe power
- `probability`* - Chance of applying scale (e.g., 0.5 = 50% chance)
### Motion*
- `size`* - Length of motion blur effect
- `angle`* - Angle of motion blur 
### Channel Control
- `y_noise`* - Probability of applying noise only to luminance (Y channel)
- `uv_noise`* - Probability of applying noise only to chrominance (UV channels)
- `sigma`* - Sharping size
- `amoiunt`* - Sharpe power
- `probability`* - Chance of applying scale (e.g., 0.5 = 50% chance)
### Procedural Noise Parameters
(For "perlinsuflet", "perlin", "opensimplex", "simplex", "supersimplex")
- `normalize`* - Normalizes noise to range [-1, 1]
- `octaves`* - Number of noise layers to combine. Format: [min, max, step]
- `frequency`* - Controls noise pattern size. Format: [min, max, step]
  - Lower values create larger patterns, higher values create finer details
- `lacunarity`* - Controls frequency change between octaves. Format: [min, max, step]
  - Higher values create more detail in subsequent octaves

### Salt and Pepper Parameters
(For "salt", "pepper", "salt_and_pepper")
- `probability_salt_or_pepper`* - Percentage of pixels affected by the noise
  - For "salt": pixels set to 1 (white)
  - For "pepper": pixels set to 0 (black)
  - For "salt_and_pepper": pixels randomly set to either 0 or 1

## Examples 1:
### all alpha: 0.5
<div> raw</div>
<img src="images/noise/raw.png" title="raw_img">
<div> uniform</div>
<img src="images/noise/uniform.png" title="uniform_img">
<div> gauss</div>
<img src="images/noise/gauss.png" title="gauss_img">

### all octaves: 10 frequency: 0.5 lacunarity: 0.5
<div> perlinsuflet</div>
<img src="images/noise/perlinsuflet.png" title="perlinsuflet_img">
<div> opensimplex</div>
<img src="images/noise/opensimplex.png" title="opensimplex_img">
<div> perlin</div>
<img src="images/noise/perlin.png" title="perlin_img">
<div> simplex</div>
<img src="images/noise/simplex.png" title="simplex_img">
<div> supersimplex</div>
<img src="images/noise/supersimplex.png" title="supersimplex_img">

### all probability_salt_or_pepper: 0.05
<div> salt</div>
<img src="images/noise/salt.png" title="salt_img">
<div> pepper</div>
<img src="images/noise/pepper.png" title="pepper_img">
<div> salt_and_pepper</div>
<img src="images/noise/salt_and_pepper.png" title="salt_and_pepper_img">

## Examples 2:
### all alpha = 0.5
<div> uniform y_noise = 1.0</div>
<img src="images/noise/uniform_y.png" title="uniform_y_img">
<div> uniform uv_noise = 1.0</div>
<img src="images/noise/uniform_uv.png" title="uniform_uv_img">
### introducing compression artfacts  / degradations
```hcl
degradation {
  type ="compress"
  algorithm = ["jpeg", "webp", "h264", "hevc", "mpeg2", "mpeg4", "vp9"]
  jpeg_sampling = [
    "4:4:4", "4:4:0", "4:2:2", "4:2:0"
  ]
  target_compress = {
    h264 = [23,32]
    hevc = [20,34]
    mpeg4 = [2,20]
    mpeg2 = [2,20]
    vp9 = [20,35]
    jpeg = [40,100]
    webp = [40,100]
  }
  compress = [40, 100]
  probability = 0.5
}
```

### Basic Parameters
- `algorithm` - List of compression algorithms to choose from
  - One is randomly selected per image
  - Note: HEVC encoding may cause errors (pts<dts or memory segmentation)

- `probability`* - Chance of applying compression
  - Range: 0.0 to 1.0
  - Default: 1.0
  - Example: 0.5 = 50% chance

### Compression Settings
- `target_compress`* - Algorithm-specific compression ranges
  - Format: `{"algorithm": [min, max]}`
  - Values are randomly selected from the specified ranges
  - If not specified for an algorithm, falls back to `comp` value

- `comp`* - Default compression range
  - Format: [min, max]
  - Default: [90, 100]
  - Used when algorithm has no `target_compress` entry

### JPEG-Specific Settings
- `jpeg_sampling`* - Chroma subsampling options
  - Default: ["4:2:2"]
  - Available options:
    - "4:4:4": No subsampling, highest quality
    - "4:4:0": Reduced vertical chroma resolution
    - "4:2:2": Horizontal chroma subsampling
    - "4:2:0": Both horizontal and vertical subsampling

### Compression Ranges by Algorithm
1. Image Formats:
   - JPEG: 40-100 (higher = better quality)
   - WebP: 40-100 (higher = better quality)

2. Video Codecs:
   - H.264: 23-32 (lower = better quality)
   - HEVC: 20-34 (lower = better quality)
   - VP9: 20-35 (lower = better quality)
   - MPEG4: 2-20 (lower = better quality)
   - MPEG2: 2-20 (lower = better quality)

### Examples:

<div> Raw</div>
<img src="images/compress/raw.png" title="raw_img">
<div> Jpeg 50 4:2:0</div>
<img src="images/compress/jpeg_50.png" title="jpeg_img">
<div> Jpeg 50 4:4:4</div>
<img src="images/compress/jpeg_50_4_4_4.png" title="jpeg_img">
<div> Webp 50</div>
<img src="images/compress/webp_50.png" title="webp_img">
<div> h264 32</div>
<img src="images/compress/h264_32.png" title="h264_img">
<div> Mpeg2 20</div>
<img src="images/compress/mpeg2_20.png" title="mpeg2_img">
<div> Vp9 35</div>
<img src="images/compress/vp9_35.png" title="vp9_img">
### "screentone" degradation operations
```hcl
degradation {
  type = "screentone"
  lqhq = false
  dot_size = [7]
    color {
      type_halftone = ["rgb","cmyk","gray","not_rot","hcl"]
      dot{
        angle = [-45,45]
        type = ["circle"]
      }
      dot{
        angle = [-45,45]
        type = ["ellipse"]
      }
      dot{
        angle = [-45,45]
        type = ["circle"]
      }
      dot{
        angle = [-45,45]
        type = ["circle"]
      }
      cmyk_alpha = [0.5,1.0]
    }
  dot_type = ["circle"]
  angle = [-45,45]
  probability = 0.5
}
```
`*` = optional parameters

### Basic Parameters
- `dot_size`* - Controls size of screentone pattern
  - Format: [min, max]
  - Default: [7, 7]
  - Larger values create bigger dots/patterns
  - Smaller values create finer details

- `dot_type`* - Controls shape of screentone pattern
  - Default: ["circle"]
  - Options:
    - "circle": Round dots (classic manga style)
    - "diamond": Diamond shapes
    - "line": Linear patterns
  - One type is randomly selected per image

- `angle`* - Controls rotation of pattern
  - Format: [min, max] in degrees
  - Default: [0, 0]
  - Range: 0-360
  - Affects pattern orientation

### Color Settings
- `color`* - These settings apply if the image has 3 channels
- `type_halftone`* - Color separation method
  - Default: ["rgb"]
  - Options:
    - "rgb": Red, Green, Blue separation
    - "cmyk": Cyan, Magenta, Yellow, Black separation

- Channel Angles (c, m, y, k, r, g, b)*
  - Format: [min, max] in degrees
  - Default: [0, 0] for each
  - Controls angle for each color channel
  - Different angles prevent moiré patterns

- `cmyk_alpha`* - CMYK color intensity
  - Format: [min, max]
  - Default: [1.0, 1.0]
  - Range: 0.0-1.0
  - Lower values create lighter colors

- `probability`* - Chance of applying effect
  - Default: 1.0
  - Range: 0.0 to 1.0

### Examples:
### all dot_size = 7
<div> raw</div>
<img src="images/screentone/raw.png" title="raw_img">
<div> not_rot</div>
<img src="images/screentone/not_rot.png" title="not_rot_img">
<div> gray</div>
<img src="images/screentone/gray.png" title="gray_img">
<div> rgb r = -30 g = 0 b = 30 </div>
<img src="images/screentone/rgb.png" title="rgb_img">
<div> cmyk c = -45 m = 30 y = 15 k = 15</div>
<img src="images/screentone/cmyk.png" title="cmyk_img">
<div> cmyk_alpha 0.8</div>
<img src="images/screentone/cmyk_alpha.png" title="cmyk_alpha_img">
### "halo" effect degradation operations
```hcl
degradation {
  type = "halo"
  type_halo = ["unsharp_mask","unsharp_halo","unsharp_gray"]
  kernel = [0,3]
  amount = [0,1]
  threshold = [0,0.5]
  probability = 0.5
}
```
### Parameters
- `type_halo`* - Controls how halos are generated
  - Default: ["unsharp_mask"]
  - Uses unsharp masking to create light/dark edges around objects
  - Creates the "glow" or "halo" effect often seen in over-processed images
  - Simulates over-sharpening artifacts

- `kernel`* - Controls the width of the halo effect
  - Format: [min, max]
  - Default: [0, 2]
  - Larger values create wider, more noticeable halos
  - Smaller values create thin, subtle edges

- `amount`* - Controls the intensity of the halo
  - Format: [min, max]
  - Default: [1, 1]
  - Higher values create stronger contrast at edges
  - Values > 2 create very obvious halos

- `threshold`* - Controls which edges get halos
  - Format: [min, max]
  - Default: [0, 0]
  - Range: 0.0-1.0
  - Only applies halos where pixel difference > threshold
  - Higher values only affect high-contrast edges

- `probability`* - Chance of applying effect
  - Default: 1.0
  - Range: 0.0 to 1.0

## Examples:
all kernel = 3 amount = 4
<div> Raw</div>
<img src="images/halo/raw.png" title="raw_img">
<div> Unsharp_halo</div>
<img src="images/halo/unsharp_halo.png" title="unsharp_halo_img">
<div> Unsharp_gray</div>
<img src="images/halo/unsharp_gray.png" title="unsharp_halo_img">
<div> Unsharp_mask threshold: 10</div>
<img src="images/halo/unsharp_mask.png" title="unsharp_mask_img">
### "canny" degradation operations
```hcl
degradation {
  type = "canny"
  thread1 = [10, 150, 1]
  thread2 = [10, 100, 1]
  aperture_size = [3, 5]
  scale = [0, 1, 0.25]
  white = 0.5
  lq_hq = true
  probability = 0.5
}
```
### Edge Detection Parameters
- `thread1`* - First threshold for the Canny edge detector
  - Format: [min, max, step]
  - Default: [10, 10, 1]
  - Controls edge sensitivity: lower values detect more edges

- `thread2`* - Second threshold for the Canny edge detector
  - Format: [min, max, step]
  - Default: [0, 10, 1]
  - Must be less than thread1
  - Helps connect edge segments

- `aperture_size`* - Size of the Sobel operator kernel
  - Format: [min, max]
  - Default: [3, 5]
  - Must be odd numbers (3, 5, 7)
  - Larger values detect stronger edges

### Appearance Control
- `scale`* - edge size control
  - Format: [min, max, step]
  - Default: [0, 1, 0.25]
  - If the value is 0, no outlines are applied; otherwise, the contour is increased to the rounded-up value of the contour. For example, if Canny typically produces a contour of 1 pixel, with a value of 3, the contour will be 3 pixels.
- `white`* - Controls edge appearance
  - Range: 0.0 to 1.0
  - Default: 0.0
  - Probability of showing edges on white background
  - 0.0 = black background
  - 1.0 = white background

### Processing Options
- `probability`* - Chance of applying the effect
  - Range: 0.0 to 1.0
  - Default: 1.0
  - Example: 0.5 = 50% chance

- `lq_hq`* - Output image control
  - Default: false
  - When true: processed image is used for both low and high quality
  - When false: original image is preserved as high quality

### Examples:
<div> Raw</div>
<img src="images/canny/raw.png" title="raw_img">
<div> canny thread1 = 150 thread2 = 100 aperture_size = 3 white = 0</div>
<img src="images/canny/canny_150_100_3.png" title="canny_150_100_3">
<div> canny thread1 = 150 thread2 = 100 aperture_size = 3 white = 1</div>
<img src="images/canny/canny_white_150_100_3.png" title="canny_white_150_100_3">
### "shift" degradation operations
```hcl
degradation {
  type = "shift"
  shift_type= ["cmyk","rgb","yuv"]
  percent = true
  rgb = {
    r = [[-10,10], [-10,10]]
    g = [[-10,10], [-10,10]]
    b = [[-10,10], [-10,10]]
  }
  cmyk = {
    c = [[-10,10], [-10,10]]
    m = [[-10,10], [-10,10]]
    y = [[-10,10], [-10,10]]
    k = [[0,0], [0,0]]
  }
  yuv = {
    y = [[-10,10], [-10,10]]
    u = [[-10,10], [-10,10]]
    v = [[-10,10], [-10,10]]
  }
  not_target = [[-10,10], [-10,10]]
  probability = 0.5
}
```
`*` = optional parameters

### Basic Parameters
- `shift_type`* - Color spaces to apply shifts in
  - Default: ["rgb"]
  - Options: "rgb", "yuv", "cmyk"
  - Multiple types can be used
  - Each type creates different artifacts

- `percent`* - Controls shift measurement
  - Default: false
  - true: Shifts are % of image size
  - false: Shifts are in pixels

- `probability`* - Chance of applying effect
  - Default: 1.0
  - Range: 0.0 to 1.0

### Channel Shift Settings
Each channel has format: [[x_min,x_max], [y_min,y_max]]
- x values control horizontal shift
- y values control vertical shift
- Positive = right/down shift
- Negative = left/up shift

#### RGB Shifts
- `rgb.r`* - Red channel shift range
- `rgb.g`* - Green channel shift range
- `rgb.b`* - Blue channel shift range
Example: [[0,5], [0,5]] means:
  - Random x shift: 0-5 pixels/percent right
  - Random y shift: 0-5 pixels/percent down

#### YUV Shifts
- `yuv.y`* - Luminance shift range
- `yuv.u`* - Blue-difference shift range
- `yuv.v`* - Red-difference shift range

#### CMYK Shifts
- `cmyk.c`* - Cyan shift range
- `cmyk.m`* - Magenta shift range
- `cmyk.y`* - Yellow shift range
- `cmyk.k`* - Black shift range

## Examples:
### all percent = true
<div> raw</div>
<img src="images/shift/raw.png" title="raw_img">
<div> rgb r = [[-1], [1]] g = [[1], [1]] b = [[-1], [-1]]</div>
<img src="images/shift/rgb.png" title="rgb_img">
<div> cmyk c = [[-1], [1]] m = [[1], [1]] y = [[-1], [-1]] k = [[0], [0]]</div>
<img src="images/shift/cmyk.png" title="cmyk_img">
<div> yuv y = [[-1], [1]] u = [[1], [1]] v = [[-1], [-1]]</div>
<img src="images/shift/yuv.png" title="yuv_img">
### SIN Degradation operations
```hcl
degradation {
   type = "sin"
   shape = [100,1000,100]
   alpha = [0.1,0.5]
   bias = [-1,1]
   vertical = 0.5
   probability = 0.5
}
```
### Parameters
- `shape`* - Controls sinusoidal pattern frequency
  - Format: [min, max, step]
  - Default: [100, 1000, 100]
  - Example: [100,1000,100] means:
    - Values: 100, 200, 300, ..., 1000
    - Lower values = wider waves
    - Higher values = tighter waves
  - Affects pattern density

- `alpha`* - Controls pattern intensity
  - Format: [min, max]
  - Default: [0.1, 0.5]
  - Range: 0.0-1.0
  - Lower values = subtle effect
  - Higher values = stronger effect

- `bias`* - Controls pattern brightness
  - Format: [min, max]
  - Default: [0.8, 1.2]
  - Values < 1.0 darken the pattern
  - Values > 1.0 brighten the pattern

- `vertical`* - Controls pattern orientation
  - Default: 0.5
  - Range: 0.0-1.0
  - Higher values = more likely vertical
  - Lower values = more likely horizontal

- `probability`* - Chance of applying effect
  - Default: 1.0
  - Range: 0.0 to 1.0

## Examples:
### all shape = 200  alpha = 0.3
<div> raw</div>
<img src="images/sin/raw.png" title="raw_img">
<div> sin</div>
<img src="images/sin/sin.png" title="sin_img">
<div> sin_vertical</div>
<img src="images/sin/sin_vertical.png" title="sin_vertical_img">
<div> sin_bias_0_5</div>
<img src="images/sin/sin_bias_0_5.png" title="sin_bias_0_5_img">
<div> sin_bias_-0_5</div>
<img src="images/sin/sin_bias_-0_5.png" title="sin_bias_-0_5_img">

## config for game textures
```hcl
input = "path/to/input"
output = "path/to/output"

degradation {
  type = "dithering"
  dithering_type =["floydsteinberg", "jarvisjudiceninke", "quantize"]
  color_ch = [32,128]
  probability = 0.5
}

degradation {
  type = "resize"
  alg_lq = ["box", "linear", "cubic_catrom", "lanczos"]
  alg_hq = ["lagrange"]
  spread = [1, 2, 0.05]
  scale = 4
  color_fix = true
  gamma_correction = false
}

degradation {
  type = "halo"
  type_halo = ["unsharp_mask"]
  kernel = [0,1]
  amount = [0,1]
  threshold = [0,0.079]
  probability = 0.4
}

degradation {
  type = "blur"
  filter = ["box", "gauss","lens"]
  kernel = [0, 1]
  target_kernel = {
    box = [0,2]
    gauss = [0,2]
    lens =[1,2]
  }
  probability = 0.15
}

degradation {
  type = "noise"
  type_noise = ["uniform", "gauss"]
  y_noise = 0.3
  uv_noise = 0.1
  alpha = [0.1,0.3,0.05]
  bias =  [-0.1, 0.1]
  probability =  0.35
}

laplace_filter = 0.02
num_workers = 8
map_type = "thread"
```
