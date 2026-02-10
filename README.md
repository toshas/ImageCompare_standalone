# Image Compare

A standalone single-file HTML tool for comparing multiple images (modalities) with synchronized zoom and pan.

## Features

- **Multi-modality comparison**: Compare 2+ images with instant switching via keyboard or clicks
- **Multi-tuple mode**: Load a folder with subdirectories (each subdirectory = one modality) to browse multiple image sets
- **Smart matching**: Two-pass trie-based algorithm matches images across modalities by filename, even with different suffixes
- **Synchronized viewing**: Zoom and pan are preserved when switching between modalities
- **Thumbnail carousel**: Visual navigation with placeholder support for missing images
- **Spacebar flip**: Hold spacebar to temporarily view previous modality (release to flip back)
- **Crop tool**: Draw a rectangle to crop all modalities at the same coordinates, saved as `_cropNN.png` files
- **Winner voting**: Mark the best modality per tuple, persisted to `results.txt`
- **PowerPoint export**: Export voted tuples to `.pptx` with caption bars and crop callout overlays
- **Live file polling**: Automatically detects new files, deletions, and renames every 2 seconds
- **PPMX support**: Custom float32 grayscale format used in some imaging workflows
- **Fully offline**: Zero external dependencies for core functionality (PPTX export lazy-loads pptxgenjs from CDN)

## Usage

1. Open `image_compare.html` in a modern browser (Chrome recommended)
2. Either:
   - **Drop 2+ images** directly to compare them as a single tuple
   - **Drop a folder** containing subdirectories (each subdirectory becomes a modality)
   - **Click "Select Folder"** to use the file picker

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `← →` | Switch modality |
| `↑ ↓` | Previous/next tuple |
| `Space` | Flip to previous modality (hold) |
| `1-9` | Jump to modality N |
| `[ ]` | Reorder current modality left/right |
| `Enter` | Toggle winner for current modality |
| `Scroll` | Zoom in/out |
| `Drag` | Pan image |
| `C` | Toggle crop mode |
| `Del` | Delete current tuple files |
| `Esc` | Reset zoom (press twice to return to start screen) |

## Folder Structure for Multi-Tuple Mode

```
my_images/
├── modality_A/
│   ├── 001_something.png
│   ├── 002_something.png
│   └── 003_something.png
├── modality_B/
│   ├── 001_other.png
│   ├── 002_other.png
│   └── 003_other.png
└── modality_C/
    ├── 001_another.png
    └── 003_another.png  # 002 missing - will show placeholder
```

Images are matched across modalities using a two-pass algorithm (exact basenames first, then fuzzy trie-based matching). Missing images are handled gracefully with placeholder thumbnails.

## Browser Compatibility

- Chrome/Chromium (recommended — required for File System Access API features: crop, delete, voting, PPTX export)
- Firefox
- Safari
- Edge

## License

MIT
