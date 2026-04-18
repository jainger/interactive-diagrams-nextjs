# Lucid Diagram Orchestrator

## Účel
Tento projekt slouží k tvorbě a správě business diagramů v Lucidu pomocí přirozeného jazyka.
Jako lead agent čteš tento soubor jako první a řídíš celý workflow.

## Složková struktura
- `docs/YYYY-MM-DD/` — business zadání, open points, decisions z daného dne
- `diagrams/<název>/current.json` — aktuální Lucid JSON export (verzovaný v gitu)
- `diagrams/<název>/history/YYYY-MM-DD.json` — snapshot před větší změnou
- `.claude/agents/` — definice agentů: analytic, workshop, validator

## Workflow (vždy v tomto pořadí)

### 1. Analytic fáze
Spawni `analytic` teammate s tímto kontextem:
- Business záměr od uživatele
- Případné existující diagramy z `diagrams/`
- Instrukce: uložit výstup do `docs/YYYY-MM-DD/brief.md`

### 2. Workshop fáze
Spawni `workshop` teammate s tímto kontextem:
- Obsah `docs/YYYY-MM-DD/brief.md` z analytic fáze
- Instrukce: produkovat Lucid-ready BPMN instrukce

### 3. Validator fáze
Spawni `validator` teammate s tímto kontextem:
- Výstup workshop fáze
- Instrukce: zkontrolovat BPMN notaci a konzistenci, vypsat findings se severity

### 4. Syntéza
Po dokončení všech teammates:
- Syntetizuj výstupy
- Předej uživateli finální Lucid instrukce
- Požádej uživatele o export JSON z Lucidu

### 5. Verzování diagramu
Po obdržení JSON od uživatele:
- Pokud `diagrams/<název>/current.json` již existuje → překopíruj ho do `diagrams/<název>/history/YYYY-MM-DD.json`
- Uložit nový JSON jako `diagrams/<název>/current.json`
- Commitnout: `git add diagrams/ && git commit -m "diagram: update <název>"`

## Jak spawnat agent team

Při každém business požadavku spawni teammates pomocí jejich agent type:

```
Spawn an analytic teammate to prepare the workshop brief. Save output to docs/YYYY-MM-DD/brief.md.
Spawn a workshop teammate to model the BPMN based on the brief.
Spawn a validator teammate to review the BPMN output.
```

Počkej na dokončení všech teammates před syntézou.

## Pravidla
- Nikdy nepřeskakuj validator fázi.
- Vždy uložit brief před spawnutím workshop.
- Pojmenování diagramů: lowercase, pomlčky místo mezer (např. `order-process`).
- Každý commit diagramu = jedna logická změna.

## Lucid MCP
MCP server pro Lucid je nakonfigurovaný v `.mcp.json`.
Při prvním použití proběhne OAuth flow v browseru (jednorázové per uživatel).
Lucid nástroje jsou dostupné pro všechny teammates automaticky.
