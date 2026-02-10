# CLAUDE.md

## Project Overview

Single-file HTML image comparison tool. All code (HTML, CSS, JavaScript) is in `image_compare.html`.

Zero external dependencies for core functionality — works fully offline. The only external dependency is pptxgenjs, which is lazy-loaded from CDN only when the user clicks the PPTX export button.

## Architecture

### Key Data Structures

- `modalities`: Array of modality names (e.g., `["modA", "modB", "modC"]`)
- `images`: Sparse array indexed by modality position. `images[i]` is `{img, name, width, height, url, modality}` or `undefined` if missing
- `tuples`: Array of `{name, imageData}` where `imageData` contains `{file, modality}` objects
- `currentModalityIndex`: Position in `modalities` array (not images array)
- `currentTupleIndex`: Current tuple being viewed
- `winners`: Map of `tupleIndex → modalityIndex` for voted tuples
- `modalityDirHandles`: Map of `modalityName → FileSystemDirectoryHandle` (directory mode only)
- `rootDirHandle`: `FileSystemDirectoryHandle` for the root folder (used for `results.txt` I/O)

### Navigation Model

Navigation uses modality position, not image index. This allows navigating to "slots" where an image is missing, showing a placeholder. The `previousModalityIndex` enables spacebar flip behavior.

### Tuple Matching (Two-Pass)

`matchTuplesWithTrie(filesByModality, modalityNames)` uses a two-pass trie-based algorithm:

1. **Reference modality**: Picks modality with most files as reference
2. **Trie construction**: Builds trie from reference filenames
3. **Pass 1 - Exact matching**: Files with identical basenames across modalities are matched first (handles crop files like `img_00079_crop01.png`)
4. **Pass 2 - Fuzzy trie matching**: Remaining files use LCP walk + tie-breaking, excluding refs already claimed by exact matches

**Tie-breaking** (when multiple reference files share the same LCP):
- Prefer non-crop reference over crop reference (`_crop\d+$` pattern)
- Then prefer smaller length difference (`|refLen - queryLen|`)
- Then prefer higher LCS (Longest Common Subsequence)

Complexity: O(N × L) for trie operations, O(ties × L²) for LCS tie-breaking

This handles:
- Crop files with identical names across modalities (exact match in pass 1)
- Different naming conventions across modalities (fuzzy match in pass 2)
- Missing files in some modalities (gracefully creates partial tuples)
- Identifiers embedded in middle of filenames (LCS catches common substrings)
- **Crop files**: Original and cropped files coexist — crop references are explicitly deprioritized so they never steal matches from originals, regardless of query length

### Thumbnail Generation

Uses browser-native APIs (no Sharp dependency):
- `createImageBitmap()` for off-thread decoding
- `OffscreenCanvas` when available for off-thread rendering
- Fallback to regular canvas for older browsers
- `Map` cache keyed by `"modality/filename"` strings (survives poller re-reads; a `WeakMap<File>` fails because polling creates new File objects for the same files)

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

