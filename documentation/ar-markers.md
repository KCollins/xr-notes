# A-Frame AR Markers

## What Are These Markers?

A-Frame's marker-based AR uses black-and-white fiducials that encode a numeric ID. When the webcam feed contains one of these high-contrast squares, AR.js (the AR engine behind A-Frame's marker components) decodes the marker and aligns a 3D model relative to that marker's coordinate frame. Each marker acts like a physical anchor for placing virtual content on a tabletop or printed sheet.

## Marker Format: 5×5 BCH (22, 7, 7)

For this project we standardize on the **5×5 BCH (Bose–Chaudhuri–Hocquenghem) code** with parameters **(22, 7, 7)**. Key reasons:

- **Error correction**: BCH encoding adds redundancy so the system can still recognize a marker if the camera sees it at oblique angles, with blur, or partially occluded.
- **Discrete ID space**: The 5×5 grid yields 25 bits, but only 7 are payload bits; the rest provide error detection/correction. This keeps IDs resilient without needing huge markers.
- **Compatibility**: AR.js ships with built-in support for this marker family, so it works out of the box in both desktop browsers and mobile WebXR sessions.

## How Many Unique Markers?

The BCH (22, 7, 7) scheme encodes up to **2⁷ = 128** unique IDs. In practice AR.js/ARToolKit reserves one ID (usually 0), leaving **127 usable markers** for custom content. This gives plenty of variety while keeping recognition reliable.

## Generate Your Own

You can create printable markers with the AR.js-compatible generator at:

- <https://au.gmented.com/app/marker/marker.php>

Steps:
1. Choose a `5x5` marker grid with `BCH (22,7,7)` encoding (error checking and correction type) for more reliable tracking.  
> If you need a larger ID pool and are willing to trade some tracking and occlusion resilience, consider using a larger `Matrix` code such as `3x3` or `4x4`, which can also be easily hand-drawn.
2. Enter the marker ID you need (1–127).
3. Export as SVG or PNG for high-quality prints.

Print the markers with good contrast on matte paper and keep them flat for best tracking performance.
