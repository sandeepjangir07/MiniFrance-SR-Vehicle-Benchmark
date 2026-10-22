# Mini-France SR Vehicle-Detection Benchmark

A benchmark for evaluating **super-resolution (SR) and image-enhancement methods for a downstream task**: oriented vehicle detection in real, heavily compressed 50 cm aerial imagery.

> S. K. Jangir, M. N. Mühlhaus, C. Henry, N. Merkle, R. Bahmanyar. **U2D2: Dual-Conditioned Diffusion Refinement for Cross-Sensor Blind Super-Resolution of Aerial and Satellite Images.**

| | |
|---|---|
| Source imagery | [Mini-France](https://dx.doi.org/10.21227/b9pt-8x03), test split, RGB aerial orthophotos |
| Ground sampling distance | 50 cm (benchmark images) → 25 cm after 2× SR |
| Crops | 6 crops  at 1024 × 1024 px |
| Annotations | 1,941 vehicles, oriented quadrilaterals in DOTA format |
| Degradations | Heavy JPEG compression, aliasing and noise |
| Coordinate system | Pixel coordinates of each crop; source tiles in Lambert-93 (EPSG:2154) |

## Repository contents

```
crops.csv            crop windows in the original Mini-France images
imageset.txt         image list for the DOTA devkit evaluation
labels/              annotations at the native 50 cm resolution (1024 px crops)
labels_x2/           the same annotations scaled ×2, for 2× SR outputs (2048 px)
images/              the six 50 cm benchmark crops (GeoTIFF)
preview/             annotations drawn on each crop, for visual checking
```

## The crops

| Short name | Mini-France image | x | y | Width × height | Vehicles |
|---|---|---:|---:|---|---:|
| harbour | `13-2014-0860-6260-LA93-0M50-E080.jp2` | 880 | 2936 | 1024 × 828 | 706 |
| town1 | `29-2012-0150-6845-LA93-0M50-E080.jp2` | 7792 | 7422 | 1024 × 1024 | 232 |
| industrial1 | `35-2012-0315-6775-LA93-0M50-E080.jp2` | 3521 | 4120 | 1024 × 1024 | 191 |
| industrial2 | `35-2012-0345-6785-LA93-0M50-E080.jp2` | 5495 | 7126 | 1024 × 1024 | 358 |
| town2 | `49-2013-0395-6680-LA93-0M50-E080.jp2` | 5854 | 5686 | 1024 × 1024 | 209 |
| town3 | `49-2013-0425-6725-LA93-0M50-E080.jp2` | 6704 | 8646 | 1024 × 1024 | 245 |
| **Total** | | | | | **1,941** |

`x` and `y` are the column and row of the crop's top-left pixel in the full Mini-France image. The crops were chosen to cover the range of degradations in Mini-France: residential streets, industrial parking and a harbour car park.

## Annotation format

Labels use the [DOTA](https://captain-whu.github.io/DOTA/) format, one vehicle per line:

```
x1 y1 x2 y2 x3 y3 x4 y4 category difficult
720.75 490.25 721.85 486.05 730.35 488.40 729.05 492.40 car 0
```

- Coordinates are in pixels relative to the top-left corner of the crop (x to the right, y downwards).
- The first point is the **front-left** corner of the vehicle, and the remaining points follow **clockwise**, so the edge from point 1 to point 2 is the front of the vehicle.
- There is one category, `car`, covering passenger vehicles. `difficult` is always `0`; both fields exist only for DOTA compatibility.
- A label file has the same name as its image: `labels/<image_name>.txt` belongs to `images/<image_name>.tif`.
- Vehicles are small at 50 cm: median size is about 9 × 4.4 px (4.5 × 2.2 m).

`labels_x2/` holds the same boxes multiplied by 2. Use it when you evaluate 2× super-resolved images. For another factor, regenerate the labels with `--sr-scale` (see below).


## Evaluation protocol

To make results comparable with the paper:

1. **Super-resolve** each crop in `images/` by 2× (50 cm → 25 cm) with your method. Do not use any benchmark image for training or tuning.
2. **Detect** vehicles in the SR images with a detector trained only on native high-resolution imagery and kept **fixed** for all methods. In the paper this is DINO-OBB trained on the EAGLE dataset (5–45 cm GSD) with multi-scale training at {0.5, 1.0, 1.5}; it is never fine-tuned on SR output. That way, any difference in the scores is caused by the SR step alone.
3. **Score** the detections with the [DOTA devkit](https://github.com/CAPTAIN-WHU/DOTA_devkit) Task 1 evaluation (`dota_evaluation_task1.py`) at IoU ≥ 0.5, using `labels_x2/` as the annotation path, `imageset.txt` as the image set and `car` as the class.
4. Also report **orientation accuracy**: the share of correctly detected vehicles whose predicted heading is within ±10° of the ground truth.

`<CONFIRM: VOC07 11-point AP (devkit default) or area-under-curve AP>`

### Results from the paper

Fixed DINO-OBB detector, 2× SR from 50 cm to 25 cm.

| Method | AP₅₀ (%) | Orientation within ±10° (%) |
|---|---:|---:|
| Bicubic | 25.31 | 74.79 |
| EDSR | 22.49 | 76.04 |
| ESRGAN | 26.12 | 73.62 |
| Real-ESRGAN | 59.47 | 78.41 |
| SR3 | 47.43 | 72.75 |
| MAT | 25.95 | 74.61 |
| PFT | 29.53 | 76.46 |
| FMSR | 26.69 | 76.52 |
| IRR-DiffSR | 52.55 | 75.45 |
| DiffBSR | 60.98 | 75.56 |
| TexaDiff | 29.90 | 72.39 |
| **U2D2** | **70.70** | **80.85** |

## Citation

If you use this benchmark, please cite the paper and the Mini-France dataset:

```bibtex
@article{jangir2026u2d2,
  title   = {U2D2: Dual-Conditioned Diffusion Refinement for Cross-Sensor Blind Super-Resolution of Aerial and Satellite Images},
  author  = {Jangir, Sandeep Kumar and M{\"u}hlhaus, Manuel Nikolas and Henry, Corentin and Merkle, Nina and Bahmanyar, Reza},
  journal = {Remote Sensing},
  year    = {2026},
  note    = {<VOLUME, ARTICLE NUMBER, DOI>}
}

@misc{MiniFrance,
  author    = {Castillo Navarro, Javiera and Le Saux, Bertrand and Boulch, Alexandre and Audebert, Nicolas and Lef{\`e}vre, S{\'e}bastien},
  title     = {MiniFrance},
  publisher = {IEEE Dataport},
  year      = {2020},
  doi       = {10.21227/b9pt-8x03}
}
```

## License

The annotations in `labels/` and `labels_x2/` and the code in `scripts/` are released under `<LICENSE, e.g. CC BY 4.0 for annotations and MIT for code>`. The images are crops of the Mini-France dataset and remain subject to its original license; see the Mini-France page on IEEE DataPort.

## Contact

Sandeep Kumar Jangir, German Aerospace Center (DLR), Remote Sensing Technology Institute: sandeep.jangir@dlr.de
