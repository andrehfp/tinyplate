# Structured Data (JSON-LD) Templates

Structured data helps search engines understand your content and can enable rich snippets in search results.

## Implementation in Next.js

### Method 1: Script Tag in Layout/Page

> **Security Note**: When using script tags for JSON-LD, ensure the data comes from trusted sources. JSON.stringify() safely escapes the content, but never include raw user input in structured data.

```typescript
// app/layout.tsx or any page
export default function Layout({ children }: { children: React.ReactNode }) {
  const organizationSchema = {
    "@context": "https://schema.org",
    "@type": "Organization",
    name: "Brand Name",
    url: "https://example.com",
    logo: "https://example.com/logo.png",
    sameAs: [
      "https://twitter.com/brand",
      "https://linkedin.com/company/brand",
    ],
  };

  return (
    <html lang="en">
      <head>
        <script
          type="application/ld+json"
          // JSON.stringify safely escapes content for JSON-LD
          // Only use with trusted, server-generated data
          dangerouslySetInnerHTML={{ __html: JSON.stringify(organizationSchema) }}
        />
      </head>
      <body>{children}</body>
    </html>
  );
}
```

### Method 2: Component-Based (Recommended)

```typescript
// components/JsonLd.tsx
type JsonLdProps = {
  data: Record<string, unknown>;
};

export function JsonLd({ data }: JsonLdProps) {
  // JSON.stringify escapes special characters, making this safe
  // for server-generated structured data
  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(data) }}
    />
  );
}

// Usage - only with trusted data
<JsonLd data={articleSchema} />
```

---

## Schema Templates

### Organization (Homepage)

```typescript
const organizationSchema = {
  "@context": "https://schema.org",
  "@type": "Organization",
  name: "Brand Name",
  alternateName: "Brand",
  url: "https://example.com",
  logo: "https://example.com/logo.png",
  description: "Your brand description",
  foundingDate: "2024",
  founders: [
    {
      "@type": "Person",
      name: "Founder Name",
    },
  ],
  address: {
    "@type": "PostalAddress",
    addressCountry: "US",
  },
  contactPoint: {
    "@type": "ContactPoint",
    contactType: "customer service",
    email: "support@example.com",
  },
  sameAs: [
    "https://twitter.com/brand",
    "https://linkedin.com/company/brand",
    "https://instagram.com/brand",
  ],
};
```

### WebSite (Homepage)

```typescript
const websiteSchema = {
  "@context": "https://schema.org",
  "@type": "WebSite",
  name: "Brand Name",
  url: "https://example.com",
  description: "Your site description",
  potentialAction: {
    "@type": "SearchAction",
    target: {
      "@type": "EntryPoint",
      urlTemplate: "https://example.com/search?q={search_term_string}",
    },
    "query-input": "required name=search_term_string",
  },
};
```

### SoftwareApplication (Product/SaaS)

```typescript
const softwareSchema = {
  "@context": "https://schema.org",
  "@type": "SoftwareApplication",
  name: "Product Name",
  applicationCategory: "BusinessApplication",
  operatingSystem: "Web",
  offers: {
    "@type": "Offer",
    price: "19.00",
    priceCurrency: "USD",
    priceValidUntil: "2025-12-31",
  },
  aggregateRating: {
    "@type": "AggregateRating",
    ratingValue: "4.8",
    ratingCount: "150",
    bestRating: "5",
    worstRating: "1",
  },
  screenshot: "https://example.com/screenshot.png",
  featureList: [
    "Feature 1",
    "Feature 2",
    "Feature 3",
  ],
};
```

### Article (Blog Posts)

```typescript
const articleSchema = {
  "@context": "https://schema.org",
  "@type": "Article",
  headline: "Article Title",
  description: "Article description...",
  image: [
    "https://example.com/blog/article/cover-1x1.jpg",
    "https://example.com/blog/article/cover-4x3.jpg",
    "https://example.com/blog/article/cover-16x9.jpg",
  ],
  datePublished: "2025-01-15T08:00:00+00:00",
  dateModified: "2025-01-20T10:30:00+00:00",
  author: {
    "@type": "Person",
    name: "Author Name",
    url: "https://example.com/team/author",
  },
  publisher: {
    "@type": "Organization",
    name: "Brand Name",
    logo: {
      "@type": "ImageObject",
      url: "https://example.com/logo.png",
    },
  },
  mainEntityOfPage: {
    "@type": "WebPage",
    "@id": "https://example.com/blog/article-slug",
  },
};
```

