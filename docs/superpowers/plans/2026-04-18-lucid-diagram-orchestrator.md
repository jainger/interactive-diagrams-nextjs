# Lucid Diagram Orchestrator Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Sestavit projekt-strukturu, CLAUDE.md orchestrátor a MCP konfiguraci tak, aby jakýkoliv business uživatel mohl přirozeným jazykem tvořit BPMN diagramy v Lucidu přes Claude Code.

**Architecture:** CLAUDE.md jako orchestrační instrukce pro Claude (lead) + Claude Code agent teams (experimentální) využívající existující agenty (analytic, validator, workshop). Lucid je napojený přes remote MCP server s OAuth. Diagramy jsou verzované jako JSON v gitu.

**Tech Stack:** Claude Code agent teams, Lucid MCP (`https://mcp.lucid.app/mcp`), git

---

## File Map

| Soubor | Akce | Obsah |
|--------|------|-------|
| `.claude/settings.json` | Modify | + mcpServers (Lucid) + env CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS |
| `CLAUDE.md` | Create | Orchestrační instrukce pro lead agenta |
| `diagrams/.gitkeep` | Create | Placeholder pro git tracking prázdné složky |
| `diagrams/README.md` | Create | Konvence pojmenování a workflow |

---

### Task 1: Aktualizovat .claude/settings.json

**Files:**
- Modify: `.claude/settings.json`

- [ ] **Step 1: Zapsat nový obsah settings.json**

Nahradit celý obsah souboru `.claude/settings.json`:

```json
{
  "enabledPlugins": {
    "superpowers@claude-plugins-official": true
  },
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

- [ ] **Step 2: Ověřit syntaxi JSON**

```bash
python3 -m json.tool .claude/settings.json
```

Očekávaný výstup: vypíše naformátovaný JSON bez chyby.

- [ ] **Step 3: Commit**

```bash
git add .claude/settings.json
git commit -m "feat: add Lucid MCP server and agent teams config"
```

---

### Task 2: Vytvořit CLAUDE.md orchestrátor

**Files:**
- Create: `CLAUDE.md`

- [ ] **Step 1: Vytvořit CLAUDE.md s tímto přesným obsahem**

```markdown
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
MCP server pro Lucid je nakonfigurovaný v `.claude/settings.json`.
Při prvním použití proběhne OAuth flow v browseru (jednorázové per uživatel).
Lucid nástroje jsou dostupné pro všechny teammates automaticky.
```

- [ ] **Step 2: Ověřit soubor existuje a má správný obsah**

```bash
head -5 CLAUDE.md
```

Očekávaný výstup: `# Lucid Diagram Orchestrator`

- [ ] **Step 3: Commit**

```bash
git add CLAUDE.md
git commit -m "feat: add CLAUDE.md orchestrator for Lucid diagram workflow"
```

---

### Task 3: Vytvořit diagrams/ strukturu

**Files:**
- Create: `diagrams/.gitkeep`
- Create: `diagrams/README.md`

- [ ] **Step 1: Vytvořit složku a placeholder**

```bash
mkdir -p diagrams
touch diagrams/.gitkeep
```

- [ ] **Step 2: Vytvořit README.md pro konvence**

Vytvořit soubor `diagrams/README.md`:

```markdown
# Diagrams

Tato složka obsahuje Lucid JSON exporty business diagramů verzované v gitu.

## Struktura

```
diagrams/
└── <nazev-diagramu>/
    ├── current.json        ← aktuální verze (vždy přepsána)
    └── history/
        └── YYYY-MM-DD.json ← snapshot před větší změnou
```

## Jak exportovat JSON z Lucidu

1. Otevřít diagram v Lucidu
2. File → Export → JSON (Lucid format)
3. Uložit jako `diagrams/<nazev>/current.json`
4. Spustit v Claude Code: Claude provede git commit

## Pojmenování

- Lowercase, pomlčky místo mezer
- Popisný název procesu: `order-process`, `invoice-approval`, `onboarding-flow`
```

- [ ] **Step 3: Commit**

```bash
git add diagrams/
git commit -m "feat: add diagrams folder structure and conventions"
```

---

### Task 4: Ověřit Lucid MCP připojení (manuální)

**Files:** žádné (verifikační krok)

- [ ] **Step 1: Restartovat Claude Code session**

Zavřít a znovu otevřít Claude Code v tomto projektu, aby načetl nový MCP config.

- [ ] **Step 2: Spustit OAuth flow**

V Claude Code napsat:
```
Jaké Lucid nástroje mám k dispozici?
```

Claude Code otevře browser → přihlásit se do Lucid účtu → potvrdit přístup.

- [ ] **Step 3: Ověřit dostupné nástroje**

Po OAuth flow by Claude měl vypsat dostupné Lucid MCP nástroje (např. list documents, update diagram, apod.).

Pokud OAuth selže: přejít na `https://lucid.app/user#/appsAndIntegrations/lucidchart` → Disconnect → znovu spustit krok 2.

---

### Task 5: Ověřit agent teams (manuální smoke test)

**Files:** žádné (verifikační krok)

- [ ] **Step 1: Ověřit verzi Claude Code**

```bash
claude --version
```

Očekávaný výstup: verze ≥ 2.1.32 (agent teams requirement).

- [ ] **Step 2: Smoke test agent team**

V Claude Code napsat:
```
Chci vytvořit jednoduchý diagram procesu schvalování faktury. Použij agent team.
```

Očekávané chování:
- Claude spawne 3 teammates (analytic, workshop, validator)
- Použij Shift+Down pro přepínání mezi teammates
- Analytic uloží `docs/YYYY-MM-DD/brief.md`
- Workshop produkuje Lucid instrukce
- Validator vypíše review

- [ ] **Step 3: Ověřit docs/ výstup**

```bash
ls docs/$(date +%Y-%m-%d)/
```

Očekávaný výstup: `brief.md` (případně `open-points.md`, `decisions.md`)

- [ ] **Step 4: Commit výsledků testu**

```bash
git add docs/
git commit -m "docs: add first workshop brief from smoke test"
```

---

## Onboarding checklist pro nového uživatele

Po dokončení implementace si každý nový uživatel udělá jen tyto kroky:

1. `git clone <repo>`
2. Otevřít Claude Code v projektu
3. Napsat cokoliv — Claude OAuth flow spustí automaticky
4. Přihlásit se do Lucid v browseru (jednorázové)
5. Pracovat přirozeným jazykem

## Hotovo

Po Tasku 5 je projekt plně funkční. Každý business uživatel může:
- Popsat záměr přirozeným jazykem
- Nechat Claude orchestrovat analytic → workshop → validator workflow
- Exportovat JSON z Lucidu a verzovat v gitu
