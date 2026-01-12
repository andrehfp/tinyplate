# PostHog Feature Flags

## Flag Types

| Type | Use Case | Example |
|------|----------|---------|
| Boolean | On/off features | `new-dashboard` |
| Multivariate | A/B/n testing | `pricing-page` |
| JSON Payload | Configuration | `rate-limits` |

---

## Client-Side Patterns

### Boolean Flags

```typescript
'use client'
import { useFeatureFlagEnabled } from 'posthog-js/react'

function Dashboard() {
  const showNewDashboard = useFeatureFlagEnabled('new-dashboard')

  if (showNewDashboard === undefined) {
    return <DashboardSkeleton />
  }

  return showNewDashboard ? <NewDashboard /> : <OldDashboard />
}
```

### Multivariate Flags

```typescript
'use client'
import { useFeatureFlagPayload } from 'posthog-js/react'

function PricingPage() {
  const variant = useFeatureFlagPayload('pricing-experiment')

  switch (variant) {
    case 'control':
      return <PricingControl />
    case 'variant-a':
      return <PricingVariantA />
    default:
      return <PricingControl />
  }
}
```

---

## Server-Side Patterns

### In API Routes

```typescript
import { getPostHogServer } from '@/lib/posthog-server'
import { auth } from '@clerk/nextjs/server'

export async function POST(request: NextRequest) {
  const { userId } = await auth()
  const posthog = getPostHogServer()

  const useNewModel = await posthog.isFeatureEnabled('new-model', userId!)

  await posthog._shutdown()
  // ... rest of logic
}
```

---

## Rollout Strategies

### Percentage Rollout

```
Flag: new-feature
├── 0% → Testing internally
├── 10% → Beta users
├── 50% → Half of users
└── 100% → Full rollout
```

### User Targeting

```
Flag: enterprise-features
├── User property: plan = 'enterprise'
├── OR User property: is_beta_tester = true
└── OR User ID in list
```

---

## Flag Cleanup Checklist

When removing a feature flag:

- [ ] Flag is at 100% rollout
- [ ] No active experiments using it
- [ ] Remove flag checks from code
- [ ] Archive flag in PostHog
