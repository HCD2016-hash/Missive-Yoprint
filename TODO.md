# Missive-YoPrint Integration Enhancement
**Status:** Planned - Ready for implementation
**Last Updated:** 2026-01-14

---

## Goal
Enhance the YoPrint sidebar in Missive to automatically show customer data based on who you're emailing.

---

## TODO List

### Immediate Fixes
- [ ] Fix `Missive.openForm()` syntax - uses `inputs` but should use `fields`
- [ ] Fix `title` → `name` in form options
- [ ] Add `buttons` array to forms

### Core Features
- [ ] Add `Missive.on('change:conversations')` listener
- [ ] Extract emails from conversation using `Missive.getEmailAddresses()`
- [ ] Build customer-to-email cache (YoPrint can't search by email directly)
- [ ] Create `lookupCustomerByEmails()` function
- [ ] Create customer view UI mode
- [ ] Add "View All" button to switch to dashboard
- [ ] Implement fallback to dashboard mode

### API Migration
- [ ] Switch from `/api/external/` to `/v1/api/store/hub-city-design-inc/`
- [ ] Update endpoint paths for sales_order, shipment, purchase_order
- [ ] Test authentication with v1 API

---

## Architecture

### Two UI Modes

**Customer Mode** (auto-triggered when email matches):
```
┌─────────────────────────────────┐
│ 👤 Acme Corp                    │
│ john@acme.com                   │
├─────────────────────────────────┤
│ Recent Orders                   │
│ ├─ SO1234 - $500 - Pending     │
│ ├─ SO1230 - $200 - Shipped     │
│ └─ Q1225 - $150 - Quote        │
├─────────────────────────────────┤
│ [View All Dashboard]            │
└─────────────────────────────────┘
```

**Dashboard Mode** (fallback):
```
┌─────────────────────────────────┐
│ Quotes | Orders | Ship | PO     │
├─────────────────────────────────┤
│ [Search...] [Status ▼] [↻]     │
├─────────────────────────────────┤
│ Item list...                    │
└─────────────────────────────────┘
```

---

## Key Code Snippets

### Conversation Listener
```javascript
Missive.on('change:conversations', async (ids) => {
  if (ids.length === 1) {
    const [conversation] = await Missive.fetchConversations(ids);
    const emails = Missive.getEmailAddresses([conversation]);
    const customer = await lookupCustomerByEmails(emails);
    if (customer) {
      showCustomerView(customer);
    } else {
      showDashboard();
    }
  } else {
    showDashboard();
  }
}, { retroactive: true });
```

### Correct Form Syntax
```javascript
const result = await Missive.openForm({
  name: 'YoPrint API Key',  // NOT 'title'
  fields: [  // NOT 'inputs'
    {
      type: 'input',  // NOT 'text'
      data: {
        name: 'api_key',
        placeholder: 'Find it in YoPrint Settings',
        required: true
      }
    }
  ],
  buttons: [
    { type: 'cancel', label: 'Cancel' },
    { type: 'submit', label: 'Save' }
  ]
});
```

---

## YoPrint API Reference

### Base URLs
- External (current): `https://app.yoprint.com/api/external`
- v1 (new): `https://app.yoprint.com/v1/api/store/hub-city-design-inc`

### Endpoints Needed
| Purpose | Endpoint |
|---------|----------|
| List customers | `GET /customer` |
| Get customer contacts | `GET /customer/{id}/contact` |
| Search orders | `POST /sales_order/search` |
| List quotes | `GET /sales_order?type=quote` |
| List shipments | `GET /shipment` |
| List POs | `GET /purchase_order` |

### API Gotchas
- Empty arrays in request body → 422 error (omit them)
- `scoped_id_search` needs `{type: "order", scoped_id: "SO1234"}` not `{id: ...}`
- v2 endpoints require UUIDs, not scoped_ids
- Can't search customers by email directly - must fetch contacts and match client-side

---

## Missive API Reference

### Key Methods
```javascript
// Storage
await Missive.storeGet('key')
await Missive.storeSet('key', value)

// Conversations
Missive.on('change:conversations', callback, { retroactive: true })
await Missive.fetchConversations(ids)
Missive.getEmailAddresses(conversations)

// Forms
await Missive.openForm({ name, fields, buttons })

// Navigation
Missive.openURL(url)
```

### Conversation Object
```javascript
{
  id: 'conv-id',
  subject: 'Subject line',
  latest_message: {
    from_field: { address: 'email@example.com', name: 'Name' },
    to_fields: [{ address: '...', name: '...' }],
    cc_fields: [],
    bcc_fields: []
  }
}
```

---

## Next Session
Start with: "Continue implementing the Missive-YoPrint integration per TODO.md"
