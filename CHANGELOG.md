# Changelog

## v0.1.1

- **Tuple matching**: Replaced regex-based `extractMatchingKey()` with trie-based `matchTuplesWithTrie()` algorithm
  - Uses longest common prefix (LCP) for efficient matching via trie
  - Falls back to longest common subsequence (LCS) for tie-breaking

## v0.1.0

- Initial release
- Multi-modality image comparison with synchronized zoom/pan
- Multi-tuple mode with folder structure support
- Thumbnail carousel with placeholder support for missing modalities
- Spacebar flip behavior
- Modality reordering with `[ ]` keys
- PPMX float32 grayscale format support
