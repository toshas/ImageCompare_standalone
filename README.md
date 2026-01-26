# Image Compare

A standalone HTML tool for comparing multiple images (modalities) side-by-side with synchronized zoom and pan.

## Features

- **Multi-modality comparison**: Compare 2+ images with instant switching via keyboard or clicks
- **Multi-tuple mode**: Load a folder with subdirectories (each subdirectory = one modality) to browse multiple image sets
- **Prefix-based matching**: Automatically matches images across modalities by filename prefix (e.g., `001_modA.png` matches `001_modB.png`)
- **Synchronized viewing**: Zoom and pan are preserved when switching between modalities
- **Thumbnail carousel**: Visual navigation for multi-tuple mode with placeholder support for missing images
- **Spacebar flip**: Hold spacebar to temporarily view previous modality (release to flip back)
- **PPMX support**: Custom float32 grayscale format used in some imaging workflows

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
| `Scroll` | Zoom in/out |
| `Drag` | Pan image |
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

Images are matched by their leading numeric or alphanumeric prefix. Missing images in some modalities are handled gracefully with placeholder thumbnails.

## Browser Compatibility

- Chrome/Chromium (recommended)
- Firefox
- Safari
- Edge

Requires modern browser features: `createImageBitmap`, `OffscreenCanvas` (optional), File System Access API (for folder drops).

## License

MIT
