# Écosystème GoldenFar — où mettre quoi

**Vision SSOT :** [dexpulse/VISION.md](https://github.com/GoldenFarFR/dexpulse/blob/master/VISION.md)  
**Carte complète :** [dexpulse/docs/ECOSYSTEM-REPOS.md](https://github.com/GoldenFarFR/dexpulse/blob/master/docs/ECOSYSTEM-REPOS.md)

## Nouveau repo depuis ce template

1. **Use this template** → nouveau repo sous `GoldenFarFR/`
2. Copier `references/scaffold/` si site web
3. Lier `aria-skills` : `aria-skills/scripts/install.ps1`
4. Lire `VISION.md` (dexpulse) avant toute feature
5. Secrets → jamais dans le repo ; `dexpulse-secrets` si deploy Render

## Repos de référence

| Repo | Rôle |
|------|------|
| dexpulse | Produit + ARIA runtime |
| aria-vanguard | Site holding |
| aria-skills | Skills Grok (SSOT) |
| aria-sandbox | Expériences |
| aria-token-base | Token R&D |
| template-grok-cursor | Ce template |
| dexpulse-secrets | Clés (exclu API Aria) |

## Patterns copiables (dexpulse)

- Skill backend : `backend/app/aria/skills/<name>_skill.py` + intent dans `brain.py`
- Politique API payante : `x_publication_policy.py`
- Objectif revenu ZHC : `revenue_goals.py`
- Peer agents : `knowledge/zhc_peer_agents.py`

Voir `references/aria-backend-patterns.md` pour le détail.