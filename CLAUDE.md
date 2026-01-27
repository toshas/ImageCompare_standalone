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

`matchTuplesWithTrie(filesByModality, modalityNames)` uses a trie-based algorithm with LCS scoring:

1. **Reference modality**: Picks modality with most files as reference
2. **Trie construction**: Builds trie from reference filenames - each node tracks which files pass through it
3. **LCP matching**: For each file in other modalities, walks trie to find longest common prefix (LCP)
4. **LCS tie-breaking**: When multiple reference files share the same LCP, uses Longest Common Subsequence (LCS) to pick best match

Complexity: O(N × L) for trie operations, O(ties × L²) for LCS tie-breaking

This handles:
- Different naming conventions across modalities (e.g., `img_00001_gt.png` matches `img_00001_pred_v2.png`)
- Missing files in some modalities (gracefully creates partial tuples)
- Single-file modalities (matches to best LCP/LCS candidate)
- Identifiers embedded in middle of filenames (LCS catches common substrings)

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

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history.

### v0.1.1
- Replaced regex-based tuple matching with trie-based algorithm using LCP/LCS scoring

## Testing

Open in browser, drop test images or folder. No build step required.
