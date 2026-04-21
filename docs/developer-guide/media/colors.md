# Color Profiles

## Standard RGB

[sRGB](https://en.wikipedia.org/wiki/SRGB) is the default color space used when [generating thumbnails](thumbnails.md). PhotoPrism and web browsers assume this color space for all pictures that do not have an embedded ICC color profile.

## ICC Profiles

An [ICC color profile](https://en.wikipedia.org/wiki/ICC_profile) for [wide-gamut displays](https://en.wikipedia.org/wiki/Gamut) can optionally be embedded in image and video files. PhotoPrism renders all thumbnails with `libvips` (via govips), which [preserves ICC profiles](https://github.com/photoprism/photoprism/issues/1474) beyond [sRGB](https://en.wikipedia.org/wiki/SRGB) and [Display P3](https://en.wikipedia.org/wiki/DCI-P3#P3_colorimetry) by default:

| Environment            | CLI Flag      | Default | Description                                                          |
|------------------------|---------------|---------|----------------------------------------------------------------------|
| PHOTOPRISM_THUMB_COLOR | --thumb-color | auto    | Standard color `PROFILE` for thumbnails (auto, preserve, srgb, none) |

`PHOTOPRISM_THUMB_LIBRARY` is retained for backwards compatibility but has no effect — libvips is the only supported image-processing library since the April 2026 release.

Image colors may otherwise not be displayed correctly, which is particularly noticeable with ProPhoto RGB and Adobe RGB, as these cover a wide range of colors:

![ICC Profiles](img/icc-profiles.svg)

## Color Detection

Color detection is performed while indexing using [a 3x3 thumbnail](thumbnails.md#standard-sizes) that covers the top, bottom, and center of an image e.g. <https://demo.photoprism.app/library/browse?color=green>.

[Learn more ›](../metadata/colors.md)
