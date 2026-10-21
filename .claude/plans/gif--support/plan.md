# Animated GIF resize support (#30)

## Problem
`ResizedImageFieldFile.save` resized and saved only the current Pillow frame, so an animated GIF
came out as a single still image.

## Plan
1. Move the crop / thumbnail / scale block of `save()` into `ResizedImageFieldFile.resize(img, resample)`.
2. When the output format is GIF and the source `is_animated`, iterate `ImageSequence.Iterator`,
   resize each frame (converted to RGBA, i.e. the fully composited visible frame), and save with
   `save_all=True`, per-frame `duration`, the source `loop` (only if present) and `disposal=2`.
3. Every other image keeps the existing single-frame path unchanged.
4. Tests: resize, crop, and a GIF without `loop` (plays once). Changelog entry.

## Decisions
- **Fixture**: the reference GIF (Wikimedia "18-bit RGB Cube", 400x400, 180 frames) is CC BY-SA 4.0
  and 4.3 MB. Options: generate a GIF inside the test / commit a reduced derivative with attribution /
  commit the original. Chosen: generate it inside the test. The cube was used only for manual verification
  (180 frames, 40 ms and loop preserved at 350x350, 300x300 and 40x40 crop).
- **Metadata on the animated path**: `img.info` is not forwarded, because `transparency` and `background`
  are palette indices that become wrong after the RGBA frames are re-quantized; only `loop` is kept.
- **Implementation inline** rather than via a subagent: ~30 lines in one file, context already loaded.

## Out of scope
- Animated WebP / APNG input (still saved as the first frame only).
- `force_format` from an animated GIF to another format (first frame, as before).
- Pre-existing flake8 errors in `convert_mode_for_format` (E302, E231).
