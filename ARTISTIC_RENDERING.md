# Artistic SVG Rendering for Frame Proofs

This feature adds artistic, hand-drawn styled rendering to the geometric frame's inner patterns using Rough.js.

## Features

### Artistic Rendering Parameters

The following configurable parameters are available in the Designer UI under **Geometric Frame → Inner Pattern → Artistic Rendering**:

1. **Enable Artistic Mode** (checkbox)
   - Toggles between standard geometric rendering and artistic hand-drawn style
   - Only affects the inner pattern, not the QR code or outer borders

2. **Fill Style** (dropdown)
   - `solid`: Solid fill with rough edges
   - `hachure`: Cross-hatch pattern (default)
   - `zigzag`: Zigzag fill pattern
   - `cross-hatch`: Dense cross-hatching
   - `dots`: Dotted fill pattern
   - `dashed`: Dashed line pattern
   - `zigzag-line`: Zigzag line pattern

3. **Roughness** (slider: 0-5, default: 1.5)
   - Controls how "sketchy" the shapes appear
   - Higher values = more hand-drawn appearance
   - Lower values = closer to geometric precision

4. **Bowing** (slider: 0-10, default: 1)
   - Controls how much straight lines curve/bow
   - Higher values = more organic, flowing lines
   - Lower values = straighter lines

5. **Fill Weight** (slider: 1-10, default: 2)
   - Controls the stroke weight for fill patterns
   - Higher values = thicker, more visible fill lines
   - Lower values = finer, more delicate patterns

## How It Works

### Implementation Details

1. **Library**: Uses [Rough.js](https://roughjs.com/) - a lightweight (<9kB) graphics library for sketchy, hand-drawn graphics
2. **Integration**: Works seamlessly with existing canvas-based rendering
3. **Compatibility**: Supports all inner pattern types:
   - Circle
   - Square
   - Diamond
   - Module-based
   - Isometric Cube
   - Isometric Cylinder
   - Isometric Pyramid

### Rendering Pipeline

When artistic mode is enabled:

```
User Input → Frame Config → GeometricFrame.svelte
                                    ↓
                          drawInnerPattern()
                                    ↓
                    [Artistic Mode Check]
                                    ↓
                    Yes ─→ Rough.js Canvas Renderer
                     |           ↓
                     |    drawArtisticShape()
                     |           ↓
                     |    - drawArtisticIsometricCube()
                     |    - drawArtisticIsometricCylinder()
                     |    - drawArtisticIsometricPyramid()
                     |           ↓
                    No ─→ Standard Canvas Renderer
                                    ↓
                          Canvas Output with Artistic Effects
```

### Blending with QR Code Design

The artistic rendering is designed to blend harmoniously with QR codes:

1. **Clipping**: Inner patterns are clipped by both:
   - The outer shape boundary
   - The inner square void (where the QR code sits)

2. **Respects QR Module Styling**: The artistic renderer works with:
   - Rounding Amount
   - Padding Amount
   - Edge Bleed
   - Geometric Chaos

3. **Color Consistency**: Uses the same `innerColor` for both standard and artistic rendering

## Usage Examples

### Example 1: Sketchy Hachure Pattern
```javascript
frameConfig = {
  artisticEnabled: true,
  artisticRoughness: 2.5,
  artisticFillStyle: 'hachure',
  artisticFillWeight: 2,
  artisticBowing: 1.5
}
```
Creates a cross-hatched, hand-drawn appearance with moderate sketchiness.

### Example 2: Smooth Dotted Pattern
```javascript
frameConfig = {
  artisticEnabled: true,
  artisticRoughness: 0.5,
  artisticFillStyle: 'dots',
  artisticFillWeight: 1,
  artisticBowing: 0.5
}
```
Creates a subtle dotted pattern with minimal roughness.

### Example 3: Bold Cross-Hatch
```javascript
frameConfig = {
  artisticEnabled: true,
  artisticRoughness: 3.5,
  artisticFillStyle: 'cross-hatch',
  artisticFillWeight: 4,
  artisticBowing: 2
}
```
Creates a bold, heavily cross-hatched appearance with strong artistic effect.

## Technical Notes

### Performance
- Rough.js is highly optimized and adds minimal overhead
- Each shape is rendered individually with unique random variations
- Every render produces slightly different results (intentional artistic variation)

### Browser Compatibility
- Works in all modern browsers that support Canvas API
- No additional polyfills required

### Future Enhancements
Potential additions:
- SVG export for high-resolution proofs
- Additional fill patterns (stipple, sunburst, etc.)
- Per-shape-type artistic parameters
- Artistic rendering for outer borders
- Seed-based random generation for reproducible results

## File Changes

### Modified Files
1. `src/Designer.svelte`
   - Added artistic parameters to `frameConfig`
   - Added UI controls in Frame Controls section
   - Added CSS for magenta sliders and checkboxes

2. `src/GeometricFrame.svelte`
   - Imported `roughjs` library
   - Added artistic parameters to default config
   - Modified `drawInnerPattern()` to support artistic mode
   - Added artistic shape drawing functions:
     - `drawArtisticShape()`
     - `drawArtisticIsometricCube()`
     - `drawArtisticIsometricCylinder()`
     - `drawArtisticIsometricPyramid()`

### Dependencies
- Added: `roughjs` (npm package)

## Testing

To test the artistic rendering:

1. Start the dev server: `npm run dev`
2. Open the Designer UI
3. Navigate to **Geometric Frame → Inner Pattern**
4. Check **Enable Artistic Mode**
5. Adjust parameters:
   - Try different Fill Styles
   - Experiment with Roughness (0-5)
   - Adjust Bowing for organic curves
   - Change Fill Weight for pattern density
6. Observe how the inner pattern changes while the QR code remains unchanged

## Credits

- **Rough.js**: Created by [Preet Shihn](https://github.com/rough-stuff/rough)
- Implementation: Integrated into QR Logo Designer for artistic frame proofs
