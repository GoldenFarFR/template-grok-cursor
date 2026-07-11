# Écosystème GoldenFar — où mettre quoi

**Vision SSOT :** [ARIA/VISION.md](https://github.com/GoldenFarFR/ARIA/blob/main/VISION.md)  
**Carte complète :** [ARIA/vanguard/docs/ECOSYSTEM-REPOS.md](https://github.com/GoldenFarFR/ARIA/blob/main/vanguard/docs/ECOSYSTEM-REPOS.md)

## Nouveau repo depuis ce template

1. **Use this template** → nouveau repo sous `GoldenFarFR/`
2. Copier `.grok/references/scaffold/` si site web holding
3. Lier les skills métier : `aria-skills/scripts/install.ps1` (statut `aria-skills` non vérifié cette session) ou `scripts/link-aria-skills.ps1` depuis ce repo
4. Lire `VISION.md` (`ARIA`) avant toute feature produit
5. Secrets → **jamais** dans le repo ; coffre local + `aria-ops/vanguard/operator/` pour deploy Render

## Repos officiels (mis à jour 11/07/2026 — voir GoldenFarFR sur GitHub)

| Repo | Rôle |
|------|------|
| **ARIA** (public) | Monorepo : holding + API (`vanguard/` : `backend/`, vitrine, DEXPulse app) + cœur agent (`packages/aria-core/`). Remplace les anciens `aria-vanguard` et `aria-sandbox`, fusionnés dedans. |
| **aria-ops** (privé) | Infra/ops : scripts déploiement, coffre secrets, mémoire opérateur — inclut `collegue-memoire/` et `local-sync/` (anciens repos séparés, absorbés ici) |
| **aria-skills**, **aria-token-base** | Mentionnés dans d'anciens docs — statut actuel non vérifié cette session |
| **template-grok-cursor** | Ce template (bootstrap nouveau repo) |
| **kikou** | Exemple site lambda holding déployé |
| ~~dexpulse~~, ~~dexpulse-secrets~~, ~~collegue-memoire~~, ~~aria-local-sync~~ | *Dépréciés* — fusionnés dans `ARIA` / `aria-ops` |

## Règle d'or

| Type | Où ça vit |
|------|-----------|
| Cerveau ARIA (runtime) | `ARIA` → package `packages/aria-core/` (anciennement `aria-sandbox`) |
| Holding + API + app produit | `ARIA` → `vanguard/` (anciennement `aria-vanguard`) |
| Skills IDE (moat) | `aria-skills` (statut non vérifié cette session) |
| Scripts opérateur + deploy | `aria-ops/vanguard/operator/` (hors Git, secrets dans coffre) |

## Patterns copiables

- Skill runtime : `ARIA/packages/aria-core/src/aria_core/skills/`
- Plugins hôte marché : `ARIA/vanguard/backend/app/integrations/aria_host.py`
- Politique coût API : `aria_core/x_publication_policy.py`
- Site holding Privy : `ARIA/vanguard/src/` ou scaffold de ce template
- Deploy holding : `aria-ops/vanguard/operator/setup-holding-render.ps1`

Voir `aria-backend-patterns.md` pour le détail backend.