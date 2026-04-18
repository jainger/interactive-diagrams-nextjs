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
