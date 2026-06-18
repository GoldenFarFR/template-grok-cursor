# Patterns backend ARIA (référence dexpulse)

À réutiliser quand un nouveau repo embarque un agent FastAPI ou s'inspire de DEXPulse.

## Structure skill runtime

```
app/aria/
├── brain.py              # INTENT_PATTERNS + routing
├── skills/
│   └── <name>_skill.py   # execute_<name>() → (text, dict)
├── memory.py             # append_memory + LLM context
├── knowledge/            # SSOT statique (YAML/Python)
└── gateway/              # Telegram, X, etc.
```

## Checklist nouveau skill

1. `SkillName` dans `models.py`
2. Patterns dans `brain.py` `INTENT_PATTERNS`
3. Handler `elif intent == SkillName.X`
4. `public_mode.py` si opérateur-only
5. Tests `tests/test_<name>.py`
6. Entrée FAQ/canonical si fait public

## Fichiers mémoire (runtime, pas git)

`data/memory/*.md` — initiative, ledger, policy.  
Template nouveau repo : documenter dans README, pas committer `data/`.

## Holding vs produit

- Site vitrine → repo séparé (comme `aria-vanguard`)
- API/agent → repo produit (comme `dexpulse`)
- Jamais merger les deux deploy surfaces (VISION §8)