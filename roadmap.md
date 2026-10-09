# Roadmap

## Migratie ROUT (rout.be)
- [x] Code overnemen van GitHub en draaiend krijgen
- [x] DATABASE_URL koppelen (Neon blijft database)
- [x] Sessiesleutels genereren (BETTER_AUTH_SECRET, APP_SESSION_SECRET, SESSION_SECRET, MASTODON_STATE_SECRET)
- [x] Inlogpagina verifiëren in preview
- [ ] OAuth-providerkeys (Google/GitHub/...) en Brevo-mailsleutel vragen bij verificatie
- [ ] Cronjobs en callback-URL's activeren na publicatie
- [ ] Domein rout.be koppelen (cutover)

## Beveiligingsrichtlijnen (gebruiker, 2026-10-09)
- [x] Rate limits forwarding-bevestigingsmails: 3/10min + 5/dag per account, 2/dag per ontvanger
      (db/55 + forwarding.server.ts, fail-closed)
- [x] Dual-tier identiteit: root-handleclaim in profielinstellingen geblokkeerd voor gratis accounts
      (writeProfileSettings); subdomeinen en donaties waren al afgeschermd via assertEntitled
- [x] Donatie-/steunpagina: al server-side afgesloten voor gratis accounts (readDonationTarget → null)
- [x] Developer Console: 4-stappen wizard (draft in sessionStorage), AI-prompts, schema-templates,
      live Auth-logs (db/56, zonder IP/PII, 7 dagen bewaard)
