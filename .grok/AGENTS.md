# Instructions du projet

Toujours répondre en français.

## Créer un nouveau site (holding / vitrine)

Avant tout scaffold ou feature site statique connecté à DEXPulse, lire **en entier** :

`references/site-fonctionnel.md`

**Livrer un site lambda** : page d'accueil statique + connexion Privy + deploy Render. **Ne pas enrichir** (sections, API contenu, DEXPulse handoff) sauf demande explicite.

Copier `.grok/references/scaffold/` vers le nouveau repo, renommer les constantes (`SITE_NAME`, `TOKEN_KEY`, port dev), puis build → `setup-holding-render.ps1 -UpdateCors` → Privy origins.

**`-UpdateCors` obligatoire** : sync CORS, redéploiement forcé du backend DEXPulse, vérification auto (évite `Failed to fetch`).

Références enrichies (phase 2 uniquement) : `GoldenFarFR/aria-vanguard`, `GoldenFarFR/lucky`.

## Conventions

- Modifier uniquement le code requis par la tâche.
- Respecter le style, les types et les abstractions existants.
- Réutiliser les fonctions et composants déjà présents plutôt que de réimplémenter.