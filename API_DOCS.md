# SmartVibe API v1 Integration Guide

SmartVibe API v1 lets client applications create AI music generations asynchronously, check generation status, and view available credits.

Base URL:

```text
https://your-smartvibe-domain.com
```

All API v1 endpoints are under:

```text
/api/v1
```

Do not call internal or frontend routes such as `/api/internal/*`, `/api/music/*`, `/api/user/*`, `/api/whatsapp/*`, or `/api/cron/*`.

## Authentication

Send your API key in the `Authorization` header:

```http
Authorization: Bearer sv_live_xxxxxxxxxxxxxxxx_secret
```

Rules:

- Keep the API key secret. Never expose it in browser JavaScript or mobile apps without a secure backend proxy.
- Never send API keys in query strings.
- Regenerating an API key revokes the previous active key.
- Invalid, revoked, disabled, or expired keys return `401`.
- Disabled API clients return `403`.

## Request IDs

Every API response includes:

```http
X-Request-Id: req_...
```

Store this value in your logs. It helps SmartVibe support trace requests.

## Error Format

Errors use this shape:

```json
{
  "error": {
    "type": "invalid_request_error",
    "code": "VALIDATION_ERROR",
    "message": "Invalid request.",
    "param": "prompt",
    "request_id": "req_abc123"
  }
}
```

Common status codes:

| HTTP | Meaning |
|---:|---|
| `400` | Invalid JSON or invalid request |
| `401` | Missing or invalid API key |
| `402` | Insufficient credits |
| `403` | API client is disabled or not allowed |
| `404` | Generation not found |
| `409` | Idempotency conflict or duplicate request still processing |
| `422` | Request validation failed |
| `429` | Rate limit exceeded |
| `503` | Queue or generation service temporarily unavailable |

## Rate Limits

API v1 endpoints are currently limited to 100 requests per minute per authenticated user per endpoint.

If the limit is exceeded, the API returns:

```json
{
  "error": {
    "type": "rate_limit_error",
    "code": "RATE_LIMITED",
    "message": "Rate limit exceeded.",
    "request_id": "req_abc123"
  }
}
```

Clients should back off and retry after the next minute window.

## Idempotency

`POST /api/v1/generations` supports idempotency.

Use an `Idempotency-Key` header for retry-safe generation creation:

```http
Idempotency-Key: order_12345
```

Behavior:

- Same API client, same key, same payload returns the original accepted response.
- Same API client, same key, different payload returns `409`.
- Use a stable key from your system, such as an order ID or request ID.

## Create Generation

```http
POST /api/v1/generations
```

Creates a generation job and returns immediately with `202 Accepted`.

### Request Body

```json
{
  "prompt": "An uplifting electronic pop song with female vocals",
  "external_reference": "order_12345"
}
```

Optional fields:

```json
{
  "prompt": "An uplifting electronic pop song with female vocals",
  "language": "en",
  "external_reference": "order_12345",
  "metadata": {
    "customer_id": "cus_4829"
  }
}
```

Fields:

| Field | Type | Required | Notes |
|---|---|---:|---|
| `prompt` | string | yes | Music prompt, minimum 3 characters |
| `language` | string | no | Defaults to `en` |
| `external_reference` | string | no | Your own order/customer/reference ID, max 200 chars |
| `metadata` | object | no | Optional client metadata returned in API responses |

SmartVibe sets model, style, title, and instrumental options internally for v1. Put any style or title guidance directly in the `prompt`.

### Response

Status: `202 Accepted`

```json
{
  "id": "gen_01abc...",
  "status": "queued",
  "created_at": "2026-07-16T10:30:00Z",
  "estimated_credits": 1,
  "external_reference": "order_12345"
}
```

If you pass `metadata`, it is included in the response:

```json
{
  "id": "gen_01abc...",
  "status": "queued",
  "created_at": "2026-07-16T10:30:00Z",
  "estimated_credits": 1,
  "external_reference": "order_12345",
  "metadata": {
    "customer_id": "cus_4829"
  }
}
```

Save the `id`. Use it to check status.

### cURL

