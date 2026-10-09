# Redirect-URI hulp voor inlogproviders in de Developer Console

## Wat je krijgt

Een nieuwe pagina in de Developer Console: **Inlogproviders** (`/console/providers`), alleen zichtbaar voor beheerders. Per provider staat daar:

- de exacte callback-URL die je bij de provider invult, met één klik te kopiëren
- welke sleutelnamen je op de server moet zetten, ook te kopiëren
- één zin over wáár je het plakt (Google Cloud Console, GitHub OAuth App, Apple, GitLab, Infomaniak, eigen OIDC-provider)
- of die provider nu al werkt of nog sleutels mist, zonder de waarden zelf te tonen

Onderaan staan de eindpunten die je zelf aan een app meegeeft die "Login met ROUT" gebruikt, zodat je beide kanten op één plek hebt.

## Stappen

1. **Registry** — `src/lib/provider-setup.ts` (nieuw): één vaste lijst met per inlogprovider de merknaam, het callbackpad, de sleutelnamen en de plaats waar je het invult. Dit wordt de enige bron voor de schermen.

2. **Kopieerveld** — `src/components/console/CopyField.tsx` (nieuw): label, waarde in monospace, kopieerknop met "Gekopieerd"-melding. Lege velden tonen een streepje.

3. **De pagina** — `src/routes/_authenticated/console.providers.tsx` + `src/pages/ConsoleProviders.tsx`: status per provider, de callback-URL als kopieerveld, de ontbrekende sleutelnamen als kopieerveld, en de invulinstructie. Bovenaan de basis-URL met de drie feiten die nu verrassend zijn: preview-URLs werken niet voor social login, lokaal loopt de callback over de eigen dev-poort, en op rout.be is alles https. Onderaan de ROUT-eindpunten voor eigen apps.

4. **Navigatie** — item "Inlogproviders" in de console-zijbalk, alleen getoond aan beheerders.

5. **Healthpaneel opwaarderen** — `src/components/admin/EnvHealthPanel.tsx` toont de callback-URLs nu als kopieerveld in plaats van platte tekst, met een doorverwijzing naar de nieuwe pagina.

6. **Diagnose afzetten naar beheerders** — de diagnosevraag op `/api/public/auth/providers?diagnose=1` is nu voor elke bezoeker te lezen en laat zien welke sleutelnamen ontbreken, of de sessiesleutel staat en een ruwe database-foutmelding. Die vraag krijgt dezelfde beheerderscontrole als de andere beheerfuncties. De inlogpagina gebruikt alleen de gewone providerlijst zonder diagnose en blijft dus werken.

7. **Documentatie** — `ENVIRONMENT.md` en `.env.example` (nieuw, alleen namen, nooit waarden): elke variabele die de code uitleest, per provider met de bijbehorende callback-URL. Nu ontbreken die bestanden volledig.

## Daarna

Zodra de pagina staat, open ik het beveiligde formulier voor de echte sleutels (Google, GitHub, GitLab, Apple, Infomaniak, eigen OIDC) zodat je ze niet in de chat hoeft te plakken.

## Technische toelichting

- `src/lib/provider-setup.ts`: browser-veilig, geen server-imports. Beperkt tot de zes providers die `src/lib/better-auth.server.ts` nu realmente bedraadt (google, github, gitlab, apple via socialProviders; oidc en infomaniak via de genericOAuth-plugin). Per provider: `id`, label, callbackpad (`/api/auth/callback/<id>` voor de eerste vier, `/api/auth/oauth2/callback/<id>` voor de laatste twee), de omgevingsnamen inclusief de extra's (`APPLE_APP_BUNDLE_IDENTIFIER`, `GITLAB_ISSUER`, `OIDC_DISCOVERY_URL`), en een korte invulinstructie.
- Live data komt uit een nieuwe server-functie `getProviderSetup` in `src/lib/provider-setup.functions.ts`: `createServerFn({ method: "GET" })` met `requireAuth` en `assertAdminRole`, die `authDiagnostics()` uit `src/lib/better-auth.server.ts` en de registry samenvoegt. De UI rendert dus geen eigen URL-sommen; `callbackUrl` blijft afgeleid van `baseUrlFor(request)` zoals nu.
- Route-bestand volgt het bestaande patroon van `console.apps.new.tsx`: `createFileRoute("/_authenticated/console/providers")` met een `head()` en een pagina-component onder `src/pages/`. `src/routeTree.gen.ts` wordt niet aangeraakt.
- Zichtbaarheid in de zijbalk via de bestaande `src/hooks/useIsAdmin.ts`; de server-functie controleert de rol zelf opnieuw, dus een directe URL-aanroep van een niet-beheerder faalt.
- Kopieergedrag leunt op `navigator.clipboard.writeText`, dezelfde aanpak als de kopieerknop in `src/components/console/CodeBlock.tsx`.
- Afsluiten van `?diagnose=1` gebeurt in `src/routes/api_.public.auth.providers.ts`: die tak leest de sessie via de bestaande Better Auth-cookie en roept `assertAdminRole` aan; zonder geldende sessie of rol geeft de route alleen de publieke lijst terug. De tak zonder `diagnose` blijft publiek, want `src/pages/AuthNeon.tsx` leest alleen die lijst.
- Test: `src/lib/provider-setup.test.ts` vergelijkt elk pad uit de registry met wat `authDiagnostics()` werkelijk bouwt, zodat een toekomstige provider niet stil van de lijst verdwijnt. Bestaande suite blijft groen met `bunx vitest run`.
- Na het bouwen: visuele controle in de preview via Playwright (pagina openen, kopieerknoppen, statuslijst) en `build-errors.log` lezen.
