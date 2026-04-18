# Lucid Diagram Orchestrator — Design Spec

**Date:** 2026-04-18  
**Status:** Approved

## Cíl

Umožnit jakémukoliv business uživateli tvořit a upravovat BPMN diagramy v Lucidu pomocí přirozeného jazyka přes Claude Code — bez nutnosti znát BPMN notaci nebo agenty.

## Architektura

Přístup B: workflow-driven orchestrace přes CLAUDE.md + Claude Code agent teams (experimentální).

```
interactive-diagrams-nextjs/
├── CLAUDE.md                        ← orchestrátor, hlavní vstupní bod
├── docs/
│   └── YYYY-MM-DD/                  ← jedna složka per den
│       ├── brief.md                 ← business záměr
│       ├── open-points.md           ← otevřené body
│       └── decisions.md             ← potvrzená rozhodnutí
├── diagrams/
│   └── <nazev-diagramu>/
│       ├── current.json             ← aktuální Lucid JSON export
│       └── history/
│           └── YYYY-MM-DD.json      ← snapshoty starších verzí
└── .claude/
    ├── settings.json                ← MCP konfig + agent teams flag
    └── agents/
        ├── analytic.md
        ├── validator.md
        └── workshop.md
```

## Agent Teams Workflow

Business uživatel popíše záměr → Claude (lead) přečte CLAUDE.md → spawne 3 teammates:

1. **analytic** teammate — pochopí business kontext, uloží `docs/YYYY-MM-DD/brief.md`
2. **workshop** teammate — modeluje BPMN, produkuje Lucid-ready instrukce
3. **validator** teammate — zkontroluje BPMN notaci a konzistenci

Lead syntetizuje výstupy → uživatel aktualizuje diagram v Lucidu → exportuje JSON → `diagrams/<název>/current.json` → git commit.

## MCP Konfigurace

**Lucid MCP server:** `https://mcp.lucid.app/mcp`  
**Auth:** OAuth via Dynamic Client Registration — Claude Code spustí browser flow automaticky při prvním použití. Žádný manuální token.

**settings.json:**
```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  },
  "mcpServers": {
    "lucid": {
      "type": "http",
      "url": "https://mcp.lucid.app/mcp"
    }
  }
}
```

## CLAUDE.md obsah

Orchestrační instrukce popisující:
- Složkovou strukturu a konvence pojmenování
- Workflow fáze (Analytic → Workshop → Validator)
- Kdy a jak spouštět agent team
- Pravidla pro verzování diagramů (current.json + history/)

## Diagram verzování

- Formát: Lucid JSON export
- `current.json` = živá verze (vždy přepsána po aktualizaci)
- `history/YYYY-MM-DD.json` = snapshot před větší změnou
- Verzování přes git — každá změna diagramu = commit

## Onboarding nového uživatele

1. Klonovat repo
2. Otevřít Claude Code v adresáři projektu
3. Při prvním použití Lucid MCP → browser OAuth flow (jednorázové)
4. Popsat business záměr přirozeným jazykem → Claude řídí zbytek
