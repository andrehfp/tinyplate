# Next.js SEO Implementation Templates

## Complete App Router SEO Setup

### 1. File Structure

```
app/
├── layout.tsx          # Root metadata
├── page.tsx            # Homepage
├── robots.ts           # /robots.txt
├── sitemap.ts          # /sitemap.xml
├── icon.tsx            # Dynamic favicon (optional)
├── apple-icon.tsx      # Apple touch icon (optional)
├── opengraph-image.tsx # Dynamic OG image (optional)
├── manifest.ts         # PWA manifest (optional)
└── [route]/
    └── page.tsx        # Page-specific metadata
```

### 2. Dynamic Favicon (app/icon.tsx)

```typescript
import { ImageResponse } from "next/og";

export const size = { width: 32, height: 32 };
export const contentType = "image/png";

export default function Icon() {
  return new ImageResponse(
    (
      <div
        style={{
          fontSize: 24,
          background: "#6366f1",
          width: "100%",
          height: "100%",
          display: "flex",
          alignItems: "center",
          justifyContent: "center",
          color: "white",
          borderRadius: 6,
        }}
      >
        M
      </div>
    ),
    { ...size }
  );
}
```

### 3. Dynamic OG Image (app/opengraph-image.tsx)

```typescript
import { ImageResponse } from "next/og";

export const runtime = "edge";
export const alt = "Brand Name";
export const size = { width: 1200, height: 630 };
export const contentType = "image/png";

export default async function Image() {
  return new ImageResponse(
    (
      <div
        style={{
          height: "100%",
          width: "100%",
          display: "flex",
          flexDirection: "column",
          alignItems: "center",
          justifyContent: "center",
          backgroundColor: "#0f172a",
          backgroundImage: "linear-gradient(to bottom right, #0f172a, #1e293b)",
        }}
      >
        <div
          style={{
            display: "flex",
            alignItems: "center",
            justifyContent: "center",
            marginBottom: 40,
          }}
        >
          <div
            style={{
              width: 80,
              height: 80,
              borderRadius: 16,
              background: "#6366f1",
              display: "flex",
              alignItems: "center",
              justifyContent: "center",
              marginRight: 20,
            }}
          >
            <span style={{ fontSize: 48, color: "white" }}>B</span>
          </div>
          <span style={{ fontSize: 48, fontWeight: "bold", color: "white" }}>
            Brand Name
          </span>
        </div>
        <div
          style={{
            fontSize: 32,
            color: "#94a3b8",
            textAlign: "center",
            maxWidth: 800,
          }}
        >
          Your Tagline Here
        </div>
      </div>
    ),
    { ...size }
  );
}
```

### 4. Page-Specific OG Image

```typescript
// app/blog/[slug]/opengraph-image.tsx
import { ImageResponse } from "next/og";

export const runtime = "edge";
export const size = { width: 1200, height: 630 };
export const contentType = "image/png";

export default async function Image({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug);

  return new ImageResponse(
    (
      <div
        style={{
          height: "100%",
          width: "100%",
          display: "flex",
          flexDirection: "column",
          padding: 80,
          backgroundColor: "#0f172a",
        }}
      >
        <div style={{ fontSize: 24, color: "#6366f1", marginBottom: 20 }}>
          BLOG
        </div>
        <div
          style={{
            fontSize: 64,
            fontWeight: "bold",
            color: "white",
            lineHeight: 1.2,
          }}
        >
          {post.title}
        </div>
        <div style={{ marginTop: "auto", display: "flex", alignItems: "center" }}>
          <span style={{ fontSize: 24, color: "#94a3b8" }}>yourdomain.com</span>
        </div>
      </div>
    ),
    { ...size }
  );
}
```

### 5. PWA Manifest (app/manifest.ts)

```typescript
import type { MetadataRoute } from "next";

export default function manifest(): MetadataRoute.Manifest {
  return {
    name: "Brand Name",
    short_name: "Brand",
    description: "Your app description",
    start_url: "/",
    display: "standalone",
    background_color: "#0f172a",
    theme_color: "#6366f1",
    icons: [
      {
        src: "/icon-192.png",
        sizes: "192x192",
        type: "image/png",
      },
      {
        src: "/icon-512.png",
        sizes: "512x512",
        type: "image/png",
      },
    ],
  };
}
```

---

## Advanced Sitemap Patterns

### Sitemap with Images

```typescript
import type { MetadataRoute } from "next";

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const products = await getProducts();

  return products.map((product) => ({
    url: `${BASE_URL}/products/${product.slug}`,
    lastModified: product.updatedAt,
    images: product.images.map((img) => `${BASE_URL}${img.url}`),
  }));
}
```

### Sitemap with Alternates (i18n)

