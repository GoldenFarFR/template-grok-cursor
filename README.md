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
│   └── config.toml.example
├── AGENTS.md
└── README.md
```

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