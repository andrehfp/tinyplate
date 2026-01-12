# AbacatePay API Reference

## Base URL
`https://api.abacatepay.com/v1`

## Authentication
Bearer token: `Authorization: Bearer <api_key>`

---

## Billing

### POST /billing/create

Create a new payment/billing session.

**Request:**
```json
{
  "frequency": "ONE_TIME",
  "methods": ["PIX"],
  "products": [{
    "externalId": "product-123",
    "name": "Product Name",
    "quantity": 1,
    "price": 2990,
    "description": "Optional description"
  }],
  "customer": {
    "email": "user@email.com",
    "name": "João Silva",
    "cellphone": "11999999999",
    "taxId": "12345678900"
  },
  "returnUrl": "https://app.com/pricing",
  "completionUrl": "https://app.com/success"
}
```

**Response:**
```json
{
  "data": {
    "id": "bill_abc123",
    "url": "https://pay.abacatepay.com/...",
    "amount": 2990,
    "status": "PENDING"
  }
}
```

---

## PIX QR Code

### POST /pixQrCode/create

Generate a PIX QR code for direct payment.

**Request:**
```json
{
  "amount": 2990,
  "expiresIn": 3600,
  "description": "Payment description"
}
```

**Response:**
```json
{
  "data": {
    "id": "pix_abc123",
    "amount": 2990,
    "status": "PENDING",
    "brCode": "00020126...",
    "brCodeBase64": "data:image/png;base64,..."
  }
}
```

---

## Webhooks

### Event Types

| Event | Description |
|-------|-------------|
| `billing.paid` | Payment confirmed |
| `withdraw.done` | Withdrawal completed |
| `withdraw.failed` | Withdrawal failed |

### Payload Structure

```json
{
  "id": "evt_abc123",
  "event": "billing.paid",
  "devMode": false,
  "data": {
    "billing": {
      "id": "bill_123",
      "amount": 2990,
      "status": "PAID"
    }
  }
}
```

---

## Status Values

| Status | Description |
|--------|-------------|
| `PENDING` | Awaiting payment |
| `PAID` | Payment confirmed |
| `EXPIRED` | Payment window closed |
| `CANCELLED` | Manually cancelled |

---

## Pricing

| Method | Fee |
|--------|-----|
| PIX | R$ 0,80 flat |
| Credit Card | 3.5% + R$ 0,60 |
