# template-grok-cursor

Template GitHub pour démarrer un projet avec **Grok Build** et **Cursor**, règles en français incluses.

## Contenu

```
.
├── .cursor/rules/       # Règles Cursor (langue, conventions)
├── .grok/               # Config et instructions Grok
│   ├── AGENTS.md
│   └── config.toml.example
├── AGENTS.md            # Instructions racine (lus par Grok)
└── README.md
```

## Utilisation

### Nouveau projet depuis ce template

1. Sur GitHub : **Use this template** → **Create a new repository**
2. Cloner le nouveau dépôt
3. Ouvrir dans Cursor et lancer Grok

### Configuration globale (une fois par machine)

Copier manuellement sur chaque PC :

| Fichier source | Destination |
|---|---|
| `.cursor/rules/francais.md` | `~/.cursor/rules/francais.md` |
| `.grok/AGENTS.md` | `~/.grok/AGENTS.md` |
| `.grok/config.toml.example` | `~/.grok/config.toml` (éditer les tokens) |

### Personnaliser

- Ajoutez des règles dans `.cursor/rules/`
- Éditez `AGENTS.md` et `.grok/AGENTS.md` pour vos conventions de projet
- Dupliquez `config.toml.example` en `config.toml` locale (gitignored)

## Licence

Libre d'utilisation — adaptez ce template à vos besoins.