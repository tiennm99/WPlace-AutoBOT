# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Chromium browser extension for WPlace (wplace.live), a collaborative pixel art canvas website. The extension provides automation tools to farm pixels, place images, repair artwork, extract artwork, and manage multiple accounts.

Key features include:
- Auto Farm: Automatically farms pixels with configurable charge thresholds
- Auto Image: Places images on the canvas with overlay support
- Auto Repair: Automatically repairs existing artwork
- Art Extractor: Extracts artwork areas and saves them as JSON
- Multi-Account Manager: Switch between and manage multiple WPlace accounts
- Script Manager: Neon-themed interface to launch automation scripts

## Repository Structure

```
Extension/
├── background.js              # Background service worker (cookie/account management, script execution)
├── content-script.js          # Content script (injects UI button into wplace.live)
├── Script-manager.js          # Main script launcher (neon-themed UI)
├── manifest.json              # Extension manifest
├── icons/                     # Extension icons
├── lang/                      # Language translation files (JSON)
├── themes/                    # CSS theme files
├── popup/                     # Extension popup UI
│   ├── popup.html             # Popup interface
│   ├── popup.js               # Popup logic (account management)
│   └── popup.css              # Popup styling
└── scripts/                   # Automation scripts
    ├── Auto-Farm.js           # Pixel farming automation
    ├── Auto-Image.js          # Image placement automation
    ├── Auto-Repair.js         # Artwork repair automation
    ├── Art-Extractor.js       # Artwork extraction tool
    ├── Acc-Switch.js          # Account switching utilities
    ├── image-processor.js     # Image processing utilities
    ├── overlay-manager.js     # Canvas overlay management
    ├── token-manager.js       # Turnstile token management
    └── utils-manager.js       # Shared utility functions
```

## Architecture Overview

### Extension Components

1. **Manifest V3 Extension**: Uses modern Chrome extension architecture
2. **Background Service Worker**: Handles account management, cookie handling, and script execution
3. **Content Script**: Injects a button into wplace.live for launching automation scripts
4. **Popup UI**: Provides account management and script selection interface
5. **Script Manager**: Neon-themed launcher for all automation scripts
6. **Individual Scripts**: Separate modules for different automation tasks

### Key Architectural Patterns

1. **Message Passing**: Content script ↔ Background script ↔ Page context
2. **Resource Loading**: Background script pre-loads themes and language files, injects them into scripts
3. **MAIN World Execution**: Scripts run in MAIN world context to interact with wplace.live APIs
4. **Dependency Injection**: Complex scripts inject utility managers (image-processor, overlay-manager, etc.)
5. **Global Managers**: Singleton utility managers available as global objects (window.globalUtilsManager, etc.)

### Data Flow

1. User clicks extension button on wplace.live
2. Content script sends message to Background script
3. Background script:
   - Loads script dependencies (managers, themes, languages)
   - Fetches script content
   - Injects resources and executes in MAIN world
4. Script runs in page context with full access to wplace.live APIs
5. UI elements (panels, overlays) are injected into the page DOM

### Account Management

Accounts are stored in chrome.storage.local with:
- `accounts`: Array of authentication tokens
- `infoAccounts`: Detailed account information (name, charges, droplets, etc.)
- Background script handles cookie setting/getting for account switching

## Development Guidelines

### No Build Process Required

This is a browser extension with no build process. Development happens directly with the source files:

1. Make code changes
2. Reload extension in Chrome (chrome://extensions/)
3. Test on wplace.live

### File Modification Guidelines

1. **Never modify** `manifest.json` unless adding new permissions or resources
2. **Prefer editing** existing language JSON files over adding new ones
3. **Script files** can be modified directly - they're injected as-is
4. **CSS files** are loaded and injected by the background script

### Extension Loading for Development

1. Open Chrome/Edge and navigate to `chrome://extensions/`
2. Enable "Developer mode"
3. Click "Load unpacked"
4. Select the `Extension` directory
5. Test on https://wplace.live

### Testing Changes

1. Make changes to source files
2. Reload the extension in Chrome extensions page
3. Refresh wplace.live
4. Click the extension button to test functionality

### Common Development Tasks

#### Adding a New Script
1. Create script file in `Extension/scripts/`
2. Add entry to `AVAILABLE_SCRIPTS` in:
   - `Extension/Script-manager.js`
   - `Extension/popup/popup.js`
3. Update manifest if new permissions needed

#### Adding a New Language
1. Create JSON file in `Extension/lang/`
2. Add to language loading in `Extension/background.js`
3. No code changes needed - automatically loaded

#### Adding New UI Elements
1. CSS is automatically loaded from `Extension/themes/`
2. For popup changes, modify files in `Extension/popup/`
3. For in-page UI, modify individual script files

## Key APIs and Integration Points

### WPlace Integration
- Canvas interaction through simulated mouse events
- API calls to `https://backend.wplace.live/` for account data
- DOM manipulation to add UI elements

### Chrome Extension APIs
- `chrome.storage.local` for account data persistence
- `chrome.cookies` for authentication token management
- `chrome.scripting` for injecting scripts into MAIN world
- `chrome.runtime` for message passing

### Global Utility Managers
- `window.globalUtilsManager` - Shared utility functions
- `window.globalImageProcessor` - Image processing functions
- `window.globalOverlayManager` - Canvas overlay functions
- `window.globalTokenManager` - Turnstile token functions

## Debugging

- All scripts log to browser console with colored output
- Extension background console available at `chrome://extensions/`
- Console logs include detailed execution information
- Use browser dev tools to inspect injected UI elements

## Multi-Language Support

The extension supports multiple languages through JSON files in `Extension/lang/`. Text is automatically translated based on user's browser language. Translation functions are available through `window.getLanguage()` and `t()` functions in utils manager.

## Themes

Visual styling is handled through CSS files in `Extension/themes/`. The neon theme is the primary theme used throughout the extension. Themes are loaded by the background script and injected into pages.