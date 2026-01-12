# PostHog Next.js Implementation

## Project Structure

```
app/
├── layout.tsx              # Wrap with PostHogProvider
├── providers.tsx           # PostHog client provider
├── api/
│   └── [...]/route.ts      # Use posthog-node for server events
lib/
├── posthog-server.ts       # PostHog Node client singleton
components/
├── posthog-identify.tsx    # User identification component
```

---

## Complete Provider Setup

### app/providers.tsx

```typescript
'use client'

import posthog from 'posthog-js'
import { PostHogProvider as PHProvider } from 'posthog-js/react'
import { useEffect, Suspense } from 'react'
import { usePathname, useSearchParams } from 'next/navigation'

// Pageview tracking component
function PostHogPageView() {
  const pathname = usePathname()
  const searchParams = useSearchParams()

  useEffect(() => {
    if (pathname && posthog) {
      let url = window.origin + pathname
      if (searchParams.toString()) {
        url = url + '?' + searchParams.toString()
      }
      posthog.capture('$pageview', { $current_url: url })
    }
  }, [pathname, searchParams])

  return null
}

export function PostHogProvider({ children }: { children: React.ReactNode }) {
  useEffect(() => {
    posthog.init(process.env.NEXT_PUBLIC_POSTHOG_KEY!, {
      api_host: '/ingest', // Use reverse proxy
      ui_host: 'https://us.i.posthog.com',
      defaults: '2025-05-24',
      capture_pageview: false, // Manual control
      person_profiles: 'identified_only',
      session_recording: {
        maskAllInputs: false,
        maskInputOptions: { password: true },
      },
      loaded: (posthog) => {
        if (process.env.NODE_ENV === 'development') {
          posthog.debug()
        }
      },
    })
  }, [])

  return (
    <PHProvider client={posthog}>
      <Suspense fallback={null}>
        <PostHogPageView />
      </Suspense>
      {children}
    </PHProvider>
  )
}
```

### app/layout.tsx

```typescript
import { PostHogProvider } from './providers'
import { PostHogIdentify } from '@/components/posthog-identify'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>
        <PostHogProvider>
          <PostHogIdentify />
          {children}
        </PostHogProvider>
      </body>
    </html>
  )
}
```

---

## Server Client Setup

### lib/posthog-server.ts

```typescript
import { PostHog } from 'posthog-node'

export function getPostHogServer(): PostHog {
  return new PostHog(process.env.NEXT_PUBLIC_POSTHOG_KEY!, {
    host: process.env.NEXT_PUBLIC_POSTHOG_HOST || 'https://us.i.posthog.com',
    flushAt: 1,
    flushInterval: 0,
  })
}

export async function captureServerEvent(
  userId: string,
  event: string,
  properties?: Record<string, unknown>
) {
  const posthog = getPostHogServer()

  posthog.capture({
    distinctId: userId,
    event,
    properties,
  })

  // CRITICAL: Must use _shutdown() - it returns a Promise
  await posthog._shutdown()
}
```

---

## User Identification

### With Clerk

```typescript
'use client'

import { useEffect } from 'react'
import { useAuth, useUser } from '@clerk/nextjs'
import { usePostHog } from 'posthog-js/react'

export function PostHogIdentify() {
  const { isSignedIn, userId } = useAuth()
  const { user } = useUser()
  const posthog = usePostHog()

  useEffect(() => {
    if (isSignedIn && userId && user) {
      posthog.identify(userId, {
        email: user.primaryEmailAddress?.emailAddress,
        name: user.fullName,
        created_at: user.createdAt?.toISOString(),
      })
    } else if (!isSignedIn) {
      posthog.reset()
    }
  }, [isSignedIn, userId, user, posthog])

  return null
}
```

### With NextAuth

```typescript
'use client'

import { useEffect } from 'react'
import { useSession } from 'next-auth/react'
import { usePostHog } from 'posthog-js/react'

export function PostHogIdentify() {
  const { data: session, status } = useSession()
  const posthog = usePostHog()

  useEffect(() => {
    if (status === 'authenticated' && session?.user) {
      posthog.identify(session.user.id, {
        email: session.user.email,
        name: session.user.name,
      })
    } else if (status === 'unauthenticated') {
      posthog.reset()
    }
  }, [status, session, posthog])

  return null
}
```

---

## Reverse Proxy Setup

### next.config.ts

```typescript
const nextConfig: NextConfig = {
  async rewrites() {
    return [
      {
        source: "/ingest/static/:path*",
        destination: "https://us-assets.i.posthog.com/static/:path*",
      },
      {
        source: "/ingest/:path*",
        destination: "https://us.i.posthog.com/:path*",
      },
      {
        source: "/ingest/decide",
        destination: "https://us.i.posthog.com/decide",
      },
    ];
  },
};
```

---

## Environment Variables

### .env.local

```bash
NEXT_PUBLIC_POSTHOG_KEY=phc_your_project_key
NEXT_PUBLIC_POSTHOG_HOST=https://us.i.posthog.com
```

---

## Common Issues

| Issue | Solution |
|-------|----------|
| Ad blockers blocking PostHog | Use reverse proxy |
| Events not appearing | Verify API key, use reverse proxy |
| Server events not flushing | Use `await posthog._shutdown()` |
| Duplicate pageviews | Use `capture_pageview: false` |
| Missing user data | Call `identify()` BEFORE `$pageview` |