```bash
curl -X POST https://your-smartvibe-domain.com/api/v1/generations \
  -H "Authorization: Bearer sv_live_xxxxxxxxxxxxxxxx_secret" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: order_12345" \
  -d '{
    "prompt": "An uplifting electronic pop song with female vocals",
    "external_reference": "order_12345",
    "metadata": { "customer_id": "cus_4829" }
  }'
```

## List Generations

```http
GET /api/v1/generations
```

Returns generations created by the authenticated API client, newest first. This includes queued, processing, completed, failed, and cancelled generations.

### Query Parameters

| Parameter | Type | Required | Notes |
|---|---|---:|---|
| `external_reference` | string | no | Filter to generations created with this reference |
| `limit` | number | no | Defaults to `20`, maximum `100` |
| `before` | ISO datetime | no | Cursor from `next_before` for the next page |

### Response

```json
{
  "data": [
    {
      "id": "gen_01abc...",
      "status": "completed",
      "created_at": "2026-07-16T10:30:00Z",
      "completed_at": "2026-07-16T10:35:00Z",
      "title": "Summer Lights",
      "audio_url": "https://...",
      "image_url": "https://...",
      "lyrics": "...",
      "external_reference": "order_12345"
    },
    {
      "id": "gen_02def...",
      "status": "processing",
      "created_at": "2026-07-16T10:40:00Z",
      "external_reference": "order_12345"
    }
  ],
  "has_more": true,
  "next_before": "2026-07-16T10:40:00Z"
}
```

Use `external_reference` to group or retrieve generations associated with your own order, customer, or project ID.

### cURL

```bash
curl "https://your-smartvibe-domain.com/api/v1/generations?external_reference=order_12345&limit=20" \
  -H "Authorization: Bearer sv_live_xxxxxxxxxxxxxxxx_secret"
```

## Retrieve Generation Status

```http
GET /api/v1/generations/{generationId}
```

Use this endpoint to poll until the generation reaches a terminal status.

Webhook callbacks are planned for a future API phase. For the current v1 integration, clients should use this status endpoint as the supported way to track generation progress.

### Status Values

| Status | Meaning | Terminal |
|---|---|---:|
| `queued` | Waiting for processing | no |
| `processing` | Generation is running | no |
| `completed` | Audio result is available | yes |
| `failed` | Generation failed | yes |
| `cancelled` | Generation was cancelled | yes |

Recommended polling interval: 5 to 15 seconds.

### Processing Response

```json
{
  "id": "gen_01abc...",
  "status": "processing",
  "created_at": "2026-07-16T10:30:00Z",
  "external_reference": "order_12345"
}
```

### Completed Response

```json
{
  "id": "gen_01abc...",
  "status": "completed",
  "created_at": "2026-07-16T10:30:00Z",
  "completed_at": "2026-07-16T10:35:00Z",
  "title": "Summer Lights",
  "audio_url": "https://...",
  "image_url": "https://...",
  "lyrics": "...",
  "external_reference": "order_12345"
}
```

### Failed Response

```json
{
  "id": "gen_01abc...",
  "status": "failed",
  "created_at": "2026-07-16T10:30:00Z",
  "error": {
    "code": "GENERATION_FAILED",
    "message": "Generation failed.",
    "retryable": true
  },
  "external_reference": "order_12345"
}
```

### cURL

```bash
curl https://your-smartvibe-domain.com/api/v1/generations/gen_01abc... \
  -H "Authorization: Bearer sv_live_xxxxxxxxxxxxxxxx_secret"
```

## Credit Balance

```http
GET /api/v1/credits/balance
```

Returns the authenticated account's available and reserved credits.

### Response

```json
{
  "availableCredits": 25,
  "reservedCredits": 1,
  "rules": [
    {
      "requestType": "standard_audio",
      "displayName": "Standard Audio Generation",
      "estimatedCost": 1
    }
  ]
}
```

### cURL

```bash
curl https://your-smartvibe-domain.com/api/v1/credits/balance \
  -H "Authorization: Bearer sv_live_xxxxxxxxxxxxxxxx_secret"
```

## Webhooks

Webhooks are not available for third-party clients in the current v1 release.

The current supported integration model is polling:

```text
POST /api/v1/generations
-> save returned generation id
-> poll GET /api/v1/generations/{generationId}
-> stop when status is completed, failed, or cancelled
```

