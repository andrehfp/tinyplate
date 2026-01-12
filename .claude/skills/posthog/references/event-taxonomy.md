# PostHog Event Taxonomy

## Event Naming Framework

Use the **category:object_action** pattern:

```
category:object_action
   │        │     │
   │        │     └── verb (click, submit, view, create, delete)
   │        └── noun (button, form, page, item)
   └── context (signup, onboarding, dashboard, billing)
```

### Examples

```typescript
// Good
'signup:form_submit'
'onboarding:step_complete'
'dashboard:project_create'
'billing:plan_upgrade'

// Bad
'Signup Form Submitted'      // No spaces, no caps
'user_sign_up'               // Missing category
'formSubmit'                 // Use snake_case
```

---

## Naming Rules

| Rule | Good | Bad |
|------|------|-----|
| Lowercase only | `signup:form_submit` | `Signup:Form_Submit` |
| Snake case | `image_generate` | `image-generate` |
| Present tense verbs | `submit`, `create` | `submitted`, `created` |
| Category prefix | `canvas:image_generate` | `image_generate` |

---

## Property Naming Conventions

| Pattern | Example | When to Use |
|---------|---------|-------------|
| `object_adjective` | `user_id`, `project_name` | Most properties |
| `is_` prefix | `is_subscribed` | Boolean states |
| `has_` prefix | `has_completed_onboarding` | Boolean possession |
| `_count` suffix | `image_count` | Numbers/quantities |
| `_at` suffix | `created_at` | Timestamps |
| `_id` suffix | `user_id`, `project_id` | Identifiers |
| `_type` suffix | `plan_type` | Categories |

---

## Standard Event Library

### User Lifecycle

```typescript
'auth:signup_start'
'auth:signup_complete'
'auth:login_success'
'auth:logout'
```

### Onboarding

```typescript
'onboarding:start'
'onboarding:step_complete'      // { step_number, step_name }
'onboarding:skip'
'onboarding:complete'
```

### Subscription & Billing

```typescript
'billing:checkout_start'
'billing:checkout_complete'
'billing:plan_upgrade'
'billing:plan_cancel'
'billing:payment_fail'
```

---

## Anti-Patterns to Avoid

```typescript
// DON'T: Spaces or mixed case
'User Signed Up'

// DON'T: Past tense
'user_signed_up'

// DON'T: Nested objects
{ user: { id: '123' } }

// DO: Flat structure
{ user_id: '123' }
```