### FAQPage

```typescript
const faqSchema = {
  "@context": "https://schema.org",
  "@type": "FAQPage",
  mainEntity: [
    {
      "@type": "Question",
      name: "Question 1?",
      acceptedAnswer: {
        "@type": "Answer",
        text: "Answer 1.",
      },
    },
    {
      "@type": "Question",
      name: "Question 2?",
      acceptedAnswer: {
        "@type": "Answer",
        text: "Answer 2.",
      },
    },
  ],
};
```

### Product (with Pricing)

```typescript
const productSchema = {
  "@context": "https://schema.org",
  "@type": "Product",
  name: "Product Name",
  description: "Product description",
  image: "https://example.com/product-image.png",
  brand: {
    "@type": "Brand",
    name: "Brand Name",
  },
  offers: {
    "@type": "Offer",
    url: "https://example.com/pricing",
    priceCurrency: "USD",
    price: "19.00",
    priceValidUntil: "2025-12-31",
    availability: "https://schema.org/InStock",
    seller: {
      "@type": "Organization",
      name: "Brand Name",
    },
  },
  aggregateRating: {
    "@type": "AggregateRating",
    ratingValue: "4.9",
    reviewCount: "127",
  },
};
```

### BreadcrumbList (Navigation)

```typescript
const breadcrumbSchema = {
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  itemListElement: [
    {
      "@type": "ListItem",
      position: 1,
      name: "Home",
      item: "https://example.com",
    },
    {
      "@type": "ListItem",
      position: 2,
      name: "Blog",
      item: "https://example.com/blog",
    },
    {
      "@type": "ListItem",
      position: 3,
      name: "Article Title",
      item: "https://example.com/blog/article-slug",
    },
  ],
};
```

---

## Dynamic Schema Generation

### Helper Function

```typescript
// lib/schema.ts
const BASE_URL = process.env.NEXT_PUBLIC_SITE_URL || "https://example.com";

export function generateArticleSchema(post: {
  title: string;
  excerpt: string;
  coverImage: string;
  publishedAt: string;
  updatedAt?: string;
  author: { name: string };
  slug: string;
}) {
  return {
    "@context": "https://schema.org",
    "@type": "Article",
    headline: post.title,
    description: post.excerpt,
    image: post.coverImage,
    datePublished: post.publishedAt,
    dateModified: post.updatedAt || post.publishedAt,
    author: {
      "@type": "Person",
      name: post.author.name,
    },
    publisher: {
      "@type": "Organization",
      name: "Brand Name",
      logo: {
        "@type": "ImageObject",
        url: `${BASE_URL}/logo.png`,
      },
    },
    mainEntityOfPage: {
      "@type": "WebPage",
      "@id": `${BASE_URL}/blog/${post.slug}`,
    },
  };
}

export function generateFAQSchema(
  faqs: Array<{ question: string; answer: string }>
) {
  return {
    "@context": "https://schema.org",
    "@type": "FAQPage",
    mainEntity: faqs.map((faq) => ({
      "@type": "Question",
      name: faq.question,
      acceptedAnswer: {
        "@type": "Answer",
        text: faq.answer,
      },
    })),
  };
}

export function generateBreadcrumbSchema(
  items: Array<{ name: string; url: string }>
) {
  return {
    "@context": "https://schema.org",
    "@type": "BreadcrumbList",
    itemListElement: items.map((item, index) => ({
      "@type": "ListItem",
      position: index + 1,
      name: item.name,
      item: item.url,
    })),
  };
}
```

---

## Testing Structured Data

1. **Google Rich Results Test**: https://search.google.com/test/rich-results
2. **Schema.org Validator**: https://validator.schema.org/
3. **JSON-LD Playground**: https://json-ld.org/playground/

### Common Issues

| Issue | Fix |
|-------|-----|
| Missing required fields | Check schema.org docs for required properties |
| Invalid date format | Use ISO 8601: `2025-01-15T08:00:00+00:00` |
| Invalid URL | Use absolute URLs, not relative |
| Image not accessible | Ensure images are publicly accessible |
| Duplicate schemas | Only one of each type per page |
