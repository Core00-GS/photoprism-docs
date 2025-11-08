---
hide:
- toc
---

# Mobile Device Resolutions

| Device              | Native      | DPR[^1] | Virtual     | Additional Info                                                     |
|---------------------|-------------|---------|-------------|---------------------------------------------------------------------|
| Google Pixel 10     | 1080 × 2424 | 2.625   | 411 × 923   | HDR; 24-bit color depth                                             |
| Google Pixel 10 Pro | 1280 × 2856 | 3.0     | 427 × 952   | LTPO OLED; HDR                                                      |
| iPad Air (11-inch)  | 1640 × 2360 | 2.0     | 820 × 1180  | Liquid Retina; Wide color (P3)                                      |
| iPad Air (13-inch)  | 2048 × 2732 | 2.0     | 1024 × 1366 | Liquid Retina; Wide color (P3)                                      |
| iPad Pro (11-inch)  | 1668 × 2420 | 2.0     | 834 × 1210  | Ultra Retina XDR (Tandem OLED); Wide color (P3); ProMotion 10–120Hz |
| iPad Pro (13-inch)  | 2064 × 2752 | 2.0     | 1032 × 1376 | Ultra Retina XDR (Tandem OLED); Wide color (P3); ProMotion 10–120Hz |
| iPad mini           | 1488 × 2266 | 2.0     | 744 × 1133  | Liquid Retina; Wide color (P3)                                      |
| iPhone 12           | 1170 × 2532 | 3.0     | 390 × 844   | Super Retina XDR; P3; True Tone; HDR                                |
| iPhone 12 mini      | 1080 × 2340 | 2.88    | 375 × 812   | Super Retina XDR; P3; True Tone; HDR (native scale is 2.88×)        |
| iPhone 13           | 1170 × 2532 | 3.0     | 390 × 844   | Super Retina XDR; P3; True Tone; HDR                                |
| iPhone 14           | 1170 × 2532 | 3.0     | 390 × 844   | Super Retina XDR; P3; True Tone; HDR                                |
| iPhone 15           | 1179 × 2556 | 3.0     | 393 × 852   | Super Retina XDR; P3; True Tone; HDR                                |
| iPhone 15 Pro       | 1179 × 2556 | 3.0     | 393 × 852   | Super Retina XDR; P3; True Tone; ProMotion 1–120Hz; HDR             |
| iPhone 16           | 1179 × 2556 | 3.0     | 393 × 852   | Super Retina XDR; P3; True Tone; HDR                                |
| iPhone 16e          | 1170 × 2532 | 3.0     | 390 × 844   | Super Retina XDR; P3; True Tone; HDR                                |
| iPhone 16 Pro       | 1206 × 2622 | 3.0     | 402 × 874   | Super Retina XDR; P3; True Tone; ProMotion 1–120Hz; HDR             |
| iPhone 17           | 1206 × 2622 | 3.0     | 402 × 874   | Super Retina XDR; P3; True Tone; 120Hz; HDR                         |
| iPhone 17 Air       | 1260 × 2736 | 3.0     | 420 × 912   | Super Retina XDR; P3; True Tone; ProMotion 1–120Hz; HDR             |
| iPhone 17 Pro       | 1206 × 2622 | 3.0     | 402 × 874   | Super Retina XDR; P3; True Tone; ProMotion 1–120Hz; HDR             |

## Display & DPR Notes

- **Virtual resolution = physical pixels ÷ device-pixel ratio (DPR).**  
  Use this to reason about CSS layout size. It represents the logical coordinate space a browser exposes.

- **iPhone DPR conventions.**  
  Current iPhone models typically render at **3× DPR**. The **iPhone 12/13 mini** use a native scale of **2.88×**, which yields a virtual resolution of **375 × 812** even though many APIs round DPR to 3.0.

- **iPad DPR conventions.**  
  Current iPads render at **2× DPR**. The listed Air/Pro/mini values reflect Apple’s default render scale and logical points used by iPadOS and Safari.

- **Android/Pixels may vary.**  
  The reported DPR can differ by device settings (Display size, font scaling), OEM defaults, and browser zoom. Values provided for Pixel devices reflect common defaults; teams should **detect DPR at runtime** for precise behavior.

- **Orientation does not change pixel counts.**  
  Native pixel dimensions are fixed; orientation only swaps width/height. Virtual resolution follows the same rule.

- **Color spaces and HDR.**  
  Modern iPhones and iPads support **Wide color (Display P3)** and system-level **HDR** playback/tonemapping; Pro models add **ProMotion** (high refresh rate). Many recent Android flagships support HDR and wide-gamut panels; verify support via runtime feature detection.

- **Design guidance.**  
  Prefer **fluid layouts** and **responsive images** (`srcset`, `sizes`, CSS `image-set()`) over device-specific breakpoints. Use **CSS media queries** for resolution/refresh-rate hints and **feature detection** for color-gamut/HDR.

- **Testing & telemetry.**  
  When accurate sizing matters (e.g., canvas, WebGL, photo grids), read `window.devicePixelRatio`, `screen.width/height`, and use `matchMedia()` for resolution/gamut queries. Log real-world DPR/viewport metrics to catch OEM/setting differences.

- **Terminology.**  
  “Native resolution” in this table refers to the panel’s **physical pixel matrix**; “virtual resolution” refers to the **CSS pixel** space (logical points). The two are related by DPR.

- **Future drift.**  
  Hardware and firmware updates can change reported DPR or color-management behavior. Treat these values as **baselines** and validate against actual devices during QA.

[^1]: Device pixel ratio (DPR) is the ratio of a device's physical pixels to its CSS pixels, meaning it determines how many physical pixels are used to render a single CSS pixel.
