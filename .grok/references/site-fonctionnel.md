# Blueprint — site holding / vitrine qui fonctionne

> **Pour l'agent** : lire ce fichier **en entier** avant de créer ou modifier un site vitrine connecté au backend DEXPulse. Référence validée en prod : `GoldenFarFR/aria-vanguard` → [ariavanguardzhc.com](https://ariavanguardzhc.com).

## Quand utiliser ce blueprint

- Nouveau site **statique** (holding, filiale, landing) qui consomme l'API DEXPulse
- Auth membre via **Privy** (email, X, Discord) — pas le vieux flux code Telegram
- Déploiement **Render static site** + domaine custom
- Handoff session vers une app produit sur un autre domaine (ex. DEXPulse)

Ne pas réinventer l'auth : copier la structure, renommer les constantes, adapter le design.

---

## Architecture (vue d'ensemble)

```
┌─────────────────────┐     access_token + identity_token      ┌──────────────────────┐
│  Site statique      │ ─────────────────────────────────────► │  DEXPulse backend    │
│  (Vite + React)     │ ◄── aria_session_token (JWT) ─────── │  POST /auth/privy/   │
│  PrivyProvider      │                                      │  login               │
└─────────────────────┘                                      └──────────────────────┘
         │                                                              ▲
         │  APIs publiques : header X-Visitor-Id                        │
         │  APIs membre   : Authorization: Bearer <token>              │
         └──────────────────────────────────────────────────────────────┘

Handoff cross-origin (site → produit) :
  https://produit.example.com?aria_token=<session_token>
  → importé côté produit, paramètre retiré de l'URL
```

### Deux jetons Privy (obligatoire)

| Jeton | Rôle |
|-------|------|
| `access_token` | Prouve que l'utilisateur est authentifié |
| `identity_token` | Contient `linked_accounts` (X, email…) — **nécessaire** pour créer la session backend |

