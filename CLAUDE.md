@AGENTS.md
@DESIGN.md

# Oktobus marketing-site

3-koppig AI-software bureau (Jochem Michiels, David, Max). Marketing-site met drie pagina's: `/`, `/belofte`, `/werkwijze`. Pitch-klaar.

## Stack
- Next.js 16 (App Router) + React 19 + TypeScript strict
- Tailwind v4 — alle tokens via `@theme` in `app/globals.css`. Géén `tailwind.config.ts`.
- `motion` (react animaties), `lenis` (smooth scroll), `clsx` + `tailwind-merge` (className-utils), `lucide-react` (generic icons)
- `zod` voor env-validatie in `lib/env.ts`
- Live op https://oktobus.nl (eigen VPS via shared Docker-proxy, Let's Encrypt)

## Naamgeving
- Brand: **Oktobus** (met K). Altijd zo schrijven in code, metadata, page titles, copy.
- Logo-asset bestand heet `octobus_v2.svg` (met C) — alleen file-naam, geen tekst-content.

## Design system
DESIGN.md is de single source of truth. Lees vóór UI-werk:
- §2 Tokens (kleuren, type-scale)
- §5 Animatie-regels (sway/breathe = CSS loop, rest = Motion `whileInView`)
- §6 Component-regels (buttons, cards, eyebrows)

## Component-bibliotheek
- `components/ui/` — Eyebrow, Display, Body, Num, Hairline, Button
- `components/marks/` — OctopusLogo, LogoTile
- `components/motion/` — RiseIn, FadeIn (gebruiken `useReducedMotion`)
- `components/layout/` — Container, Section, Header, Footer, SmoothScroll
- `components/sections/` — PageHero, PillarCard, CTABlock

## Conventies
- Server Components default. `'use client'` alleen waar interactieve state nodig is (Header, motion-wrappers, SmoothScroll).
- Internal links → Next `<Link>`. External / mailto → `<a>`.
- ClassNames via `cn()` uit `lib/cn.ts`.
- Geen italics, geen box-shadows, geen gradients (DESIGN.md §8).

## Commands
- `pnpm dev` — local dev (hot reload)
- `pnpm build && pnpm start` — productie-build lokaal valideren
- `pnpm typecheck` — TS check
- `pnpm lint` — ESLint

## Deploy-flow (alles via GitHub)

Productie draait op `https://oktobus.nl`, host = VPS `77.42.82.154`, repo-pad = `/opt/oktobus` (git-clone, beheerd door GitHub Actions). GitHub-repo: `Oktobus-tech/oktobus` (org). Verhuisd vanaf `Davidemeer/oktobus` op 2026-05-29 — oude URL werkt nog via GitHub-redirect.

**Werkwijze voor élke wijziging — code, copy, infra:**

1. Branch maken vanaf `main`: `git checkout -b feat/korte-omschrijving`
2. **Lokale dev-server starten en wijziging in de browser testen:**
   - `pnpm install` (eerste keer of na lockfile-wijzigingen)
   - `pnpm dev` — opent op `http://localhost:3000` (of een ander vrij poortnummer als 3000 bezet is — kijk naar de output)
   - Hot reload: bestand opslaan = browser ververst automatisch
   - Test ook op `/`, `/over-ons`, `/werkwijze` — niet alleen de pagina waar je werkt
3. Lokaal valideren vóór commit: `pnpm typecheck && pnpm lint`
4. Committen + pushen: `git push -u origin feat/korte-omschrijving`
5. PR openen tegen `main` (`gh pr create --base main`)
6. CI draait automatisch (`.github/workflows/ci.yml`): `pnpm typecheck`, `pnpm lint`, `docker build`. Rood = niet mergen.
7. Reviewen + mergen via GitHub UI of `gh pr merge`
8. `deploy.yml` triggert na CI-success op `main`: SSH naar VPS, `git fetch && reset --hard origin/main`, `docker compose build`, `up -d`, health check op `http://app:3000/`. Bij failure: automatische rollback naar vorige SHA.

**Belangrijk — lokaal eerst:** de live site is niet je sandbox. Geen "even snel pushen om te kijken hoe het eruitziet". Alles eerst op `localhost` zien werken, dán PR. Een mislukte deploy rolt automatisch terug, maar verspilt CI-tijd en zet een gefaalde run op de geschiedenis.

**Niet doen:**
- ❌ Niet rechtstreeks naar `main` pushen (gaat via PR)
- ❌ Niet handmatig SSH'en naar `/opt/oktobus` om files te wijzigen — de volgende deploy reset de tree (`git reset --hard origin/main`) en je werk is weg
- ❌ Niet `rsync` / `scp` gebruiken om snel iets te updaten — zelfde reden
- ❌ Geen secrets / `.env*` committen (`.env*` staat in `.gitignore`, behalve `.env.example`)

**Wel doen:**
- ✅ Lokaal testen met `pnpm dev` voor je een PR opent
- ✅ `pnpm typecheck && pnpm lint` lokaal voor je commit (matcht CI)
- ✅ Voor handmatige re-deploy zonder code-change: GitHub → Actions → "Deploy" → Run workflow, of `gh workflow run Deploy --repo Oktobus-tech/oktobus`

**Server-side state (alleen als het echt nodig is):**
- Server-changes (nginx, fail2ban, Docker proxy): `/opt/proxy/` — niet in deze repo, alleen via SSH op de VPS aanpassen. Documenteer wat je doet.
- `.env.production` op de VPS: bewerken via SSH; staat buiten git. Bij wijziging: `docker compose up -d` om door te zetten.
- VPS-toegang: vraag Jochem om je SSH-pubkey toe te voegen aan `/root/.ssh/authorized_keys`. Voor reguliere feature-werk niet nodig.

**Deploy-config** (op `Oktobus-tech/oktobus`) — al ingesteld, niet aan komen:
- Repo-**variables**: `DEPLOY_HOST` (`77.42.82.154`), `DEPLOY_USER` (`root`) — gebruikt als `${{ vars.* }}` in `deploy.yml`.
- Repo-**secret**: `DEPLOY_SSH_KEY` — gebruikt als `${{ secrets.DEPLOY_SSH_KEY }}`.

**Andere sites op dezelfde VPS:** `hartconstructions.nl`, `nieuwsradar.online`. Eigen repo's, eigen deploy-flows. Niet kruisen.