```typescript
import type { MetadataRoute } from "next";

const locales = ["en", "es", "fr", "de"];

export default function sitemap(): MetadataRoute.Sitemap {
  const pages = ["/", "/about", "/pricing"];

  return pages.flatMap((page) =>
    locales.map((locale) => ({
      url: `${BASE_URL}/${locale}${page}`,
      lastModified: new Date(),
      alternates: {
        languages: Object.fromEntries(
          locales.map((l) => [l, `${BASE_URL}/${l}${page}`])
        ),
      },
    }))
  );
}
```

### Multiple Sitemaps (Sitemap Index)

```typescript
// app/sitemap/[id]/route.ts - For custom sitemap handling
import { NextResponse } from "next/server";

export async function GET(
  request: Request,
  { params }: { params: { id: string } }
) {
  const sitemapId = parseInt(params.id);
  const urls = await getUrlsForSitemap(sitemapId);

  const xml = `<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
${urls.map((url) => `<url><loc>${url}</loc></url>`).join("\n")}
</urlset>`;

  return new NextResponse(xml, {
    headers: { "Content-Type": "application/xml" },
  });
}
```

---

## Metadata Patterns

### Conditional Metadata

```typescript
export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const { slug } = await params;
  const post = await getPost(slug);

  // Handle not found
  if (!post) {
    return {
      title: "Post Not Found",
      robots: { index: false }, // Don't index 404-ish pages
    };
  }

  // Handle draft posts
  if (post.status === "draft") {
    return {
      title: post.title,
      robots: { index: false, follow: false },
    };
  }

  return {
    title: post.title,
    description: post.excerpt,
  };
}
```

### Metadata with Search Params

```typescript
type Props = {
  params: Promise<{ slug: string }>;
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>;
};

export async function generateMetadata({ params, searchParams }: Props): Promise<Metadata> {
  const { slug } = await params;
  const { page } = await searchParams;

  const pageNum = page ? ` - Page ${page}` : "";

  return {
    title: `${slug}${pageNum}`,
    // Paginated pages should point to page 1 as canonical
    alternates: {
      canonical: `${BASE_URL}/category/${slug}`,
    },
  };
}
```

### Shared Metadata Function

```typescript
// lib/seo.ts
import type { Metadata } from "next";

const BASE_URL = process.env.NEXT_PUBLIC_SITE_URL || "https://example.com";

export function createMetadata({
  title,
  description,
  image,
  path = "",
  noIndex = false,
}: {
  title: string;
  description: string;
  image?: string;
  path?: string;
  noIndex?: boolean;
}): Metadata {
  const url = `${BASE_URL}${path}`;
  const ogImage = image || "/og-image.png";

  return {
    title,
    description,
    robots: noIndex ? { index: false, follow: false } : undefined,
    openGraph: {
      title,
      description,
      url,
      images: [{ url: ogImage, width: 1200, height: 630 }],
    },
    twitter: {
      card: "summary_large_image",
      title,
      description,
      images: [ogImage],
    },
    alternates: {
      canonical: url,
    },
  };
}

// Usage in page
export const metadata = createMetadata({
  title: "Pricing",
  description: "Simple pricing for teams of all sizes.",
  path: "/pricing",
});
```

---

## Performance SEO

### Preconnect to External Resources

```typescript
// app/layout.tsx
export const metadata: Metadata = {
  // ... other metadata
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <head>
        {/* Preconnect to external origins */}
        <link rel="preconnect" href="https://fonts.googleapis.com" />
        <link rel="preconnect" href="https://fonts.gstatic.com" crossOrigin="anonymous" />
        <link rel="dns-prefetch" href="https://analytics.example.com" />
      </head>
      <body>{children}</body>
    </html>
  );
}
```

### Image Optimization Metadata

```typescript
export const metadata: Metadata = {
  robots: {
    googleBot: {
      "max-image-preview": "large", // Allow large image previews
    },
  },
};
```

---

## Common Issues & Fixes

### Issue: Sitemap returns 404 or auth redirect

**Fix:** Add to public routes in middleware:
```typescript
const isPublicRoute = createRouteMatcher([
  // ... other routes
  "/robots.txt",
  "/sitemap.xml",
  "/sitemap(.*).xml",
]);
```

### Issue: OG image not showing

**Fix:** Check `metadataBase` is set:
```typescript
export const metadata: Metadata = {
  metadataBase: new URL(process.env.NEXT_PUBLIC_SITE_URL!),
  // Without metadataBase, relative image paths won't work
};
```

### Issue: Duplicate content from trailing slashes

**Fix:** Configure in `next.config.js`:
```javascript
module.exports = {
  trailingSlash: false, // or true, but be consistent
};
```

### Issue: www vs non-www duplicate content

**Fix:** Add redirect in `next.config.js`:
```javascript
module.exports = {
  async redirects() {
    return [
      {
        source: "/:path*",
        has: [{ type: "host", value: "www.example.com" }],
        destination: "https://example.com/:path*",
        permanent: true,
      },
    ];
  },
};
```