**Dashboard Privy (à faire à chaque nouvelle app)** :
1. [dashboard.privy.io](https://dashboard.privy.io) → ton app
2. **User management → Authentication → Advanced**
3. Activer **« Return user data in an identity token »**
4. Sauvegarder, attendre ~1 min

Sans cette case : Privy connecte l'utilisateur mais le site affiche « Identity token manquant ».

---

## Stack technique

| Couche | Choix |
|--------|-------|
| Build | Vite 8 + TypeScript |
| UI | React 19 + Tailwind CSS 4 (`@tailwindcss/vite`) |
| Auth | `@privy-io/react-auth` ^3.31 |
| Icônes | `lucide-react` |
| Hébergement | Render **static site** (`dist/`) |
| API | Backend DEXPulse existant (`VITE_*_API_URL`) |

---

## Arborescence minimale (`src/`)

```
src/
├── main.tsx                 # PrivyProvider + mount React
├── App.tsx                  # MemberGate + pages + MemberWelcome
├── api.ts                   # fetch vers backend (visitor + auth headers)
├── types.ts                 # types API / contenu
├── index.css                # tokens design (couleurs, typo)
├── vite-env.d.ts            # VITE_* env types
├── pages/
│   └── Site.tsx             # page principale (contenu, sections)
├── components/
│   ├── Nav.tsx              # nav + MemberSignInButton + lien produit
│   ├── MemberSignInButton.tsx   # Sign in / Activer l'accès / Sign out
│   ├── MemberGate.tsx       # valide session existante (ne bloque pas le public)
│   └── MemberWelcome.tsx    # bannière post-login (optionnel)
└── lib/
    ├── site.ts              # URLs, noms, constantes du site
    ├── privy-config.ts      # PRIVY_APP_ID, loginMethods, appearance
    ├── privy-session.ts     # exchangePrivyForAriaSession (UN seul échange)
    ├── auth.ts              # localStorage token + authHeaders()
    ├── visitor.ts           # X-Visitor-Id persistant
    ├── member-profile.ts    # profil membre sessionStorage + welcome
    ├── dexpulse-handoff.ts  # URL produit + ?aria_token=
    └── cn.ts                # utilitaire classes (optionnel)
```

Fichiers racine obligatoires : `package.json`, `vite.config.ts`, `tsconfig.json`, `index.html`, `render.yaml`.

---

## Variables d'environnement

### Frontend (préfixe `VITE_` — build-time)

| Variable | Exemple | Rôle |
|----------|---------|------|
| `VITE_PRIVY_APP_ID` | `cmqi…` | App Privy (secret côté Render, pas dans le repo public) |
| `VITE_DEXPULSE_URL` | `https://dexpulse-m3bp.onrender.com` | URL produit (liens, handoff) |
| `VITE_DEXPULSE_API_URL` | `https://…/api` | Base API backend |

### Backend (déjà dans `dexpulse-secrets/production.env`)

| Variable | Rôle |
|----------|------|
| `PRIVY_APP_ID` | Même app que le frontend |
| `PRIVY_JWT_VERIFICATION_KEY` | Clé PEM (ou JWKS auto si vide) |
| `ACCESS_CODE_ENABLED=true` | Active l'auth membre |

---

## Patterns de code (à reproduire)

### 1. `lib/site.ts` — constantes du site

```ts
export const SITE_DOMAIN = 'example.com'
export const SITE_URL = `https://${SITE_DOMAIN}`
export const SITE_NAME = 'Mon Site'

export const PRODUCT_URL = import.meta.env.VITE_PRODUCT_URL ?? 'https://…'
export const API_URL = import.meta.env.VITE_API_URL ?? `${PRODUCT_URL}/api`
```

### 2. `lib/auth.ts` — session backend

```ts
const TOKEN_KEY = 'mon_site_session_token'  // unique par site

export function getToken() { return localStorage.getItem(TOKEN_KEY) }
export function setToken(t: string) { localStorage.setItem(TOKEN_KEY, t) }
export function clearToken() { localStorage.removeItem(TOKEN_KEY) }
export function authHeaders(): HeadersInit {
  const t = getToken()
  return t ? { Authorization: `Bearer ${t}` } : {}
}
```

### 3. `lib/visitor.ts` — visiteur anonyme

Toujours envoyer `X-Visitor-Id` sur les appels API publics (chat, contenu, analytics).

### 4. `api.ts` — client HTTP

```ts
async function apiFetch(path: string, init?: RequestInit) {
  const headers = { ...visitorHeaders(), ...authHeaders(), ...init?.headers }
  return fetch(`${API_URL}${path}`, { ...init, headers })
}
```

Endpoints auth utilisés :
- `GET /auth/required` — le site est-il en mode membre ?
- `GET /auth/session` — token backend encore valide ?
- `POST /auth/privy/login` — body `{ access_token, identity_token }` → `{ token, twitter_username, message }`

### 5. `lib/privy-session.ts` — échange unique (critique)

Règles apprises en prod :
- **Un seul** `POST /auth/privy/login` par connexion (dédupliquer avec `exchangeInFlight`)
- Utiliser `getIdentityToken()` **async** (pas seulement le hook sync)
- Après `refreshUser()`, retenter 3–4 fois avec délai
- Cooldown sessionStorage si rate limit Privy (90 s)
- Ne **pas** auto-déclencher l'échange dans `MemberGate` — seulement via le bouton

### 6. `MemberSignInButton.tsx` — UX des 3 états

| État | Bouton |
|------|--------|
| Non connecté Privy | **Sign in** → ouvre modal Privy |
| Privy OK, pas de session backend | **Activer l'accès** → `exchangePrivyForAriaSession` |
| Session backend OK | **Sign out** |

`useLogin({ onComplete: finishSignIn })` pour le flux modal. Si déjà `authenticated`, le clic appelle directement `finishSignIn`.

### 7. `MemberGate.tsx` — site public

- **Ne jamais** bloquer la page pour les visiteurs
- Au mount : si token local existe → `checkSession()` ; invalide → `clearToken()`
- L'échange Privy reste dans `MemberSignInButton`

### 8. Handoff cross-origin

**Site holding** (`lib/product-handoff.ts`) :
```ts
export function productHandoffUrl(): string {
  const token = getToken()
  if (!token) return PRODUCT_URL
  const url = new URL(PRODUCT_URL)
  url.searchParams.set('aria_token', token)  // nom du param fixe écosystème
  return url.toString()
}
```

**App produit** (`session-handoff.ts`) — appeler au boot :
```ts
export function importVanguardSession(): void {
  const token = new URLSearchParams(location.search).get('aria_token')
  if (!token || token.length < 16) return
  setToken(token)
  // retirer aria_token de l'URL (replaceState)
}
```

### 9. Événement custom session

Après login/logout : `window.dispatchEvent(new Event('aria:member-session'))`  
→ `MemberWelcome` et la nav se mettent à jour sans reload.

---

## `main.tsx` — PrivyProvider

```tsx
import { PrivyProvider } from '@privy-io/react-auth'
import { PRIVY_APP_ID, privyProviderConfig } from './lib/privy-config'

if (!PRIVY_APP_ID) {
  // afficher erreur config — ne pas monter Privy sans app ID
}
root.render(
  <PrivyProvider appId={PRIVY_APP_ID} config={privyProviderConfig}>
    <App />
  </PrivyProvider>
)
```

`privy-config.ts` : `loginMethods: ['email', 'twitter', 'discord']`, `embeddedWallets.ethereum.createOnLogin: 'off'`.

---

## `render.yaml` — static site

```yaml
services:
  - type: web
    name: mon-site
    runtime: static
    buildCommand: npm ci && npm run build
    staticPublishPath: dist
    envVars:
      - key: VITE_PRODUCT_URL
        value: https://…
      - key: VITE_API_URL
        value: https://…/api
      - key: VITE_PRIVY_APP_ID
        sync: false   # injecté via secrets, pas dans le YAML public
    routes:
      - type: rewrite
        source: /*
        destination: /index.html
```

---

## Checklist — nouveau site (ordre strict)

### Phase 1 — Repo & scaffold
- [ ] GitHub → **Use this template** (`template-grok-cursor`) → nouveau repo
- [ ] Cloner, `npm create vite@latest` ou copier structure depuis `aria-vanguard`
- [ ] Renommer constantes (`TOKEN_KEY`, `SITE_NAME`, domaines)
- [ ] `npm run build` passe en local

### Phase 2 — Privy
- [ ] Créer app Privy (ou réutiliser) → noter `PRIVY_APP_ID`
- [ ] Activer login methods (email, twitter, discord)
- [ ] **Activer identity tokens** (Advanced) ← souvent oublié
- [ ] Ajouter domaines autorisés : `localhost`, domaine prod
- [ ] Configurer `VITE_PRIVY_APP_ID` sur Render

### Phase 3 — Backend
- [ ] `PRIVY_APP_ID` + `PRIVY_JWT_VERIFICATION_KEY` dans `dexpulse-secrets/production.env`
- [ ] `.\sync-all.ps1` vers Render
- [ ] Tester `POST /api/auth/privy/login` (401 sans token, pas 503)

### Phase 4 — Déploiement Render (automatisé)

Depuis `projets/dexpulse-secrets` (clé `.render-api-key` requise) :

```powershell
cd projets/dexpulse-secrets
.\setup-holding-render.ps1 -SiteName <nom-du-site> -Repo GoldenFarFR/<nom-du-site> -Branch master -UpdateCors
.\sync-render.ps1
```

Le script :
1. Crée le **static site** Render (ou resync si déjà existant)
2. Injecte `VITE_DEXPULSE_URL`, `VITE_DEXPULSE_API_URL`, `VITE_PRIVY_APP_ID` (copié depuis aria-vanguard)
3. Enregistre le service dans `site.config.json` → `holdingSites.<nom>`
4. Avec `-UpdateCors` : ajoute l’URL Render à `CORS_ORIGINS` → `sync-render.ps1` pousse vers le backend

Ensuite dans le repo du site :
- [ ] Mettre à jour `src/lib/site.ts` (`SITE_DOMAIN` = slug Render ou domaine custom)
- [ ] Commit + push → Render redéploie automatiquement

**Privy Dashboard** (manuel, une fois par URL) :
- [ ] Allowed origins → ajouter `https://<slug>.onrender.com` et `http://localhost:5175`

Option manuelle : Render Dashboard → New Static Site → repo GitHub → `render.yaml`.

### Phase 5 — Validation prod
- [ ] Visiteur anonyme : contenu + chat OK (`X-Visitor-Id`)
- [ ] Sign in Privy → **Activer l'accès** → bannière bienvenue
- [ ] Lien produit avec handoff → session reprise sur DEXPulse
- [ ] Sign out → token effacé, retour visiteur
- [ ] Ctrl+F5 : session persiste si token encore valide

---

## Pièges connus (ne pas refaire)

| Symptôme | Cause | Fix |
|----------|-------|-----|
| « Identity token manquant » | Case Privy Advanced non cochée | Activer identity tokens dashboard |
| « Too many requests » / rate limit | Double échange Privy + polling | Un seul échange, dédup `exchangeInFlight`, pas d'auto-exchange dans Gate |
| « Please wait a few seconds » | Cooldown Privy / spam clic | Attendre 90 s, Ctrl+F5, un seul clic |
| Privy OK mais pas membre | Pas cliqué « Activer l'accès » | Normal — session backend ≠ session Privy |
| Handoff DEXPulse échoue | `importVanguardSession()` pas appelé au boot produit | Appeler dans `main.tsx` du frontend produit |
| 401 sur `/auth/privy/login` | `PRIVY_JWT_VERIFICATION_KEY` incorrect | Resync secrets Render |
| **« Failed to fetch »** après Privy OK | CORS backend pas encore redéployé avec l’URL du nouveau site | `setup-holding-render.ps1 -UpdateCors` puis attendre **1–2 min** ; vérifier header `access-control-allow-origin` sur `/api/auth/required` |

---

## Personnalisation par site

| Élément | À changer |
|---------|-----------|
| Design | `index.css`, composants pages, tokens couleur |
| Contenu | Endpoints `/aria/content/*` ou pages statiques |
| Nom token localStorage | `TOKEN_KEY` unique (évite conflit multi-sites même origine) |
| Param handoff | Garder `aria_token` pour compatibilité écosystème |
| Langue UI | Holding = anglais public ; messages agent = selon `VISION.md` |

---

## Références vivantes

| Ressource | Chemin / URL |
|-----------|----------------|
| Site prod validé | `projets/aria-vanguard` — [ariavanguardzhc.com](https://ariavanguardzhc.com) |
| Backend auth | `projets/dexpulse/backend/app/api/routes/auth.py` |
| Vérif JWT Privy | `projets/dexpulse/backend/app/auth/privy_verify.py` |
| Handoff produit | `projets/dexpulse/frontend/src/lib/session-handoff.ts` |
| Secrets Render | `projets/dexpulse-secrets/` + `sync-all.ps1` |
| Deploy static site | `dexpulse-secrets/setup-holding-render.ps1` |
| Exemple holdingSites | `site.config.json` → `holdingSites.harmony` |
| Vision produit | `projets/dexpulse/VISION.md` |
| Doc Privy identity tokens | [docs.privy.io — Identity tokens](https://docs.privy.io/user-management/users/identity-tokens) |

---

## Instruction agent (résumé une ligne)

**Nouveau site** = copier l'ossature `aria-vanguard`, renommer les constantes, configurer Privy (identity tokens !), brancher `VITE_*` sur Render, valider les 5 tests prod de la checklist.