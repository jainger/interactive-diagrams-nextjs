# Lucid Diagram Orchestrator

## Účel
Tento projekt slouží k tvorbě a správě business diagramů v Lucidu pomocí přirozeného jazyka.
Jako lead agent čteš tento soubor jako první a řídíš celý workflow.

## Složková struktura

```
docs/YYYY-MM-DD/          ← business zadání per den (YYYY-MM-DD = dnešní datum)
  brief.md                ← analytic výstup: as-is shrnutí, scope, stakeholders
  open-points.md          ← analytic výstup: otevřené body seskupené po tématech
  decisions.md            ← workshop výstup: potvrzená rozhodnutí z workshopu

diagrams/<název>/
  current.json            ← aktuální Lucid JSON export
  history/YYYY-MM-DD.json ← snapshot před větší změnou

.claude/agents/           ← analytic · workshop · validator
```

Vždy použij dnešní datum pro `YYYY-MM-DD`. Pokud si nejsi jistý, spusť `date +%Y-%m-%d`.

## Scénáře

### A) Nový diagram
Uživatel popisuje nový business proces, žádný `diagrams/<název>/` neexistuje.
→ Spusť plný workflow: Analytic → Workshop → Validator → Commit.

### B) Update existujícího diagramu
Uživatel chce upravit existující diagram. Složka `diagrams/<název>/` existuje.
→ Přečti `diagrams/<název>/current.json` a předej ho analytic a workshop jako kontext.
→ Před uložením nového JSONu překopíruj `current.json` do `history/YYYY-MM-DD.json`.
→ Spusť plný workflow: Analytic → Workshop → Validator → Commit.

### C) Jen workshop (bez analytic)
Uživatel přichází s hotovým briefem nebo chce rovnou modelovat.
→ Přeskoč analytic, spusť workshop a validator.

## Workflow (fáze)

### 1. Analytic fáze — spawn `analytic` teammate

Předej mu v spawn promptu:
- Business záměr od uživatele (doslova)
- Pokud existuje: obsah `diagrams/<název>/current.json` jako "existing diagram context"
- Instrukce: `Save output files to docs/YYYY-MM-DD/ — brief.md, open-points.md.`

Počkej na zprávu od analytic že je hotovo.

### 2. Workshop fáze — spawn `workshop` teammate

Předej mu v spawn promptu:
- `Read docs/YYYY-MM-DD/brief.md and docs/YYYY-MM-DD/open-points.md as input.`
- Business záměr od uživatele
- Pokud existuje: obsah `diagrams/<název>/current.json`
- Instrukce: `Save confirmed decisions to docs/YYYY-MM-DD/decisions.md. Return Lucid-ready BPMN instructions.`

Počkej na zprávu od workshop že je hotovo.

### 3. Validator fáze — spawn `validator` teammate

Předej mu v spawn promptu:
- `Read docs/YYYY-MM-DD/brief.md, open-points.md, decisions.md as input.`
- Workshop BPMN instrukce (zkopíruj z workshop výstupu)
- Instrukce: `Return readiness verdict: Ready / Ready with fixes / Not ready. List all Critical and High findings.`

Počkej na zprávu od validator že je hotovo.

### 4. Syntéza — po dokončení všech teammates

Pokud validator vrátil **"Not ready"** nebo má **Critical** findings:
- Informuj uživatele o blockerech
- Neptej se na export — nejdřív musí být opraveny Critical issues
- Navrhni spustit workshop znovu s opravami

Pokud validator vrátil **"Ready"** nebo **"Ready with fixes"**:
- Shrň finální Lucid instrukce pro uživatele
- Požádej o export JSON z Lucidu: `File → Export → Lucid Document JSON`

### 5. Verzování diagramu — po obdržení JSON od uživatele

```bash
# Pokud current.json existuje, archivuj ho
cp diagrams/<název>/current.json diagrams/<název>/history/$(date +%Y-%m-%d).json

# Ulož nový JSON
# (uživatel ho vloží nebo nahraje)

git add diagrams/
git commit -m "diagram: update <název>"
```

Commitni také docs z dnešního dne:
```bash
git add docs/$(date +%Y-%m-%d)/
git commit -m "docs: add workshop notes $(date +%Y-%m-%d) <název>"
```

## Lucid MCP — kdy použít

MCP server `lucid` je dostupný všem teammates automaticky.

| Situace | Akce |
|---------|------|
| Chceš přečíst existující Lucid dokument | Použij MCP nástroj pro fetch dokumentu |
| Chceš poslat instrukce pro kreslení | Vracej textové Lucid-ready instrukce (MCP není potřeba) |
| Uživatel chce přímou aktualizaci v Lucidu | Použij MCP nástroj pro update/push do Lucidu |
| Uživatel exportuje JSON ručně | Jen ulož soubor a commitni |

## Pravidla

- Nikdy nepřeskakuj validator fázi.
- Critical findings od validatoru = blokují export a commit.
- Vždy uložit brief.md před spawnutím workshop.
- Pojmenování diagramů: lowercase, pomlčky místo mezer (`order-process`, `invoice-approval`).
- Každý commit diagramu = jedna logická změna.
- Teammates komunikují přes zprávy — po dokončení každé fáze pošli leadovi zprávu s výsledkem.
- Nikdy nepřepisuj `history/` soubory — jsou to snapshoty.
