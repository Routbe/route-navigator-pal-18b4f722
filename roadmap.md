# Roadmap

## Migratie ROUT (rout.be)
- [x] Code overnemen van GitHub en draaiend krijgen
- [x] DATABASE_URL koppelen (Neon blijft database)
- [x] Sessiesleutels genereren (BETTER_AUTH_SECRET, APP_SESSION_SECRET, SESSION_SECRET, MASTODON_STATE_SECRET)
- [ ] Inlogpagina verifiëren in preview
- [ ] OAuth-providerkeys (Google/GitHub/...) en Brevo-mailsleutel vragen bij verificatie
- [ ] Cronjobs en callback-URL's activeren na publicatie
- [ ] Domein rout.be koppelen (cutover)

## Beveiligingsrichtlijnen (gebruiker, 2026-10-09)
- [ ] Rate limits op forwarding-bevestigingsmails (backend, strikt):
      max 3 per 10 min per account, max 5 per dag per account, max 2 per dag per ontvangeradres
- [ ] Dual-tier identiteit afdwingen: gratis gebruikers alleen via /u/ namespace (rout.be/u/alias),
      nooit root-handle/subdomein; root-handle alleen voor Paid Lifetime + geverifieerd;
      admin kan alias upgraden in Super Admin Portal
- [ ] Donatie-/steunpagina volledig afgesloten voor gratis accounts (backend + UI)
