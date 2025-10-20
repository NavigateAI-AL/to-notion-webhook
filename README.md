# to-notion-webhook

Skapa Notion-sida från GPT via webhook med kontrollerad autentisering och spårbarhet.

## Syfte och Mål

**Syfte:** Skapa Notion-sida från GPT via webhook med kontrollerad autentisering och spårbarhet.

**Mål:**
- <500 ms cold-start latens
- <3 s end-to-end responstid
- 99.9% lyckade anrop under normal last

## Målgrupp och Åtkomst

- Endast interna GPTs på Navigate AI
- Webhook kräver Bearer-autentisering
- Token roteras kvartalsvis

## Krav

1. **Funktionalitet:**
   - Kan skapa sidor i en given Notion-databas
   - Stöd för Name + valfria fält + children-blocks
   - Idempotens via `request_id` för att undvika dubbletter
   - Loggning av audit-fält
   - Hård timeout och retry-policy

2. **Säkerhet:**
   - Bearer-token autentisering
   - Kontrollerad åtkomst
   - Säker hantering av hemligheter (NOTION_TOKEN, NOTION_DATABASE_ID, AUTH_TOKEN)

3. **Prestanda:**
   - Snabb respons
   - Robust felhantering
   - Rate limiting

## Process

### A. Förberedelser

1. Skapa Notion-integration (internal)
2. Koppla integrationen till databasen
3. Hämta database_id
4. Säkra hemligheter: NOTION_TOKEN, NOTION_DATABASE_ID, AUTH_TOKEN
5. Skapa separat staging-databas

### B. Teknisk Stack

- **Runtime:** Node.js med TypeScript
- **Framework:** Express
- **Validering:** Zod
- **API-klient:** node-fetch
- **Loggning:** Pino
- **Deployment:** Vercel

### C. Projektstruktur

```
to-notion-webhook/
├── src/
│   ├── server.ts       # Express server
│   ├── notion.ts       # Notion API-klient
│   ├── schema.ts       # Zod-validering
│   ├── idempotency.ts  # Idempotens-hantering
│   └── logger.ts       # Loggning
├── vercel.json         # Vercel-konfiguration
├── tsconfig.json       # TypeScript-konfiguration
└── package.json        # NPM-konfiguration
```

### D. API-specifikation

**Endpoint:** `POST /api/to-notion`

**Headers:**
```
Authorization: Bearer <AUTH_TOKEN>
Content-Type: application/json
```

**Request Body:**
```json
{
  "title": "string (required)",
  "content": "string (optional, max 10000)",
  "properties": "object (optional)",
  "children": "array (optional)",
  "request_id": "uuid (optional, för idempotens)"
}
```

**Response:**
```json
{
  "success": true,
  "page_id": "string",
  "url": "string",
  "title": "string"
}
```

## Säkerhet och Drift

- **Autentisering:** Alla anrop kräver giltig Bearer-token
- **Rate Limiting:** 10 requests/min per IP (in-memory token bucket)
- **Timeout:** 8 sekunder med retry-logik (max 3 försök)
- **Loggning:** Alla anrop loggas med status, latens och page_id
- **Alerting:** >2% fel under 5 min triggar varning
- **Token Rotation:** Kvartalsvis rotation av alla tokens

## Installation och Deployment

### Lokalt

```bash
# Installera beroenden
npm install

# Konfigurera miljövariabler (.env)
NOTION_TOKEN=secret_xxx
NOTION_DATABASE_ID=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
AUTH_TOKEN=to-notion-auth-<slump>
NOTION_VERSION=2022-06-28
LOG_LEVEL=info

# Kör lokalt
npm run dev
```

### Vercel

1. Konfigurera miljövariabler i Vercel Project Settings
2. Deploy via GitHub-integration
3. Verifiera deployment med testanrop

## GPT Actions Integration

**URL:** `https://<deploy>.vercel.app/api/to-notion`

**Method:** POST

**Schema:**
```json
{
  "type": "object",
  "properties": {
    "title": { "type": "string" },
    "content": { "type": "string" },
    "properties": { "type": "object" },
    "children": { "type": "array" },
    "request_id": { "type": "string", "description": "UUID for idempotency" }
  },
  "required": ["title"]
}
```

## License

MIT
