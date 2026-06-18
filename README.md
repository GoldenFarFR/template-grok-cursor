# template-grok-cursor

Template GitHub pour démarrer chaque nouveau projet avec **Grok Build** et **Cursor** — **réponses toujours en français**.

## Règle principale (incluse automatiquement)

Fichier : `.cursor/rules/francais.md`

```markdown
Toujours répondre en français.
```

À chaque nouveau projet créé depuis ce template, cette règle est **copiée dans le dépôt** et chargée automatiquement par Cursor et Grok. Aucune action manuelle requise par projet.

## Contenu

```
.
├── .cursor/rules/
│   ├── francais.md      # ← règle langue (alwaysApply: true)
│   └── conventions.md
├── .grok/
│   ├── AGENTS.md        # même règle côté Grok
│   ├── config.toml.example
│   ├── references/
│   │   ├── site-fonctionnel.md   # ← blueprint site lambda (accueil + Privy)
│   │   └── scaffold/             # site Vite/React minimal à copier vers chaque nouveau repo
│   └── skills/
│       └── install-skill/   # procédure d'audit + installation de skills
├── AGENTS.md
└── README.md
```

## Blueprint site lambda

Fichier : [`.grok/references/site-fonctionnel.md`](.grok/references/site-fonctionnel.md)

À chaque **nouveau repo** : livrer un site **minimal** (accueil statique + Privy + deploy Render). Copier [`.grok/references/scaffold/`](.grok/references/scaffold/) et renommer les constantes. Le contenu riche (sections, API, DEXPulse) s'ajoute **plus tard** quand l'utilisateur alimente le site.

Procédure : repo → scaffold → `npm run build` → `setup-holding-render.ps1 -UpdateCors` → Privy origins → test auth.

Exemples enrichis (phase 2) : `aria-vanguard`, `harmony`.

## Skills inclus

| Skill | Rôle |
|-------|------|
| `install-skill` | Auditer, installer et valider un nouveau skill avant ajout au template |

### Ajouter un skill au template

1. Envoyer le lien ou la source à Grok dans ce dépôt
2. Lancer `/install-skill` — la procédure couvre :
   - audit sécurité (scripts, commandes, secrets)
   - détection de doublons (template + skills globaux + bundled Grok)
   - qualité et efficacité du SKILL.md
   - test via `grok inspect` et invocation `/nom`
   - mise à jour du README et commit Git
3. Ne jamais copier un skill internet sans passer par cette procédure

Checklist imprimable : `.grok/skills/install-skill/references/checklist.md`

## Nouveau projet (à chaque fois)

1. GitHub → [template-grok-cursor](https://github.com/GoldenFarFR/template-grok-cursor) → **Use this template** → **Create a new repository**
2. Cloner le nouveau dépôt
3. Ouvrir dans Cursor — la règle française est déjà active

## Configuration globale (une seule fois par PC)

Optionnel, pour avoir le français **hors projet** (dossier home, sessions sans repo) :

| Fichier dans le template | Copier vers |
|---|---|
| `.cursor/rules/francais.md` | `~/.cursor/rules/francais.md` |
| `.grok/AGENTS.md` | `~/.grok/AGENTS.md` |

## Licence

Libre d'utilisation — adaptez ce template à vos besoins.