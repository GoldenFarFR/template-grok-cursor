# template-grok-cursor

Template GitHub pour démarrer chaque nouveau projet **Aria-ready** avec **Grok Build** et **Cursor** — réponses toujours en **français**.

## Règles incluses (always-on)

> Les anciens fichiers séparés (`.cursor/rules/francais.md`, `vision.md`, `conventions.md`,
> `skills-on-demand.md`, `.grok/rules/vision-enforcer.md`) ont été archivés hors repo et
> consolidés (commit `afd866c`, 2026-07-05) dans les deux fichiers suivants :

| Fichier | Rôle |
|---------|------|
| `.cursor/rules/regles-uniques.mdc` | Règle unique Cursor (`alwaysApply: true`) : langue, workflow proposer/valider, mémoire, journal, runbook, format technique par outil |
| `AGENTS.md` (racine) | Instructions Grok Build : langue, vision Aria, skills à la demande, conventions |

## Contenu

```
.
├── .cursor/rules/
│   └── regles-uniques.mdc         # Règle unique Cursor (alwaysApply)
├── .grok/
│   ├── AGENTS.md                  # Copie globale (voir "Configuration globale IDE")
│   ├── references/
│   │   ├── site-fonctionnel.md    # Blueprint site holding lambda
│   │   ├── ecosystem-repos.md     # Carte GoldenFar 2026
│   │   ├── aria-backend-patterns.md
│   │   ├── skill-authoring.md     # Comment créer une skill métier
│   │   └── scaffold/              # Site Vite/React + Privy à copier
│   └── skills/
│       ├── install-skill/         # Audit avant ajout skill au template
│       └── spawn-holding-site/    # Création site holding (à la demande)
├── scripts/
│   └── link-aria-skills.ps1       # Lier aria-skills sur le PC
├── VISION.md               # Stub → ARIA/VISION.md
├── AGENTS.md
└── README.md
```

## Écosystème GoldenFar / Aria (2026)

**Vision SSOT :** [ARIA/VISION.md](https://github.com/GoldenFarFR/ARIA/blob/main/VISION.md)  
**Carte repos :** [ARIA/vanguard/docs/ECOSYSTEM-REPOS.md](https://github.com/GoldenFarFR/ARIA/blob/main/vanguard/docs/ECOSYSTEM-REPOS.md)  
**Référence locale :** [`.grok/references/ecosystem-repos.md`](.grok/references/ecosystem-repos.md)

| Repo | Rôle |
|------|------|
| `ARIA` (public) | Monorepo — holding + API (`vanguard/`) + cœur agent (`packages/aria-core/`). Remplace les anciens `aria-vanguard` et `aria-sandbox`, fusionnés dedans. |
| `aria-ops` (privé) | Infra/ops : scripts déploiement, coffre secrets, mémoire opérateur (`vanguard/operator/`, `local-sync/`) |
| `template-grok-cursor` | Ce template |
| `aria-skills`, `aria-token-base` | Mentionnés dans d'anciens docs — statut non vérifié cette session |
| ~~`dexpulse`~~, ~~`collegue-memoire`~~, ~~`aria-local-sync`~~ | *Dépréciés* — fusionnés dans `ARIA` / `aria-ops` |

Secrets : coffre `%LOCALAPPDATA%\GoldenFar\vault` — **jamais** dans le repo.

## Skills du template

| Skill | Invocation | Rôle |
|-------|------------|------|
| `spawn-holding-site` | `/spawn-holding-site <nom>` | Site lambda Privy + Render |
| `install-skill` | `/install-skill` | Auditer une skill avant ajout au template |

Toutes ont `disable-model-invocation: true` — pas de chargement automatique. Voir `AGENTS.md`.

**Skills métier** (marketing, vision, deploy, mini-jeux) : repo `aria-skills` (statut non vérifié cette session) — ne pas les dupliquer ici.

## Nouveau projet (à chaque fois)

1. GitHub → [template-grok-cursor](https://github.com/GoldenFarFR/template-grok-cursor) → **Use this template** → **Create a new repository**
2. Cloner le nouveau dépôt dans `%USERPROFILE%\projets\`
3. Ouvrir dans Cursor — règles FR + vision déjà actives
4. Site holding ? → `/spawn-holding-site <nom>` (lit `site-fonctionnel.md` + scaffold)

## Setup PC opérateur (une fois)

```powershell
# Skills métier Aria (vision-enforcer, marketing, operator-runbook, …)
cd $env:USERPROFILE\projets\template-grok-cursor
.\scripts\link-aria-skills.ps1

# Écosystème complet (optionnel)
cd $env:USERPROFILE\projets\aria-ops\vanguard\operator
.\new-pc.ps1
```

## Configuration globale IDE (optionnel)

| Fichier template | Copier vers |
|------------------|-------------|
| `.cursor/rules/regles-uniques.mdc` | `~/.cursor/rules/regles-uniques.mdc` |
| `.grok/AGENTS.md` | `~/.grok/AGENTS.md` |

## Blueprint site lambda

[`.grok/references/site-fonctionnel.md`](.grok/references/site-fonctionnel.md) — accueil + Privy + Render.  
Exemples enrichis : `ARIA` (vitrine `vanguard/`, anciennement `aria-vanguard`), `kikou`.

Deploy : `aria-ops/vanguard/operator/setup-holding-render.ps1 -SiteName <nom> -UpdateCors`

## Ajouter un skill au template

1. Envoyer la source à Grok dans ce dépôt
2. Lancer `/install-skill` (audit sécurité, doublons, tests)
3. Checklist : `.grok/skills/install-skill/references/checklist.md`

## Licence

Libre d'utilisation — adaptez ce template à vos besoins.