Webhook support is planned as a next-phase implementation. When released, clients will be able to register a callback URL and receive signed events for generation status changes.

Planned webhook events:

| Event | Meaning |
|---|---|
| `generation.processing` | Generation processing started |
| `generation.completed` | Generation completed and result is available |
| `generation.failed` | Generation failed |

Until webhook endpoint management and delivery retries are released, do not depend on webhook callbacks for production integrations.

## TypeScript Client Example

```ts
type SmartVibeGeneration = {
  id: string;
  status: "queued" | "processing" | "completed" | "failed" | "cancelled";
  created_at: string;
  completed_at?: string;
  title?: string | null;
  audio_url?: string | null;
  image_url?: string | null;
  lyrics?: string | null;
  external_reference?: string | null;
  metadata?: Record<string, unknown>;
  error?: {
    code: string;
    message: string;
    retryable: boolean;
  };
};

class SmartVibeApiError extends Error {
  constructor(
    message: string,
    public readonly status: number,
    public readonly requestId: string | null,
    public readonly code?: string,
  ) {
    super(message);
  }
}

async function smartVibeFetch<T>(
  path: string,
  options: RequestInit & { apiKey: string },
): Promise<T> {
  const response = await fetch(`https://your-smartvibe-domain.com${path}`, {
    ...options,
    headers: {
      Authorization: `Bearer ${options.apiKey}`,
      "Content-Type": "application/json",
      ...options.headers,
    },
  });

  const requestId = response.headers.get("x-request-id");
  const body = await response.json();

  if (!response.ok) {
    throw new SmartVibeApiError(
      body.error?.message ?? "SmartVibe API request failed",
      response.status,
      requestId,
      body.error?.code,
    );
  }

  return body as T;
}

async function createGeneration(apiKey: string) {
  return smartVibeFetch<SmartVibeGeneration>("/api/v1/generations", {
    apiKey,
    method: "POST",
    headers: {
      "Idempotency-Key": "order_12345",
    },
    body: JSON.stringify({
      prompt: "An uplifting electronic pop song with female vocals",
      external_reference: "order_12345",
      metadata: { customer_id: "cus_4829" },
    }),
  });
}

async function listGenerations(apiKey: string, externalReference?: string) {
  const params = new URLSearchParams({ limit: "20" });
  if (externalReference) {
    params.set("external_reference", externalReference);
  }

  return smartVibeFetch<{
    data: SmartVibeGeneration[];
    has_more: boolean;
    next_before: string | null;
  }>(`/api/v1/generations?${params}`, {
    apiKey,
    method: "GET",
  });
}

async function waitForGeneration(apiKey: string, generationId: string) {
  while (true) {
    const generation = await smartVibeFetch<SmartVibeGeneration>(
      `/api/v1/generations/${generationId}`,
      { apiKey, method: "GET" },
    );

    if (["completed", "failed", "cancelled"].includes(generation.status)) {
      return generation;
    }

    await new Promise((resolve) => setTimeout(resolve, 10_000));
  }
}
```

## Recommended Integration Flow

1. Store the API key securely on your backend.
2. Check credits with `GET /api/v1/credits/balance` if you want to preflight availability.
3. Create a generation with `POST /api/v1/generations`.
4. Save the returned generation `id` with your local order or user record.
5. Optionally list related generations with `GET /api/v1/generations?external_reference=...`.
6. Poll `GET /api/v1/generations/{generationId}` every 5 to 15 seconds.
7. Stop polling when status is `completed`, `failed`, or `cancelled`.
8. On `completed`, download or store the returned `audio_url`, `image_url`, and `lyrics` as needed.

Webhook callbacks will be added in a future phase. Polling is required for current v1 clients.

## Production Checklist

- Keep API keys on the server side only.
- Use `Idempotency-Key` for every create request.
- Log `X-Request-Id` for every request.
- Treat `503` as retryable with exponential backoff.
- Do not retry `402` without adding credits.
- Do not retry `409` with the same idempotency key and a different payload.
- Store the generation `id` returned by SmartVibe.
- Poll status until a terminal state is reached.
- Do not rely on webhooks until webhook support is officially released.
