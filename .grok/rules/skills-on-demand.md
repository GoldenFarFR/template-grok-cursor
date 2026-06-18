---
description: Chargement des skills — à la demande uniquement
alwaysApply: true
---

# Skills — à la demande uniquement

## Règle par défaut

- **Ne charger aucune skill** au démarrage ni « par précaution ».
- **Ne pas** lire plusieurs skills en parallèle sauf demande explicite de l'utilisateur.
- Les skills du repo restent **disponibles** via `/nom-skill` ou quand la demande correspond **clairement** à une seule skill.

## Quand charger une skill

| Demande utilisateur | Action |
|---------------------|--------|
| « site web », « landing page », app générique | **Aucune skill** — coder normalement |
| « site holding », « comme Kikou », « lambda Privy », `/spawn-holding-site` | Lire **uniquement** `.grok/skills/spawn-holding-site/SKILL.md` |
| `/nom-skill` ou « utilise la skill X » | Lire **uniquement** la skill nommée |
| « app paris sportifs » (quand la skill existe) | Lire **uniquement** `.grok/skills/paris-sportifs/SKILL.md` |
| « mini-jeux », « jeux sur l'accueil », `/graft-mini-games` | Lire **uniquement** `.grok/skills/graft-mini-games/SKILL.md` |

## Quand ne PAS charger

- Site web simple sans mention holding / Privy / DEXPulse / nom de skill
- Tâche générique (fix, refactor, question)
- L'utilisateur n'a pas demandé de workflow métier spécifique

## Interdit

- Charger `spawn-holding-site` pour un site web ordinaire
- Fusionner plusieurs workflows (holding + paris sportifs + autre) sans demande
- Lire `site-fonctionnel.md` sauf via la skill `spawn-holding-site` ou demande holding explicite