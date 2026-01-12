# AbacatePay Integration Patterns

## 1. Subscription Management

### Check User Plan

```typescript
import { db } from "@/lib/db";
import { subscriptions, plans } from "@/lib/db/schema";
import { eq } from "drizzle-orm";

async function getUserPlan(userId: string): Promise<string> {
  const [sub] = await db
    .select()
    .from(subscriptions)
    .where(eq(subscriptions.userId, userId))
    .limit(1);

  if (!sub) return "free";

  const now = new Date();
  if (sub.status !== "active" || sub.currentPeriodEnd < now) {
    return "free";
  }

  return sub.planId;
}
```

### Feature Gating

```typescript
async function checkFeatureAccess(
  userId: string,
  feature: string
): Promise<boolean> {
  const [sub] = await db
    .select({ limits: plans.limits })
    .from(subscriptions)
    .innerJoin(plans, eq(subscriptions.planId, plans.id))
    .where(eq(subscriptions.userId, userId))
    .limit(1);

  if (!sub) {
    // Free tier defaults
    return false;
  }

  return Boolean(sub.limits[feature]);
}
```

---

## 2. Idempotent Webhook Handling

### Why Idempotency Matters

Webhooks can be delivered multiple times. Always check if the event was already processed.

```typescript
// CORRECT - Check before processing
async function handleWebhook(payload: WebhookPayload) {
  // 1. Check if already processed
  const [existing] = await db
    .select()
    .from(webhookEvents)
    .where(eq(webhookEvents.id, payload.id))
    .limit(1);

  if (existing) {
    return { message: "Already processed" };
  }

  // 2. Store event BEFORE processing
  await db.insert(webhookEvents).values({
    id: payload.id,
    eventType: payload.event,
    payload: payload,
  });

  // 3. Process the event
  await createSubscription(payload);
}
```

---

## 3. Error Handling

### Service Layer Pattern

```typescript
export async function createCheckout(params: CreateCheckoutParams) {
  try {
    const response = await abacate.billing.create({...});

    if (response.error) {
      throw new Error(response.error);
    }

    return {
      billingId: response.data!.id,
      paymentUrl: response.data!.url,
    };
  } catch (error) {
    // Log safely (no secrets)
    console.error("abacatepay-checkout", error);

    throw new Error("Failed to create payment. Please try again.");
  }
}
```

---

## 4. Price Formatting

### Display Prices in BRL

```typescript
export function formatPriceBRL(centavos: number): string {
  const reais = centavos / 100;
  return reais.toLocaleString("pt-BR", {
    style: "currency",
    currency: "BRL",
  });
}

// Usage
formatPriceBRL(2990);  // "R$ 29,90"
```

---

## 5. Webhook URL Configuration

### AbacatePay Dashboard Setup

1. Go to AbacatePay Dashboard > Webhooks
2. Add webhook URL: `https://your-domain.com/api/webhooks/abacatepay`
3. Select events: `billing.paid`
4. Copy webhook secret to env

### Local Development with ngrok

```bash
# Terminal 1: Run your app
bun run dev

# Terminal 2: Expose with ngrok
ngrok http 3000

# Use ngrok URL in AbacatePay dashboard
```
