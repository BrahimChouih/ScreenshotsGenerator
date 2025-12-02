# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

App Store Screenshot Generator - a browser-based tool for creating App Store marketing screenshots. Built with vanilla JavaScript, HTML5 Canvas, Three.js, and CSS. No build process required.

## Development

To run locally, serve via a web server (required for IndexedDB persistence):

```bash
python3 -m http.server 8000
# or
npx serve .
```

Open `http://localhost:8000` in browser. Opening `index.html` directly from filesystem will break persistence.

## Architecture

**Main files:**

- `index.html` - UI structure with modals for settings, about, project management, translations, and language selection
- `styles.css` - Dark theme styling, responsive layout with CSS Grid (3-column: left sidebar, canvas, right sidebar)
- `app.js` - All application logic (~4200 lines)
- `three-renderer.js` - Three.js 3D rendering for iPhone mockups (~680 lines)

**Key patterns in app.js:**

- `state` object at top holds all application state (screenshots, settings, text, background config)
- `updateCanvas()` is the main render function - call after any state change
- `saveState()` persists to IndexedDB, called automatically in `updateCanvas()`
- `syncUIWithState()` updates all UI controls to reflect current state
- Project management uses IndexedDB with two stores: `projects` (data) and `meta` (project list)
- Per-screenshot settings: each screenshot stores its own background, device, and text settings

**Canvas rendering pipeline (in updateCanvas):**
1. `drawBackground()` - gradient/solid/image with optional blur and overlay
2. `drawScreenshot()` - positioned, scaled, rotated screenshot with shadow and border
3. `drawText()` - headline and subheadline with multi-language support
4. `drawNoise()` - optional noise texture overlay

**3D rendering (in three-renderer.js):**
- Uses Three.js with GLTFLoader for iPhone 15 Pro Max model
- `initThreeJS()` - initializes scene, camera, renderer, and lights
- `loadPhoneModel()` - loads the 3D iPhone model from `models/iphone-15-pro-max.glb`
- `renderThreeJSToCanvas()` - renders 3D scene to export canvas with full resolution
- `renderThreeJSForScreenshot()` - renders 3D for specific screenshot (side previews)
- Drag-to-rotate interaction on the 3D preview canvas

**Multi-language text:**
- `state.text.headlines` and `state.text.subheadlines` are objects keyed by language code
- `getTextSettings()` returns either global or per-screenshot text depending on toggle state
- AI translation calls Claude/OpenAI/Google API directly from browser (requires API key in settings)

**UI Components:**
- Right sidebar has three tabs: Background, Device, Text
- Collapsible toggle sections for Noise, Shadow, Border, Headline, Subheadline
- Device tab has 2D/3D mode selector with different controls for each mode
- Side preview carousel with sliding animation between screenshots
- Project selector dropdown with screenshot counts
- Gradient editor with draggable color stops

## Key Functions

- `createProject()` / `deleteProject()` / `switchProject()` - async, must await and call `updateProjectSelector()` after
- `handleFiles()` - processes uploaded images into screenshot array
- `exportCurrent()` / `exportAll()` - generates PNG downloads from canvas (ZIP for batch export)
- `applyPositionPreset()` - applies preset screenshot positioning (centered, bleed, tilt, perspective, etc.)
- `transferStyle()` - copies style settings from one screenshot to another
- `slideToScreenshot()` - animates carousel transition between screenshots
- `updateSidePreviews()` - renders adjacent screenshots in side preview canvases

## External Dependencies

- **Three.js** (r128) - 3D rendering for device mockups
- **GLTFLoader** - loads iPhone 3D model
- **JSZip** - creates ZIP files for batch export
- **Google Fonts API** - font picker with 1500+ fonts
