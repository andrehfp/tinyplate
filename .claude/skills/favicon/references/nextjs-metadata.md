# Next.js Favicon Metadata Reference

## File-Based Convention (Recommended for App Router)

Next.js automatically handles favicons when you place them in the `app/` directory:

```
app/
├── favicon.ico          # Required: shown in browser tab
├── icon.png             # Optional: higher quality icon
├── icon.svg             # Optional: scalable vector icon
├── apple-icon.png       # Required for iOS: 180x180
└── opengraph-image.png  # Social sharing image: 1200x630
```

### Supported File Types

| File | Formats | Output |
|------|---------|--------|
| `favicon` | `.ico` | `<link rel="icon">` |
| `icon` | `.ico`, `.jpg`, `.png`, `.svg` | `<link rel="icon">` |
| `apple-icon` | `.jpg`, `.png` | `<link rel="apple-touch-icon">` |

### Generated Sizes

When using `icon.png`, Next.js generates:
- 16x16
- 32x32
- 48x48

For Apple icons, provide 180x180 PNG.

## Programmatic Icons

### Static Export

```typescript
// app/icon.tsx
import { ImageResponse } from "next/og";

export const runtime = "edge";
export const size = { width: 32, height: 32 };
export const contentType = "image/png";

export default function Icon() {
  return new ImageResponse(
    (
      <div
        style={{
          fontSize: 24,
          width: "100%",
          height: "100%",
          display: "flex",
          alignItems: "center",
          justifyContent: "center",
          background: "#000",
          color: "#fff",
          borderRadius: 6,
        }}
      >
        A
      </div>
    ),
    { ...size }
  );
}
```

### Dynamic Icon with Props

```typescript
// app/icon.tsx
export function generateImageMetadata() {
  return [
    {
      contentType: "image/png",
      size: { width: 48, height: 48 },
      id: "small",
    },
    {
      contentType: "image/png",
      size: { width: 72, height: 72 },
      id: "medium",
    },
  ];
}

export default function Icon({ id }: { id: string }) {
  // Return different icons based on id
}
```

## Metadata API

### Full Configuration

```typescript
// app/layout.tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
  icons: {
    // Favicon
    icon: [
      { url: "/favicon.ico", sizes: "any" },
      { url: "/icon.svg", type: "image/svg+xml" },
      { url: "/favicon-16x16.png", sizes: "16x16", type: "image/png" },
      { url: "/favicon-32x32.png", sizes: "32x32", type: "image/png" },
    ],
    // Apple
    apple: [
      { url: "/apple-touch-icon.png", sizes: "180x180", type: "image/png" },
    ],
    // Other
    other: [
      {
        rel: "mask-icon",
        url: "/safari-pinned-tab.svg",
        color: "#5bbad5",
      },
    ],
  },
  manifest: "/site.webmanifest",
};
```

### Output HTML

```html
<link rel="icon" href="/favicon.ico" sizes="any" />
<link rel="icon" href="/icon.svg" type="image/svg+xml" />
<link rel="icon" href="/favicon-16x16.png" sizes="16x16" type="image/png" />
<link rel="icon" href="/favicon-32x32.png" sizes="32x32" type="image/png" />
<link rel="apple-touch-icon" href="/apple-touch-icon.png" sizes="180x180" type="image/png" />
<link rel="mask-icon" href="/safari-pinned-tab.svg" color="#5bbad5" />
<link rel="manifest" href="/site.webmanifest" />
```

## Dark Mode Support

SVG favicons can adapt to dark mode:

```svg
<!-- public/icon.svg -->
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 32 32">
  <style>
    rect { fill: #000; }
    @media (prefers-color-scheme: dark) {
      rect { fill: #fff; }
    }
  </style>
  <rect width="32" height="32" rx="4"/>
</svg>
```

Or programmatically:

```typescript
// app/icon.tsx
export default function Icon() {
  return new ImageResponse(
    (
      <div
        style={{
          background: "black",
          // CSS-in-JS doesn't support @media queries
          // Use separate icon files for dark mode
        }}
      >
        ...
      </div>
    )
  );
}
```

## Per-Route Icons

Override icons for specific routes:

```typescript
// app/dashboard/icon.tsx
// Different icon for /dashboard routes
```

## Cache Busting

Add version to force refresh:

```typescript
export const metadata: Metadata = {
  icons: {
    icon: "/favicon.ico?v=2",
  },
};
```

Or use content hash in build:

```typescript
// next.config.js
module.exports = {
  generateBuildId: async () => {
    return "build-" + Date.now();
  },
};
```
