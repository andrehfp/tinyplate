# Technical SEO Pre-Launch Checklist

## Before Development

- [ ] Define `NEXT_PUBLIC_SITE_URL` for all environments
- [ ] Decide on www vs non-www (pick one, redirect the other)
- [ ] Decide on trailing slashes (consistent throughout)
- [ ] Plan URL structure for all content types

---

## Core Files

### Sitemap (`app/sitemap.ts`)
- [ ] File exists at `app/sitemap.ts`
- [ ] Returns all public, indexable pages
- [ ] Excludes private/auth pages (dashboard, admin, settings)
- [ ] Excludes utility pages (404, 500, auth callbacks)
- [ ] `lastModified` reflects actual content updates
- [ ] Priority values make sense (1.0 homepage, 0.8-0.9 key pages)
- [ ] Dynamic content (blog posts, products) included
- [ ] Large sites use `generateSitemaps()` for sitemap index

### Robots.txt (`app/robots.ts`)
- [ ] File exists at `app/robots.ts`
- [ ] Blocks non-production environments entirely
- [ ] Allows public pages in production
- [ ] Blocks sensitive paths: `/api/`, `/dashboard/`, `/admin/`
- [ ] Blocks auth paths: `/sign-in/`, `/sign-up/`
- [ ] Doesn't block CSS/JS/images needed for rendering
- [ ] References sitemap URL
- [ ] AI bot policy defined (block or allow GPTBot, CCBot, etc.)

### Root Metadata (`app/layout.tsx`)
- [ ] `metadataBase` set to production URL
- [ ] `title.default` and `title.template` configured
- [ ] Description is compelling (150-160 chars)
- [ ] Keywords array includes target terms
- [ ] `robots` directives set for googleBot
- [ ] OpenGraph configured with all required fields
- [ ] Twitter Card configured
- [ ] Canonical URL set via `alternates.canonical`
- [ ] `viewport` export separated from `metadata`

### Images
- [ ] OG image exists at `public/og-image.png`
- [ ] OG image is exactly 1200x630 pixels
- [ ] OG image has brand name/logo visible
- [ ] Favicon exists (either `app/icon.tsx` or `public/favicon.ico`)
- [ ] Apple touch icon exists for iOS

---

## Page-Level SEO

### Every Public Page
- [ ] Has unique `title` (50-60 characters)
- [ ] Has unique `description` (150-160 characters)
- [ ] Title includes primary keyword
- [ ] Description includes primary keyword and CTA
- [ ] OpenGraph title/description set (can inherit from page metadata)
- [ ] Canonical URL points to preferred version

### Dynamic Pages (blog, products)
- [ ] `generateMetadata` function implemented
- [ ] Handles missing/draft content gracefully
- [ ] OG image specific to content (or dynamic generation)
- [ ] Proper `article` type for blog posts
- [ ] `publishedTime` and `modifiedTime` for articles

### Landing Pages
- [ ] Each landing page has targeted title/description
- [ ] Keywords align with ad campaigns
- [ ] Separate from main navigation (if applicable)
- [ ] No-index set if they're temporary/test pages

---

## Authentication/Middleware

- [ ] Sitemap accessible without authentication
- [ ] Robots.txt accessible without authentication
- [ ] Public pages don't require auth
- [ ] Auth redirects don't interfere with crawlers
- [ ] Middleware matcher excludes static files

Test these URLs directly:
```bash
curl https://yourdomain.com/robots.txt
curl https://yourdomain.com/sitemap.xml
```

---

## URL Structure

- [ ] URLs are lowercase
- [ ] URLs use hyphens, not underscores
- [ ] URLs don't have file extensions (no `.html`)
- [ ] Trailing slash behavior is consistent
- [ ] No duplicate content (www/non-www resolved)
- [ ] Query parameters don't create duplicates
- [ ] Pagination uses `?page=X` or `/page/X` consistently

### Redirects
- [ ] Old URLs redirect to new URLs (if migrating)
- [ ] Non-www redirects to www (or vice versa)
- [ ] HTTP redirects to HTTPS
- [ ] Trailing slash redirects consistent
- [ ] No redirect chains (A→B→C should be A→C)

---

## Performance (Core Web Vitals)

Core Web Vitals affect search rankings:

### Largest Contentful Paint (LCP) < 2.5s
- [ ] Hero images optimized and lazy-loaded appropriately
- [ ] Critical CSS inlined or loaded first
- [ ] Fonts preloaded or use `font-display: swap`
- [ ] Third-party scripts deferred

### First Input Delay (FID) / Interaction to Next Paint (INP) < 200ms
- [ ] JavaScript bundle sizes minimized
- [ ] Heavy JavaScript deferred
- [ ] No layout thrashing on interaction

### Cumulative Layout Shift (CLS) < 0.1
- [ ] Images have explicit width/height
- [ ] Fonts don't cause layout shift
- [ ] Ads/embeds have reserved space
- [ ] Dynamic content doesn't shift layout

---

## Structured Data

- [ ] Organization schema on homepage
- [ ] Article schema on blog posts
- [ ] Product schema on product pages
- [ ] FAQ schema where applicable
- [ ] BreadcrumbList schema for navigation
- [ ] Tested with Google Rich Results Test

---

## Post-Launch

### Google Search Console
- [ ] Property verified
- [ ] Sitemap submitted
- [ ] Coverage report reviewed
- [ ] Core Web Vitals report reviewed

### Bing Webmaster Tools
- [ ] Property verified
- [ ] Sitemap submitted

### Testing
- [ ] Test with [Google Rich Results Test](https://search.google.com/test/rich-results)
- [ ] Test with [Facebook Sharing Debugger](https://developers.facebook.com/tools/debug/)
- [ ] Test with [Twitter Card Validator](https://cards-dev.twitter.com/validator)
- [ ] Test with [LinkedIn Post Inspector](https://www.linkedin.com/post-inspector/)
- [ ] Mobile-friendly test passed
- [ ] PageSpeed Insights score > 90

### Monitoring
- [ ] Set up rank tracking for target keywords
- [ ] Monitor Search Console for issues
- [ ] Set up alerts for crawl errors
- [ ] Monitor Core Web Vitals over time

---

## Common Mistakes to Avoid

1. **Blocking JavaScript/CSS in robots.txt** - Googlebot needs these to render pages
2. **Missing metadataBase** - Causes relative URLs to break
3. **Duplicate meta descriptions** - Every page needs unique description
4. **Missing canonical URLs** - Causes duplicate content issues
5. **Not handling trailing slashes** - Creates duplicate pages
6. **Forgetting to add SEO routes to auth middleware** - Blocks crawlers
7. **Using placeholder text** - "Lorem ipsum" in production
8. **Not testing OG images** - Social shares look broken
9. **Ignoring mobile** - Most traffic is mobile
10. **Not submitting sitemap** - Google won't know about your pages