`saveCropFiles()` snapshots `images` and `modalities` at the start to avoid races with the poller. If any modality image is not loaded (e.g., poller cleared it), it loads it on-demand from `tuple.imageData` before cropping. The crop rect (drawn in the current modality's pixel space) is converted to relative coordinates (0–1), then scaled to each modality's actual resolution — this handles modalities with different image sizes (e.g., 4K vs 1080p).

Crop metadata (`x,y,w,h,srcW,srcH`) is embedded as a PNG tEXt chunk with keyword `ImageCompare:CropRect`. This enables PPTX export to show crop region callouts on the full image.

### Crop Metadata (PNG tEXt Chunks)

- **`pngCrc32(buf)`**: Pure-JS CRC-32 with lazy-initialized lookup table (browser has no `zlib`).
- **`pngInjectText(pngBytes, keyword, value)`**: Builds a PNG tEXt chunk (keyword + null + value + CRC32) and inserts before IEND. Scans for IEND chunk properly (not hardcoded offset).
- **`pngReadText(pngBytes, keyword)`**: Scans PNG chunk structure for matching tEXt keyword, returns value string or null.
- **`readCropMetadata(pngBytes)`**: Reads `ImageCompare:CropRect` tEXt chunk, returns `{x, y, w, h, srcW, srcH}` or null.

All functions operate on `Uint8Array` (browser-native, no Node.js `Buffer`).

**Cross-compatibility with VSCode extension**: Both tools write and read PNG tEXt chunks with keyword `ImageCompare:CropRect`. The VSCode extension additionally writes EXIF `ImageDescription` (Sharp path) but always includes a tEXt chunk too. Crops made in either tool are fully readable by the other.

### Delete

Press `Delete`/`Backspace` or click Delete button to remove all files for the current tuple. Uses File System Access API to delete from disk.

### PPTX Export

Click the PPTX button in the Tools panel to export voted tuples to PowerPoint.

- **Lazy loading**: pptxgenjs is loaded from CDN on first use (only external dependency, requires internet)
- **Layout**: 16:9 slides (10" × 5.625"), one slide per modality per voted tuple
- **Caption bar**: Semi-transparent gray bar at top with tuple name (left) and modality name (right, green for winner)
- **Simple slides**: Full image "contain"-fit when no crops exist, or when parent and crop are both voted (voted crops get their own slides)
- **Crop slides**: Cropped image fit to slide (bottom-anchored when shifted), bottom-right callout with full image + red rectangle overlay from PNG tEXt metadata
- **Smart parent/crop logic**: Voted crop tuples find their parent for the callout. When only parent is voted with one crop, auto-expands as crop slide. When parent and crop are both voted, parent becomes simple slide
- **Output**: Downloads as `comparison_XXXXXX.pptx` (timestamp-based naming)

### Winner Voting

Press Enter to mark current modality as winner for the current tuple. Winners are saved to `results.txt` via File System Access API. Only available in directory mode.

### File Polling

Every 2 seconds, all modality directories are re-read via File System Access API. If the file listing changed (new files, deletions, renames), tuple matching is re-run and the UI updates. Only active in directory mode (when `votingEnabled` is true).

### Floating Tools Panel

Draggable, collapsible panel in the top-right corner:
- **Minimap**: Thumbnail of current image with magenta viewport rectangle when zoomed
- **Crop**: Enter crop mode
- **Delete**: Delete current tuple files
- **PPTX**: Export voted tuples to PowerPoint

The minimap canvas starts at 160×100 to avoid size jumping. The viewport rectangle is hidden by default and only shown when zoom > 1.05.

### Debug Logging

Set `const DEBUG = true;` near the top of the script to enable console diagnostics. Logs: poll results, crop saves, tuple deletions, matching stats, PPTX export.

## Version

Version constant `VERSION` at top of script. Displayed in dropzone header and help modal.

When bumping the version, update all three files:
1. `image_compare.html` — `const VERSION = 'v0.X.Y';`
2. `CHANGELOG.md` — add new section at top
3. `CLAUDE.md` — add entry in the changelog section below

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history.

### v0.2.1
- Crop rect scaled per-modality via relative coordinates (fixes wrong region with different resolutions)
- Thumbnail cache: `WeakMap<File>` → `Map<string>` keyed by `modality/filename` (survives poller re-reads)
- Crop saves: snapshot images + on-demand loading for missing modalities (fixes race with poller)
- PPTX smart parent/crop voting logic (simple slide when crops also voted, auto-expand single unvoted crop)
- PPTX non-overlapping crop layout with bottom-anchored centerpiece

### v0.2.0
- PPTX export with pptxgenjs (lazy-loaded from CDN)
- Crop metadata embedded in PNG tEXt chunks
- Crop-deprioritized tuple matching tie-breaking
- Floating panel fixes (hidden viewport indicator, 160x100 initial canvas)

### v0.1.8
- Crop square shortcut: double-click cardinal resize handle to make crop square

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
