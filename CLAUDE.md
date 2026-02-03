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

### Tuple Matching (Two-Pass)

`matchTuplesWithTrie(filesByModality, modalityNames)` uses a two-pass trie-based algorithm:

1. **Reference modality**: Picks modality with most files as reference
2. **Trie construction**: Builds trie from reference filenames
3. **Pass 1 - Exact matching**: Files with identical basenames across modalities are matched first (handles crop files like `img_00079_crop01.png`)
4. **Pass 2 - Fuzzy trie matching**: Remaining files use LCP walk + LCS tie-breaking, excluding refs already claimed by exact matches

Complexity: O(N × L) for trie operations, O(ties × L²) for LCS tie-breaking

This handles:
- Crop files with identical names across modalities (exact match in pass 1)
- Different naming conventions across modalities (fuzzy match in pass 2)
- Missing files in some modalities (gracefully creates partial tuples)
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

### Crop Tool

Press `C` or click Crop button to enter crop mode. Draw a rectangle on the image, resize with handles, confirm with Enter. Crops are saved as `{tuple_name}_cropNN.png` to each modality directory via File System Access API. The file poller detects new crop files and adds them as a new tuple.

### Delete

Press `Delete`/`Backspace` or click Delete button to remove all files for the current tuple. Uses File System Access API to delete from disk.

### File Polling

Every 2 seconds, all modality directories are re-read via File System Access API. If the file listing changed (new files, deletions, renames), tuple matching is re-run and the UI updates. Only active in directory mode (when `votingEnabled` is true).

### Debug Logging

Set `const DEBUG = true;` near the top of the script to enable console diagnostics. Logs: poll results, crop saves, tuple deletions, matching stats.

## Version

Version constant `VERSION` at top of script. Displayed in dropzone header and help modal.

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history.

### v0.1.7
- Two-pass tuple matching (exact + fuzzy) for correct crop file grouping
- Crop tool (C key) with resize handles, saves to all modality directories
- Delete button (Del key) removes current tuple files from disk
- File polling (2s interval) detects external changes via File System Access API
- Debug logging flag (`DEBUG`) for console diagnostics

### v0.1.1
- Replaced regex-based tuple matching with trie-based algorithm using LCP/LCS scoring

## Testing

Open in browser, drop test images or folder. No build step required.
