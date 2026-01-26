# CLAUDE.md

## Project Overview

Single-file HTML image comparison tool. All code (HTML, CSS, JavaScript) is in `image_compare.html`.

## Architecture

### Key Data Structures

- `modalities`: Array of modality names (e.g., `["modA", "modB", "modC"]`)
- `images`: Sparse array indexed by modality position. `images[i]` is `{img, name, width, height, url, modality}` or `undefined` if missing
- `tuples`: Array of `{name, imageData}` where `imageData` contains `{file, modality}` objects
- `currentModalityIndex`: Position in `modalities` array (not images array)
- `currentTupleIndex`: Current tuple being viewed

### Navigation Model

Navigation uses modality position, not image index. This allows navigating to "slots" where an image is missing, showing a placeholder. The `previousModalityIndex` enables spacebar flip behavior.

### Tuple Matching

`extractMatchingKey(filename)` extracts a matching key from filenames using regex:
1. Leading numeric: `202505210021_suffix.png` → `202505210021`
2. Numeric with underscores: `250928_01_suffix.png` → `250928_01`
3. Alphanumeric: `test1_suffix.png` → `test1`
4. Fallback: everything before first underscore

### Thumbnail Generation

Uses browser-native APIs (no Sharp dependency):
- `createImageBitmap()` for off-thread decoding
- `OffscreenCanvas` when available for off-thread rendering
- Fallback to regular canvas for older browsers
- `WeakMap` cache keyed by File objects

## Important Patterns

### Modality Reordering

When reordering modalities with `[ ]` keys, swap in three arrays:
- `images`
- `modalities`
- `modalityColors`

Do NOT swap in `tuple.imageData` - it uses modality names for lookup, not array indices.

### Missing Modalities

- Carousel shows placeholder `div` with `✕` symbol
- Main viewer shows gray canvas with "Image not available"
- Modality buttons get `.unavailable` class (strikethrough)
- All placeholders are clickable/navigable

## Version

Version constant `VERSION` at top of script. Displayed in dropzone header and help modal.

## Testing

Open in browser, drop test images or folder. No build step required.
