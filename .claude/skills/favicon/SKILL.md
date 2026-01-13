---
name: favicon
description: "Generate favicons and app icons for Next.js projects. Creates all required sizes, formats, and configures metadata."
allowed-tools: Read, Glob, Grep, Write, Edit, Bash, WebFetch
---

# Favicon Generator

Generate complete favicon sets for Next.js projects from a source image.

## Prerequisites

```bash
# Install sharp for image processing
bun add sharp
```

## Workflow

### Step 1: Get Source Image

Ask the user for:
1. **Source image path** - High-res PNG/SVG (512x512 or larger recommended)
2. **Background color** (optional) - For PWA splash screens

### Step 2: Generate All Favicon Sizes

Create a generation script:

```typescript
// scripts/generate-favicons.ts
import sharp from "sharp";
import { mkdir } from "fs/promises";
import { join } from "path";

const SOURCE = "source-icon.png"; // User's source image
const OUTPUT_DIR = "public";

const sizes = [
  // Standard favicons
  { name: "favicon-16x16.png", size: 16 },
  { name: "favicon-32x32.png", size: 32 },
  { name: "favicon.ico", size: 32 }, // ICO format

  // Apple touch icons
  { name: "apple-touch-icon.png", size: 180 },

  // Android/PWA icons
  { name: "android-chrome-192x192.png", size: 192 },
  { name: "android-chrome-512x512.png", size: 512 },

  // Microsoft tiles
  { name: "mstile-150x150.png", size: 150 },
];

async function generateFavicons() {
  for (const { name, size } of sizes) {
    const outputPath = join(OUTPUT_DIR, name);

    if (name.endsWith(".ico")) {
      // For ICO, create PNG first then convert
      await sharp(SOURCE)
        .resize(size, size)
        .png()
        .toFile(outputPath.replace(".ico", ".png"));
      console.log(`Generated: ${name} (as PNG, convert manually or use ico-convert)`);
    } else {
      await sharp(SOURCE)
        .resize(size, size)
        .png()
        .toFile(outputPath);
      console.log(`Generated: ${name}`);
    }
  }

  console.log("\nFavicons generated in /public");
}

generateFavicons();
```

Run with:
```bash
bun run scripts/generate-favicons.ts
```

### Step 3: Create Web Manifest

```json
// public/site.webmanifest
{
  "name": "App Name",
  "short_name": "App",
  "icons": [
    {
      "src": "/android-chrome-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/android-chrome-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ],
  "theme_color": "#ffffff",
  "background_color": "#ffffff",
  "display": "standalone"
}
```

### Step 4: Configure Next.js Metadata

For **App Router** (Next.js 13+):

```typescript
// app/layout.tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "App Name",
  description: "App description",
  icons: {
    icon: [
      { url: "/favicon-32x32.png", sizes: "32x32", type: "image/png" },
      { url: "/favicon-16x16.png", sizes: "16x16", type: "image/png" },
    ],
    apple: [
      { url: "/apple-touch-icon.png", sizes: "180x180", type: "image/png" },
    ],
    other: [
      { rel: "mask-icon", url: "/safari-pinned-tab.svg", color: "#5bbad5" },
    ],
  },
  manifest: "/site.webmanifest",
  other: {
    "msapplication-TileColor": "#da532c",
    "theme-color": "#ffffff",
  },
};
```

Or use the **file-based convention** (simpler):

```
app/
├── favicon.ico          # Browser tab icon
├── icon.png             # App icon (auto-generates sizes)
├── apple-icon.png       # Apple touch icon
└── manifest.ts          # Web manifest (or .json in /public)
```

### Step 5: Verify Setup

```bash
# Check all files exist
ls -la public/favicon* public/apple-touch-icon.png public/android-chrome* public/site.webmanifest

# Start dev server and check
bun run dev
# Visit http://localhost:3000 and inspect <head> tags
```

## Required Files Checklist

| File | Size | Purpose |
|------|------|---------|
| `favicon.ico` | 32x32 | Legacy browsers |
| `favicon-16x16.png` | 16x16 | Modern browsers (small) |
| `favicon-32x32.png` | 32x32 | Modern browsers |
| `apple-touch-icon.png` | 180x180 | iOS home screen |
| `android-chrome-192x192.png` | 192x192 | Android/PWA |
| `android-chrome-512x512.png` | 512x512 | PWA splash screen |
| `site.webmanifest` | - | PWA configuration |

## Alternative: Using ImageMagick

If sharp isn't available:

```bash
# Install ImageMagick
brew install imagemagick

# Generate all sizes
convert source.png -resize 16x16 public/favicon-16x16.png
convert source.png -resize 32x32 public/favicon-32x32.png
convert source.png -resize 180x180 public/apple-touch-icon.png
convert source.png -resize 192x192 public/android-chrome-192x192.png
convert source.png -resize 512x512 public/android-chrome-512x512.png

# Create ICO (multi-size)
convert source.png -resize 32x32 -define icon:auto-resize=32,16 public/favicon.ico
```

## Alternative: Online Tools

If you don't have the source image in the project:

1. **RealFaviconGenerator** - https://realfavicongenerator.net
   - Upload image, configure options, download package

2. **Favicon.io** - https://favicon.io
   - Text to favicon, image to favicon, emoji to favicon

## SVG Favicon (Modern Browsers)

For dynamic/dark mode support:

```typescript
// app/icon.tsx
import { ImageResponse } from "next/og";

export const runtime = "edge";
export const contentType = "image/png";
export const size = { width: 32, height: 32 };

export default function Icon() {
  return new ImageResponse(
    (
      <div
        style={{
          fontSize: 24,
          background: "linear-gradient(135deg, #667eea 0%, #764ba2 100%)",
          width: "100%",
          height: "100%",
          display: "flex",
          alignItems: "center",
          justifyContent: "center",
          borderRadius: 8,
          color: "white",
          fontWeight: "bold",
        }}
      >
        A
      </div>
    ),
    { ...size }
  );
}
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Favicon not updating | Clear browser cache, or add `?v=2` to URL |
| Apple icon not showing | Ensure exactly 180x180, PNG format |
| PWA install fails | Check manifest is valid JSON, icons accessible |
| ICO not working | Use multi-resolution ICO with 16x16 and 32x32 |

## Quick One-Liner

Generate all favicons with ImageMagick:

```bash
for size in 16 32 180 192 512; do convert source.png -resize ${size}x${size} public/favicon-${size}x${size}.png; done && convert source.png -resize 32x32 -define icon:auto-resize=32,16 public/favicon.ico && mv public/favicon-180x180.png public/apple-touch-icon.png && mv public/favicon-192x192.png public/android-chrome-192x192.png && mv public/favicon-512x512.png public/android-chrome-512x512.png
```
