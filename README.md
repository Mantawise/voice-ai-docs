# @mantawise/voice-ai — Integration Guide

This guide explains how to embed the Mantawise Voice AI assistant in your web application using the `@mantawise/voice-ai` SDK.

---

## How it works

The SDK renders a single `<iframe>` that loads the Mantawise Voice AI app. The app is hosted and maintained by Mantawise — you don't need to manage any backend infrastructure. Updates to the AI assistant are delivered automatically.

Your configuration (API key, branding, location) is passed as props to the React component. The SDK serialises these into the iframe URL.

---

## Getting started

1. **Get an API key** — contact [yvar@mantawise.ai](mailto:yvar@mantawise.ai) to receive your API key and location code.
2. **Register your domain** — provide Mantawise with the full origin(s) your application runs on (e.g. `https://yourapp.com`). Mantawise adds these to your location's allowlist. Without this step the iframe will be blocked by the browser in production.
3. **Install the SDK** — `npm install @mantawise/voice-ai`
4. **Add the component** — see Quick Start below.
5. **Go live** — deploy over HTTPS. The microphone will not work on plain HTTP.

---

## Requirements

- **React 18 or higher**
- The host page must be served over **HTTPS** (required for microphone access)
- Recommended iframe size: full-screen or minimum **768 × 1024px** (portrait, 13" iPad)
- An API key issued by Mantawise

---

## Installation

```bash
npm install @mantawise/voice-ai
```

Store your API key in an environment variable — never hardcode it in your source:

```bash
# .env.local
NEXT_PUBLIC_VOICE_AI_KEY=your-api-key
```

```tsx
<VoiceAI apiKey={process.env.NEXT_PUBLIC_VOICE_AI_KEY!} locationCode="JFK" />
```

Note: the key is included in the iframe URL and is therefore visible in the browser. The primary protection against unauthorised use is domain whitelisting — see [Security](#security).

---

## Quick Start

```tsx
import { VoiceAI } from '@mantawise/voice-ai'

export default function KioskPage() {
  return (
    <div style={{ width: '100vw', height: '100vh' }}>
      <VoiceAI
        apiKey="your-api-key"
        locationCode="JFK"
      />
    </div>
  )
}
```

The component fills 100% of its parent container. Wrap it in a `<div>` with explicit dimensions to control its size.

---

## Props

### Required

| Prop | Type | Description |
|---|---|---|
| `apiKey` | `string` | Your Mantawise API key |

### Location

| Prop | Type | Default | Description |
|---|---|---|---|
| `locationCode` | `string` | `"JFK"` | Location identifier. Determines which flight data source and AI agent are used. Contact Mantawise to request a new location. |

### Behaviour

| Prop | Type | Default | Description |
|---|---|---|---|
| `mode` | `"kiosk" \| "mobile"` | `"kiosk"` | `kiosk`: inactivity timer auto-resets to home after 60s. `mobile`: no auto-reset. |
| `avatar` | `boolean` | `true` | `true`: show the AI avatar. `false`: voice-only UI with a microphone icon. Use `false` for native mobile app WebViews — the AI avatar is not supported in that context. |
| `deviceId` | `string` | — | Unique identifier for the kiosk or device. Used for analytics. |

### Branding

All branding props are optional. When omitted, Mantawise defaults are used.

| Prop | Type | Description |
|---|---|---|
| `brandName` | `string` | Name shown as a wordmark in the header and on the home screen. |
| `logoUrl` | `string` | URL to your logo image. Shown instead of the wordmark when provided. Recommended size: 180 × 60px. |
| `primaryColor` | `string` | Accent colour (hex, e.g. `"#0057A8"`). Used for pulse rings, borders, and interactive elements. |
| `backgroundColor` | `string` | Page background colour (hex, e.g. `"#0B1A2E"`). |

### Passenger Context

| Prop | Type | Description |
|---|---|---|
| `context` | `Record<string, string>` | Key-value pairs injected into the AI agent's prompt at session start. The agent will proactively use this information in the conversation. |

Example:
```tsx
context={{
  passengerName: 'Maria Santos',
  bookingReference: 'ABC123',
  assistanceRequired: 'wheelchair',
}}
```

The agent will greet the passenger by name and be aware of their assistance needs before they say anything.

Context values are injected as dynamic variables into the agent's prompt. To use them, the agent must be configured by Mantawise with the corresponding `{{placeholder}}` tags. Contact [yvar@mantawise.ai](mailto:yvar@mantawise.ai) to set up context variables for your location — provide the key names you intend to pass and Mantawise will configure the agent accordingly.

### Post-Call Screen

| Prop | Type | Default | Description |
|---|---|---|---|
| `postCall` | `boolean \| PostCallConfig` | `true` | Controls the post-call screen shown after the conversation ends. |

`PostCallConfig` object:
```ts
{
  showQR?: boolean           // Show a QR code linking to the full transcript. Default: true
  showRating?: boolean       // Show a 5-point satisfaction rating. Default: true
  transcriptBaseUrl?: string // Base URL for the transcript QR link. Default: your Mantawise app URL
}
```

Examples:
```tsx
// Disable the post-call screen entirely
postCall={false}

// Show rating only, no QR code
postCall={{ showQR: false, showRating: true }}

// Use your own domain for the transcript link
postCall={{ transcriptBaseUrl: 'https://yourapp.com' }}
```

By default the QR code links to the Mantawise-hosted transcript viewer — no setup required. Set `transcriptBaseUrl` only if you want the QR code to point to your own domain. In that case you are responsible for building a `/view-transcript` page that reads the `id` and `f` query parameters appended by the SDK. Contact [yvar@mantawise.ai](mailto:yvar@mantawise.ai) for the transcript API details.

```tsx
```

### Development

| Prop | Type | Description |
|---|---|---|
| `appUrl` | `string` | Override the hosted app URL. Use `"http://localhost:3000"` when running the app locally. |

---

## Examples

### Standard kiosk (JFK, with branding)

```tsx
<VoiceAI
  apiKey="your-api-key"
  locationCode="JFK"
  mode="kiosk"
  brandName="Ostrum"
  primaryColor="#0057A8"
  backgroundColor="#0B1A2E"
/>
```

### Mobile embed (no avatar, no auto-reset)

```tsx
<VoiceAI
  apiKey="your-api-key"
  locationCode="AMS"
  mode="mobile"
  avatar={false}
/>
```

### With passenger context (pre-check-in flow)

```tsx
<VoiceAI
  apiKey="your-api-key"
  locationCode="JFK"
  context={{
    passengerName: user.name,
    flightNumber: booking.flightNumber,
    assistanceRequired: booking.specialAssistance,
  }}
/>
```

### Custom post-call with your own transcript domain

```tsx
<VoiceAI
  apiKey="your-api-key"
  locationCode="AMS"
  postCall={{
    showQR: true,
    showRating: true,
    transcriptBaseUrl: 'https://yourapp.com',
  }}
/>
```

### Local development

```tsx
<VoiceAI
  apiKey="your-api-key"
  locationCode="JFK"
  appUrl="http://localhost:3000"
/>
```

---

## Sizing

The `<VoiceAI>` component renders an `<iframe>` with `width: 100%` and `height: 100%`. You control the size by sizing the parent container.

**Full screen:**
```tsx
<div style={{ width: '100vw', height: '100vh' }}>
  <VoiceAI apiKey="..." locationCode="JFK" />
</div>
```

**Fixed size (iPad portrait):**
```tsx
<div style={{ width: 768, height: 1024 }}>
  <VoiceAI apiKey="..." locationCode="JFK" />
</div>
```

**Tailwind CSS:**
```tsx
<div className="w-full h-screen">
  <VoiceAI apiKey="..." locationCode="JFK" />
</div>
```

You can also pass `className` or `style` directly to the component — these are applied to the `<iframe>` element itself.

---

## Languages

The app always shows a language selection screen before the conversation starts. The passenger selects their preferred language from a list of 11:

English · Arabic · Chinese · Dutch · French · German · Hindi · Japanese · Portuguese · Spanish · Turkish

The AI agent responds in the selected language for the entire conversation. No configuration is required on your end.

---

## TypeScript

The SDK is fully typed. Import types directly:

```ts
import type { VoiceAIProps, PostCallConfig, FlightWarning } from '@mantawise/voice-ai'
```

---

## Platform Integrations (Booking API)

If you want the AI agent to create bookings or interact with your own platform mid-conversation, Mantawise can connect to your API server-side. This is configured by Mantawise — your API credentials are stored securely as environment variables on the Mantawise platform and never exposed to the browser.

To set this up, provide Mantawise with:
- The endpoint URL of your booking API
- An API key or bearer token with write access

Mantawise will configure the AI agent with a `create_booking` tool that calls your endpoint. The agent can then create bookings, log assistance requests, or trigger any workflow in your platform mid-conversation.

Contact [yvar@mantawise.ai](mailto:yvar@mantawise.ai) to set this up for your location.

---

## Security

### How it works

The platform uses CSP `frame-ancestors` to prevent unauthorised embedding.

**CSP `frame-ancestors` (browser-enforced)**
Every response from the hosted app includes a `Content-Security-Policy: frame-ancestors` header listing the domains authorised to embed it. The browser refuses to render the iframe if the parent page's domain is not on the list. This cannot be bypassed from within a browser.

### Registering your domain

To activate domain whitelisting for your location, provide Mantawise with the full origin(s) your application runs on:

```
https://ostrum.com
https://kiosk.jfk.ostrum.com
```

Mantawise adds these to your location's `allowedOrigins` in the registry. No change is required to your SDK integration — the SDK does not need to pass any domain information.

---

## Troubleshooting

**The iframe doesn't load / shows a blank frame**
Your domain is likely not on the allowlist for your location. Provide Mantawise with your full origin (e.g. `https://yourapp.com`) to activate domain whitelisting. During local development this check is disabled — the issue only appears in production.

**"Failed to start voice call" after selecting a language**
The API key is invalid or missing. Check that `apiKey` is correctly set and matches the key issued by Mantawise.

**Microphone not working**
The host page must be served over HTTPS. Browsers block microphone access on plain HTTP. Also verify that the `<iframe>` element has `allow="microphone"` — the SDK sets this automatically, but any wrapper or CSP on your page must not override it.

**Avatar video not showing**
The AI avatar (`avatar={true}`) requires a WebRTC-capable browser. It is not supported in native mobile app WebViews. Use `avatar={false}` for native app deployments.

**Language not switching**
The language is selected by the passenger on the language screen inside the widget. There is no prop to pre-select or skip the language screen — this is by design.

---

## Support

For API keys, new locations, or technical questions, contact [yvar@mantawise.ai](mailto:yvar@mantawise.ai).
