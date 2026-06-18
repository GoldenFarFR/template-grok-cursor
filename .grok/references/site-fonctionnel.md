# Blueprint — créer un site holding / vitrine qui fonctionne

> **Pour l'agent** : lire ce fichier **en entier** avant de créer un site. Ne pas improviser l'auth Privy.

**Références prod validées** :
- [ariavanguardzhc.com](https://ariavanguardzhc.com) → `GoldenFarFR/aria-vanguard`
- [harmony-8jni.onrender.com](https://harmony-8jni.onrender.com) → `GoldenFarFR/harmony` (test blueprint)

---

## Résumé en 30 secondes

1. Repo depuis `template-grok-cursor` → copier l'ossature `aria-vanguard` ou `harmony`
2. Renommer constantes (`SITE_NAME`, `TOKEN_KEY`, design)
3. **Une seule app Privy** pour tout l'écosystème — ajouter seulement l'URL du nouveau site dans Allowed origins
4. `setup-holding-render.ps1 -UpdateCors` → attendre 1–2 min → tester

---

## Architecture

```
Site statique (Vite + Privy)
    │  access_token + identity_token
    ▼
POST /api/auth/privy/login  (backend DEXPulse)
    │  token JWT membre
    ▼
localStorage + handoff ?aria_token= → DEXPulse
```

| Appel API | Headers |
|-----------|---------|
| Public (contenu, chat) | `X-Visitor-Id` |
| Membre | `Authorization: Bearer <token>` + `X-Visitor-Id` |

---

## Privy — une app pour tous les sites

**Ne pas créer une nouvelle app Privy par site.**

| Quoi | Action |
|------|--------|
| App Privy | **Réutiliser** celle d'Aria Vanguard (`site.config.json` → `privyAppId` ou `.env.example`) |
| Identity tokens | Déjà activé (Advanced → « Return user data in an identity token ») — **une fois pour l'app** |
| Backend JWT | Déjà dans `dexpulse-secrets` — **rien à refaire** |
| Par nouveau site | Ajouter l'URL dans **Privy Dashboard → Allowed origins** |

Origines à ajouter à chaque site :
```
https://<slug>.onrender.com
http://localhost:5175
```

---

## Procédure complète (ordre strict)

### Étape 1 — Repo GitHub

```powershell
cd projets
gh repo create <nom-du-site> --template GoldenFarFR/template-grok-cursor --public -c
```

Ou : GitHub → **Use this template** → créer le repo → cloner.

### Étape 2 — Scaffold code

Copier depuis `projets/harmony` (plus récent) ou `projets/aria-vanguard` :

| À renommer | Exemple Harmony |
|------------|-----------------|
| `SITE_NAME`, `SITE_TAGLINE` | `Harmony` |
| `SITE_DOMAIN` | `harmony-8jni.onrender.com` |
| `TOKEN_KEY` (`lib/auth.ts`) | `harmony_session_token` |
| Clés sessionStorage | préfixe `harmony:` |
| Événement session | `harmony:member-session` |
| Port dev (`vite.config.ts`) | `5175` (unique par site) |
| Design (`index.css`, composants) | libre |

**`package.json` — deps uniquement** (pas de wallet crypto, pas de blockchain) :

```json
{
  "dependencies": {
    "@privy-io/react-auth": "^3.31.0",
    "lucide-react": "^1.20.0",
    "react": "^19.2.6",
    "react-dom": "^19.2.6"
  },
  "devDependencies": {
    "@types/node": "^22.15.0",
    "@tailwindcss/vite": "^4.3.1",
    "@vitejs/plugin-react": "^6.0.1",
    "tailwindcss": "^4.3.1",
    "typescript": "~6.0.2",
    "vite": "^8.0.12"
  }
}
```

**Build Privy — copier depuis `harmony`** (obligatoire) :
- `src/shims/privy-vite-peer.ts` — stubs pour le build Vite (auth email/social seulement)
- `vite.config.ts` — alias vers ce stub (section `privyPeerAliases`)
- `tsconfig.node.json` → `"types": ["node"]`

Ne pas installer de packages crypto : les stubs suffisent.

```powershell
cd projets/<nom-du-site>
copy .env.example .env   # si pas encore créé
npm install
npm run build            # doit passer avant push
git add -A && git commit -m "feat: site <nom>" && git push
```

### Étape 3 — `.env.example` (dans le repo public)

```env
VITE_DEXPULSE_URL=https://dexpulse-m3bp.onrender.com
VITE_DEXPULSE_API_URL=https://dexpulse-m3bp.onrender.com/api
VITE_PRIVY_APP_ID=<meme-app-que-aria-vanguard>
```

Ne pas committer `.env` — seulement `.env.example`.

### Étape 4 — `render.yaml`

```yaml
services:
  - type: web
    name: <nom-du-site>
    runtime: static
    buildCommand: npm ci && npm run build
    staticPublishPath: dist
    envVars:
      - key: VITE_DEXPULSE_URL
        value: https://dexpulse-m3bp.onrender.com
      - key: VITE_DEXPULSE_API_URL
        value: https://dexpulse-m3bp.onrender.com/api
      - key: VITE_PRIVY_APP_ID
        sync: false
    routes:
      - type: rewrite
        source: /*
        destination: /index.html
```

### Étape 5 — Déploiement Render (automatisé)

Prérequis : `projets/dexpulse-secrets/.render-api-key` (voir README du repo secrets).

```powershell
cd projets/dexpulse-secrets
.\setup-holding-render.ps1 -SiteName <nom> -Repo GoldenFarFR/<nom> -Branch master -UpdateCors
```

Le script fait tout :
1. Crée ou resync le **static site** Render
2. Injecte `VITE_PRIVY_APP_ID` (copié depuis aria-vanguard Render)
3. Injecte `VITE_DEXPULSE_URL` + `VITE_DEXPULSE_API_URL`
4. Enregistre dans `site.config.json` → `holdingSites.<nom>`
5. Ajoute l'URL à `CORS_ORIGINS` + lance **`sync-render.ps1`** automatiquement

**Attendre 1–2 minutes** que le backend DEXPulse redéploie avant de tester l'auth.

Puis dans le repo du site :
```powershell
# Mettre SITE_DOMAIN = URL Render retournée par le script
# Ex: harmony-8jni.onrender.com
git add src/lib/site.ts && git commit -m "chore: SITE_DOMAIN prod" && git push
```

### Étape 6 — Privy Dashboard (manuel, 30 s)

[dashboard.privy.io](https://dashboard.privy.io) → app existante → **Allowed origins** → ajouter :
- `https://<slug>.onrender.com`
- `http://localhost:<port-dev>`

### Étape 7 — Validation prod

| Test | Attendu |
|------|---------|
| Page charge | HTTP 200, pas « VITE_PRIVY_APP_ID is not configured » |
| Contenu API | Sections chargent (`/api/aria/content/site`) |
| Sign in Privy | Modal s'ouvre, connexion OK |
| Activate access | Bannière bienvenue, pas d'erreur |
| DEXPulse | Lien avec `?aria_token=` → session reprise |
| Sign out | Token effacé |

Vérification CORS (si « Failed to fetch ») :
```powershell
curl.exe -s -D - "https://dexpulse-m3bp.onrender.com/api/auth/required" -H "Origin: https://<slug>.onrender.com" -o NUL
# Doit contenir : access-control-allow-origin: https://<slug>.onrender.com
```

---

## Arborescence `src/` (minimale)

```
src/
├── main.tsx              # PrivyProvider
├── App.tsx               # MemberGate + Page + MemberWelcome
├── api.ts                # fetch API (visitor + auth headers)
├── types.ts
├── index.css
├── vite-env.d.ts
├── pages/Site.tsx
├── components/
│   ├── Nav.tsx
│   ├── MemberSignInButton.tsx
│   ├── MemberGate.tsx
│   └── MemberWelcome.tsx
└── lib/
    ├── site.ts           # SITE_NAME, DEXPULSE_URL, DEXPULSE_API_URL
    ├── privy-config.ts   # loginMethods, accentColor
    ├── privy-session.ts  # UN seul échange, getIdentityToken async
    ├── auth.ts           # TOKEN_KEY unique
    ├── visitor.ts        # X-Visitor-Id (clé partagée écosystème OK)
    ├── member-profile.ts
    ├── dexpulse-handoff.ts  # ?aria_token=
    ├── cn.ts
    └── shims/
        └── privy-vite-peer.ts   # stubs build Privy (sans wallet crypto)
```

---

## Règles de code (non négociables)

### Auth Privy → backend

- **Un seul** `POST /auth/privy/login` par connexion (`exchangeInFlight`)
- Utiliser `getIdentityToken()` **async** (import `@privy-io/react-auth`)
- Retries après `refreshUser()` (3–4 ×, délai 400–800 ms)
- Cooldown 90 s si rate limit Privy
- **Ne pas** auto-échanger dans `MemberGate` — uniquement via bouton

### MemberSignInButton — 3 états

| État | Bouton |
|------|--------|
| Non connecté Privy | Sign in |
| Privy OK, pas session backend | Activate access |
| Session OK | Sign out |

### MemberGate

Site **toujours public**. Valide seulement un token existant au mount (`checkSession`).

### Handoff DEXPulse

Paramètre fixe écosystème : `aria_token` (ne pas renommer).

---

## Pièges connus

| Symptôme | Cause | Fix |
|----------|-------|-----|
| Identity token manquant | Identity tokens pas activés (nouvelle app seulement) | Advanced → activer (déjà fait sur l'app existante) |
| Failed to fetch après Privy OK | CORS backend pas redéployé | `-UpdateCors`, attendre 1–2 min, revérifier curl CORS |
| Too many requests | Double échange Privy | Un seul échange, pas de polling |
| Build Vite échoue (peer deps Privy) | Stubs manquants | Copier `src/shims/privy-vite-peer.ts` + alias `vite.config.ts` depuis harmony |
| Privy OK en local, KO en prod | URL absente des Allowed origins | Ajouter slug Render dans Privy Dashboard |
| Activer l'accès sans effet | Pas cliqué après connexion Privy | Normal — 2 étapes distinctes |

---

## Ce que l'agent ne doit PAS faire

- Créer une nouvelle app Privy par site
- Modifier `PRIVY_JWT_VERIFICATION_KEY` backend pour un nouveau site holding
- Bloquer le site public dans `MemberGate`
- Déclencher l'échange Privy automatiquement au chargement
- Oublier `-UpdateCors` au deploy Render
- Tester l'auth immédiatement après sync CORS (attendre redeploy backend)

---

## Références

| Ressource | Chemin |
|-----------|--------|
| Site holding principal | `projets/aria-vanguard` |
| Site test blueprint | `projets/harmony` |
| Deploy Render | `projets/dexpulse-secrets/setup-holding-render.ps1` |
| Config services | `projets/dexpulse-secrets/site.config.json` → `holdingSites` |
| Backend auth | `projets/dexpulse/backend/app/api/routes/auth.py` |
| Handoff produit | `projets/dexpulse/frontend/src/lib/session-handoff.ts` |
| Privy identity tokens | [docs.privy.io](https://docs.privy.io/user-management/users/identity-tokens) |

---

## Instruction agent (une ligne)

**Nouveau site** = template → copier `harmony`/`aria-vanguard` → renommer → `npm run build` → `setup-holding-render.ps1 -UpdateCors` → attendre 2 min → Privy Allowed origins → valider les 5 tests.