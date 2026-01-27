# Changelog

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
