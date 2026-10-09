# Developer Console overhaul: wizard, AI prompts, schema templates, auth debugger

## 1. Four-step "New app" wizard
Replaces the current "Nieuwe app" dialog on the Apps page with a full-page wizard (`/console/apps/new`) and a progress bar.
- **Step 1 – App info:** name, logo upload, app type (Web server / SPA / Native-mobile). The type sets things up automatically: SPA and Native get PKCE on and no client secret; Web gets a secret, and PKCE stays recommended.
- **Step 2 – ROUT Superpowers:** two large toggle cards with plain-language explanations and a small visual example:
  - Account Auto-Discovery: "Users who already have an account in your app get linked automatically instead of creating a duplicate."
  - Rich Identity: "Get verified badges (person, business, influencer) along with the login."
- **Step 3 – Scopes & redirects:** scope checkboxes with an explanation for each, plus a redirect URI list with live validation (https only, localhost allowed in testing).
- **Step 4 – Launchpad:** client ID, client secret shown once with a copy button, the Discovery URL, and shortcuts to AI Prompts, Schema templates and the Auth debugger.
- Nothing is saved until step 4. Then the existing save functions run on the server.

## 2. AI Prompts (new app tab "AI Prompts")
- Choose a tool (Cursor, Copilot, Lovable, Generic) and a stack (Next.js, TanStack, Express, other).
- A generated prompt fills in the real client_id, Discovery URL, chosen scopes, the PKCE requirement, redirect URIs and the active Superpowers. It tells the AI to build the OIDC flow (authorization code + PKCE, state/nonce, token exchange on the server, claim validation) and to extend the database with `rout_id` plus verified-badge columns.
- Copy button. The prompt never contains the secret; it uses an env-var placeholder instead.

## 3. Schema templates (new tab "Schema")
- Tabs for SQL (Postgres) and Prisma. The templates change based on the app's Superpowers:
  - a users table with a unique `rout_id`, `email_verified`, and `is_verified_person`, `is_verified_business` and `is_influencer` flags
  - an account-link table for Auto-Discovery merges, with a uniqueness rule and a merge note
- Each template comes with a short diagram, an explanation and a copy button.

## 4. Live Auth debugger (new tab "Auth logs")
- A terminal-style view of recent sign-in attempts for this app: time, endpoint (authorize/token/userinfo), result, error code, and a short redirect/IP fingerprint (no tokens or codes).
- Each error comes with a human-readable fix, for example:
  - `invalid_redirect_uri` -> "Add exactly this URI under Redirects" (shows the URI that was received)
  - missing `code_challenge` -> "Your app type requires PKCE; send code_challenge + S256"
  - `invalid_client`, IP blocked, wrong secret, `invalid_grant`/PKCE mismatch, expired code, scope not allowed
- Refreshes automatically every 3 seconds while the tab is open, with pause and filter (errors only).
- Logs are kept for 7 days and then purged automatically.

## Technical details
- New migration `db/56_oauth_debug_events.sql`: table `oauth_debug_events` (client_id, endpoint, outcome, error_code, detail jsonb, created_at) with an index on (client_id, created_at). Plus a `runSchemaEnsure` safety net.
- `provider.server.ts`: one `logOAuthEvent()` helper that is called wherever an `OAuthError` is thrown in authorize/token/userinfo, and on success. It is fire-and-forget and never blocks the flow. Only redacted fields are stored.
- `console.functions.ts`: `listOAuthDebugEvents` (owner check, same as the other app functions). The wizard reuses `saveOAuthClient` + `saveOAuthClientSettings`, extended with `appType` and logo.
- An error-to-fix catalog in `src/lib/oauth/debug-hints.ts`, shared and tested.
- Prompt and schema generators are pure functions in `src/lib/oauth/integration-templates.ts`, with tests.
- New routes: `console.apps.new.tsx`, `console.apps.$appId.ai-prompts.tsx`, `console.apps.$appId.schema.tsx`, `console.apps.$appId.auth-logs.tsx`. These are added to the app sub-navigation inside the existing console shell.
- Purging old logs runs through the existing cron route.
