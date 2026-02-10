# Changelog

## v0.2.1

- **Crop scales to each modality's resolution**: Crop rect is converted to relative coordinates (0–1) then scaled per-modality
  - Fixes incorrect crops when modalities have different image sizes (e.g., 4K vs 1080p)
- **Thumbnail cache fix**: Switched from `WeakMap<File>` to `Map` keyed by `modality/filename` string
  - Poller re-reads directories creating new File objects, causing WeakMap cache misses and full thumbnail regeneration
  - String-keyed Map survives poller cycles, so only genuinely new files regenerate thumbnails
- **Crop saves all modalities**: `saveCropFiles()` now loads images on-demand for any modality not currently in `images[]`
  - Snapshots `images` and `modalities` at start to avoid races with the poller
  - Prevents silent skip of modalities whose images were cleared by a concurrent poll
- **PPTX smart parent/crop logic**: When parent and crop are both voted, parent shows as simple full-image slide (voted crops get their own slides). When only parent is voted with exactly one crop, presents as if the crop was voted (crop slide with callout)
- **PPTX non-overlapping crop layout**: Crop centerpiece and callout thumbnail no longer overlap — main image shifts left, thumbnail shrinks if needed, accepts overlap only for near-16:9 crops
- **PPTX crop centerpiece anchored to bottom**: When downsized to avoid overlap, the crop image stays flush with the bottom edge

## v0.2.0

- **PowerPoint export**: Export voted tuples to `.pptx` via the PPTX button in the Tools panel
  - Each modality gets its own slide with caption bar (tuple name + modality, winner highlighted green)
  - Crop tuples include a callout thumbnail with red rectangle showing crop region on the full image
  - Crop region coordinates read from PNG tEXt metadata embedded during crop
  - pptxgenjs library lazy-loaded from CDN on first use (requires internet for export only)
- **Crop metadata in PNG tEXt chunks**: Crop coordinates (`x,y,w,h,srcW,srcH`) are embedded in cropped PNG files
  - Enables PPTX callout overlays showing exact crop region
  - Uses standard PNG tEXt chunk injection (compatible with VSCode extension)
- **Robust tuple matching tie-breaking**: Crop references (`_cropNN`) are explicitly deprioritized in fuzzy matching
  - Prevents long modality names from incorrectly matching crop files instead of originals
- **Floating panel fixes**: Hidden viewport indicator before image load, canvas starts at 160x100 to prevent zero-height panel

## v0.1.8

- **Crop square shortcut**: Double-click a cardinal (N/S/E/W) resize handle to make the crop rectangle square
  - Adjusts only the clicked edge, keeping the opposite edge fixed
  - Clamps to image boundaries when a perfect square isn't possible

## v0.1.7

- **Two-pass tuple matching**: Exact basename matching (pass 1) before fuzzy trie matching (pass 2)
  - Correctly groups crop files that share identical basenames across modalities
  - Prevents crop files from stealing fuzzy matches from original tuples
- **Crop tool**: Draw a crop rectangle on the image and save cropped versions to all modalities
  - Press `C` or click Crop button to enter crop mode
  - Draw, resize (8 handles), and move the crop rectangle
  - Confirm with Enter, cancel with Escape
  - Saves as `{tuple_name}_cropNN.png` via File System Access API (Chrome/Edge)
- **Delete button**: Remove all files for the current tuple from disk
  - Press `Delete`/`Backspace` or click Delete button
  - Confirmation dialog before deletion
- **File polling**: Automatically detects external file changes every 2 seconds
  - Re-reads modality directories and re-runs tuple matching
  - Detects new files (e.g., crop results), deletions, and renames
  - Only active in directory mode with File System Access API
- **Debug logging**: Set `DEBUG = true` for console diagnostics

## v0.1.1

- **Tuple matching**: Replaced regex-based `extractMatchingKey()` with trie-based `matchTuplesWithTrie()` algorithm
  - Uses longest common prefix (LCP) for efficient matching via trie
  - Falls back to longest common subsequence (LCS) for tie-breaking
- **Winner voting**: Declare a winner for each tuple in directory mode (Chrome/Edge only)
  - Press Enter or click the indicator on thumbnails to toggle winner
  - Winners are persisted to `results.txt` in the dropped folder
  - Win counts shown in parentheses after modality names in status bar
  - Human-readable and editable results file format

## v0.1.0

- Initial release
- Multi-modality image comparison with synchronized zoom/pan
- Multi-tuple mode with folder structure support
- Thumbnail carousel with placeholder support for missing modalities
- Spacebar flip behavior
- Modality reordering with `[ ]` keys
- PPMX float32 grayscale format support
