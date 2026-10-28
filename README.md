# OptDEM: Platform-Level Co-registered Optical–DEM Patches

## Release scope

This release provides cropped Landsat 8 Operational Land Imager (OLI) surface reflectance imagery (Collection 2, Tier 1, Level-2) and corresponding digital elevation model (DEM) data in GeoTIFF format for optical–DEM cross-modal registration research. The files retain platform-level georeferencing and initial co-registration. **They are not manually fine-registered ground-truth pairs.** Matching geographic grids do not guarantee pixel-level correspondence between terrain features in the two modalities.

**This public release contains 2,000 optical–DEM pairs:** 2,000 optical GeoTIFFs and 2,000 corresponding DEM GeoTIFFs. Pair files by their numeric identifier, not by arbitrary directory-list order. The release provides source data that readers can process into image representations and fine-registered pairs suited to their research. Its scope is these 2,000 platform-level co-registered pairs; refined pairs, control-point annotations, ground-truth transformation matrices, and official experiment split lists are not included.

## Download

This release contains **2,000 optical–DEM pairs**, distributed in four independent ZIP archives (500 pairs per archive).

1. Open [Releases](https://github.com/RSDPLab/OptDEM_Dataset/releases).
2. Download `OptDEM_part01.zip`, `OptDEM_part02.zip`, `OptDEM_part03.zip`, and `OptDEM_part04.zip` from the release assets. GitHub's automatically generated **Source code** archives do not contain the TIFF dataset.
3. Extract all four archives into the same destination directory. They share the `OptDEM_raw/` root folder and contain distinct image pairs. After extraction, `optical_sampled/` and `dem_sampled/` each contain 2,000 TIFF files. An individual archive can also be extracted independently to use its 500 pairs.

`SHA256SUMS.txt` provides checksums for verifying the downloaded archives.

## Directory structure

```text
OptDEM_raw/
  README.md
  optical_sampled/
    optical_0001.tif
    ...
  dem_sampled/
    dem_0001.tif
    ...
```

For example, `optical_sampled/optical_0001.tif` corresponds to `dem_sampled/dem_0001.tif`. Keep numeric identifiers unchanged when deriving hillshade, registered imagery, or annotations.

## Derived images and representations

Users can generate the following representations from the supplied data; these derived images are not included as precomputed products.

| Source | Derived representation | Processing notes |
| --- | --- | --- |
| DEM | Elevation grayscale images or color-relief maps | Map valid elevations to a chosen display range or color scale. A display image does not replace the original elevation values. |
| DEM | Hillshade images | Specify illumination azimuth and elevation, elevation scaling, and horizontal/vertical units. Multiple illumination directions can provide alternative terrain-structure representations. |
| DEM | Slope and aspect maps | Compute terrain derivatives with appropriate horizontal distances. Record slope units and aspect conventions, including handling of flat areas. |
| DEM | Curvature, terrain-gradient, or other terrain descriptors | Choose an appropriate derivative method and spatial scale. Small patches may require neighboring elevation data to reduce boundary artifacts. |
| Registered pairs | Fine-registered images, masks, and transformed experimental pairs | Save registration parameters and valid-data masks. For synthetic geometric perturbations, retain the known transformation and its direction. |

QGIS, GDAL, rasterio, and suitable image-processing libraries can be used for these tasks. GeoTIFF is recommended when retaining georeferencing; PNG or other image formats may be generated for model input or visualization, with coordinate metadata stored separately. Preserve numeric pair identifiers and document all processing parameters. 

## Suggested fine-registration workflow

Users may perform their own fine registration and construct derived datasets:

1. Read matching optical and DEM identifiers. Inspect valid-data masks, coordinate systems, geographic footprints, resolution, and elevation reference. Preserve the original files.
2. Generate a hillshade representation from the DEM, recording illumination azimuth/elevation and the treatment of horizontal and vertical units. For geographic-coordinate DEMs, use a suitable projected grid or a method that handles angular horizontal units correctly.
3. Match stable terrain structures between the optical image and DEM-derived hillshade. Reject unreliable correspondences and manually verify spatially distributed control points as needed. Hillshade is an intermediate structural representation, not a replacement for the elevation raster.
4. Following the manuscript's approach, estimate a six-parameter 2-D affine transformation from the refined control points. Record coordinate conventions, transformation direction, and whether the matrix acts on pixel or map coordinates. A transformation estimated on a reprojected hillshade must be converted appropriately before application to a DEM on a different grid.
5. Apply the spatial transformation to the DEM using a defined output grid and documented resampling method. This adjusts spatial alignment; it does not establish vertical accuracy or correct the DEM's elevation datum.
6. Assess residual alignment with independent checkpoints excluded from transformation fitting. Save control points, transformations, quality measures, masks, and provenance alongside derived pairs. A fitted transformation is not automatically verified ground truth.

These steps describe an intended processing route, not operations already applied to every file in this release. Registration quality and downstream results depend on the user's processing and quality control.

## Minimal loading example

```python
from pathlib import Path
import rasterio

root = Path('OptDEM_raw')
sample_id = '0001'
with rasterio.open(root / 'optical_sampled' / f'optical_{sample_id}.tif') as src:
    optical = src.read()  # bands, height, width
    optical_metadata = src.profile
with rasterio.open(root / 'dem_sampled' / f'dem_{sample_id}.tif') as src:
    dem = src.read(1)
    dem_metadata = src.profile
```

## Citation and reuse

The associated manuscript provides methodological context. Full bibliographic details and the release license should be supplied by the dataset author when publishing this package. This README does not grant additional rights over the upstream optical or elevation products; their applicable terms remain relevant.
