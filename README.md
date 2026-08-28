# TapBox

A self-contained, browser-based bounding-box annotation tool for building object-detection datasets — built to be used with your thumb, on your phone.

## Overview

TapBox is a single-page web app for drawing bounding-box annotations on images and exporting them in YOLO label format. It's designed first for **Android phones and touchscreens**: large touch targets, finger-drawn boxes, and a layout that works in a browser tab rather than requiring an installed app.

It runs entirely **client-side** — there is no backend, no image upload, and no server component required for annotation, navigation, or export. The only network dependency is a CDN-hosted JavaScript library (JSZip) used to build the exported `.zip` file; see [Privacy](#privacy) and [Credits](#credits--dependencies) below for exactly what that means.

## Features

Everything below reflects what the current implementation actually does:

- **Mobile-first UI** — large buttons, touch-friendly controls, safe-area padding for notches/home indicators
- **Android Chrome support**, built on standard Pointer Events (`pointerdown` / `pointermove` / `pointerup`), so it also works with mouse input on desktop
- **Multiple bounding boxes per image** — draw any number of boxes on a single image
- **Tap-to-select** — tap an existing box to select it before moving or resizing it, so drawing a new box never accidentally drags an old one
- **Move and resize** selected boxes via corner-handle dragging
- **Delete Selected** (removes just the selected box) and **Clear All** (removes every box on the current image)
- **Undo** — steps back through box create/move/resize/delete/clear actions, per image
- **Multi-image import** — select many images at once from your device
- **Previous / Next navigation** with an image counter (`23 / 500`) and per-image annotation status (`✓ Annotated`, `○ Not annotated`, `No object`)
- **Objects counter** showing how many boxes are on the current image
- **"No Object" / negative sample marking**, producing an empty label file for images with no target object
- **In-memory annotation persistence while the app is open** — navigate back and forth between images without losing boxes (see [important limitation](#privacy) below)
- **YOLO-format label export**, one line per bounding box
- **ZIP dataset export** (images, labels, and a `classes.txt`) via JSZip
- **Original image files are preserved byte-for-byte** in the export — images are never re-encoded or re-compressed
- **Coordinates normalized to the original image dimensions**, not the on-screen scaled size (see [Coordinate System](#coordinate-system))
- **Optional keyboard shortcuts** for desktop use: `←`/`P` previous, `→`/`N` next, `S` save, `Delete`/`Backspace` delete the selected box
- Overall dataset progress indicator (`Annotated 18 / 500`)

TapBox currently supports a single object class (`object`, class ID `0`). Multi-class support is a possible future addition — see [Roadmap](#roadmap).

## Screenshots

![TapBox](screenshots/tapbox.png)

## How to use

1. Open `index.html` in a browser (see [Running locally](#running-locally)).
2. Tap **Import** and select one or more images from your device.
3. Draw a bounding box by dragging your finger over the target object. Draw as many boxes as the image needs.
4. Tap an existing box to select it, then drag inside it to move it, or drag a corner handle to resize it.
5. If an image has no target object, tap **No Object** instead of drawing a box.
6. Tap **Save** to confirm the current image's annotation, then **Next** to move on. (Boxes are also kept in memory automatically as you draw — Save is your explicit confirmation.)
7. Once you've gone through your images, tap **Export Dataset (.zip)** to download the finished dataset.

### On Android

Open `index.html` in **Chrome for Android** (directly from storage, or via a hosted URL such as GitHub Pages — see below). The interface is sized for a phone screen and works with touch alone; no mouse or keyboard is required. Rotate the device freely — the canvas re-fits the image on orientation change.

## YOLO format

Each image gets a corresponding `.txt` label file. Every bounding box on that image produces one line:

```
class_id x_center y_center width height
```

All four numeric values are normalized to a `0.0`–`1.0` range relative to the image's width and height. For example:

```
0 0.523438 0.476563 0.312500 0.421875
```

- `class_id` — always `0` in the current single-class build (`object`)
- `x_center`, `y_center` — the center of the box, as a fraction of image width/height
- `width`, `height` — the box's size, as a fraction of image width/height

An image with multiple boxes gets multiple lines, one per box, in the order they were drawn. An image marked "No Object" (or with no saved boxes) gets an **empty** `.txt` file.

## Dataset structure

Exporting produces a `dataset.zip` with this layout:

```
dataset/
├── images/
│   ├── img_00001.jpg
│   ├── img_00002.jpg
│   └── ...
├── labels/
│   ├── img_00001.txt
│   ├── img_00002.txt
│   └── ...
└── classes.txt
```

Images are renamed sequentially (`img_00001`, `img_00002`, ...) in import order, keeping their original file extension and original bytes unmodified. Each label file shares the same base name as its image (`img_00001.jpg` ↔ `img_00001.txt`), so the two directories line up index-for-index. `classes.txt` lists the class names by line, matching YOLO's `class_id` indexing (line 1 = class `0`).

## Coordinate system

Images are scaled to fit the viewer on screen — the displayed size depends on your phone's screen size, orientation, and pixel ratio, none of which match the image's actual resolution. TapBox reads the image's true dimensions (`image.naturalWidth` / `image.naturalHeight`) and converts every touch coordinate through a fit-transform (scale + centering offset) back into the image's original pixel space before storing or exporting it. Annotation math is never done in on-screen/CSS pixel space — only in original image pixels, which are then normalized to `0–1` for the YOLO output. This keeps annotations correct regardless of screen size, orientation, or how the image was scaled for display.

## Running locally

TapBox is a single HTML file with inline CSS and JavaScript — there's no build step, no `npm install`, and no server required for annotation and export.

- **Simplest**: open `index.html` directly in a browser (double-click it, or use "Open with" on a phone).
- **Alternative**: serve it from any static file server or GitHub Pages if you prefer a URL over a local file.

**Browser note:** the app needs network access the first time it loads (in that browser/session) to fetch the JSZip library from a CDN, which is required for the **Export** step. Import, drawing, navigation, and undo do not require network access. If your browser has already cached the JSZip script, export may also work without a live connection — but this isn't guaranteed.

## GitHub Pages

You can use the hosted version of **TapBox** directly without downloading or installing anything:

**https://halloweeks.github.io/tapbox/**

Open the link in **Chrome on Android** to start labeling images.

Your images and annotations are processed locally in your browser. **TapBox does not upload your imported images to the GitHub repository or to a TapBox server.**

## Privacy

Images you import are read and processed entirely in your browser (via `<input type="file">`, `FileReader`/`URL.createObjectURL`, and `<canvas>`). **TapBox itself never uploads, transmits, or sends your images or annotations to any server.** All annotation data lives only in the page's JavaScript memory for the current browser session.

Two things worth knowing:

- **Nothing is saved automatically.** Reloading the page, closing the tab, or navigating away discards all unsaved annotations. Export your dataset before you close the tab.
- **JSZip is loaded from a third-party CDN** (`cdnjs.cloudflare.com`) at page load, purely to build the exported `.zip` client-side. That single script request is the only network call the app makes; loading it is subject to that CDN's own network/logging behavior, outside TapBox's control. No image or annotation data is sent as part of that request.

## Browser compatibility

TapBox relies on standard, broadly-supported web APIs: Pointer Events, `<canvas>`, `FileReader`/`Blob`/`URL.createObjectURL`, and modern CSS (including `dvh` units and `env(safe-area-inset-*)`).

- **Chrome for Android** — primary target, fully supported
- **Chrome / Edge (desktop)** — fully supported, including mouse-based drawing and keyboard shortcuts
- **Firefox (desktop & Android)** — expected to work; uses the same standard APIs, but hasn't been exhaustively tested
- **Safari (iOS/macOS)** — expected to work on reasonably recent versions (Pointer Events and `dvh` require modern Safari), but hasn't been exhaustively tested

If you hit a browser-specific issue, please open an issue with your browser/OS version.

## License

[MIT](./LICENSE) — a simple, permissive license appropriate for a small open-source utility.

## Contributing

This is a small vanilla JavaScript project with no build step, so contributing is straightforward:

1. **Fork** the repository.
2. **Clone** your fork locally.
3. **Modify** `index.html` directly.
4. **Test** by opening `index.html` in a browser (ideally check both a desktop browser and an Android/mobile browser, since touch behavior is the app's core use case).
5. **Open a pull request** describing what changed and why.

Please keep changes dependency-free where possible, in keeping with the project's single-file, no-build-system approach.

## Roadmap

Realistic possible future improvements — nothing listed here is implemented yet:

- [ ] Multiple classes (currently single-class only)
- [ ] Additional export formats (e.g. COCO, Pascal VOC)
- [ ] Dataset train/validation splitting on export
- [ ] Optional local persistence (e.g. saving progress between sessions)
- [ ] Box copy/duplicate between similar frames

## Credits / dependencies

- **[JSZip](https://stuk.github.io/jszip/)** — used to build the exported `.zip` archive client-side. Loaded from a public CDN (`cdnjs.cloudflare.com`) at runtime; not bundled into this repository. JSZip is dual-licensed under MIT and GPLv3 — see the [JSZip repository](https://github.com/Stuk/jszip) for full license details.

## Disclaimer

TapBox is an **annotation utility only**. It helps you create labeled image datasets in YOLO format — it does not train, evaluate, or run any object-detection model itself.
