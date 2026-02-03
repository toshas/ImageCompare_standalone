# Changelog

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
