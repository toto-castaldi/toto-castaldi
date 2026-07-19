# Project Retrospective

*A living document updated after each milestone. Lessons feed forward into future planning.*

## Milestone: v1.2 — Rimozione Helix

**Shipped:** 2026-07-19
**Phases:** 1 | **Plans:** 3 | **Sessions:** 1

### What Was Built
- Helix rimosso da tutti gli artefatti (data/projects.toml, description SEO IT/EN, CLAUDE.md) — zero match residui nei sorgenti e nel build
- Griglia progetti desktop riflowata da 3 a 2 colonne (Docora, Lumio) con card GitHub a tutta larghezza sotto
- Verifica umana firmata: layout, responsive mobile e contrasto WCAG AA in entrambi i temi su IT/EN

### What Worked
- Esecuzione parallela in worktree (07-01 + 07-02 senza overlap di file) — wave 1 completata in ~3 min per piano
- Checkpoint human-verify come piano dedicato (07-03): gate automatici prima, occhio umano solo sul visivo
- Modifica minimale: 5 file sorgente, +6/-14 righe — nessun refactoring fuori scope

### What Was Inefficient
- REQUIREMENTS.md non aggiornato automaticamente da `phase.complete` — checkbox sistemate a mano in chiusura milestone
- Una build cache locale stantia (`public/`, `resources/`) ha temporaneamente mostrato CSS vecchio durante la verifica — serviva build pulita

### Patterns Established
- Regola desktop `repeat(2,1fr)` ora identica a quella tablet: candidata a pulizia (dead code segnalato in 07-REVIEW.md IN-01)

### Key Lessons
1. Per verifiche su siti Hugo, fare sempre `rm -rf public resources` prima della build di controllo — la cache fingerprinted inganna
2. I piani checkpoint (autonomous: false) funzionano bene come ultima wave: separano i gate automatici dal sign-off visivo

### Cost Observations
- Model mix: ~80% opus (executor), ~20% sonnet (verifier)
- Sessions: 1
- Notable: fase completa (plan → execute → verify → ship) in una singola sessione, ~1 ora

---

## Cross-Milestone Trends

### Process Evolution

| Milestone | Sessions | Phases | Key Change |
|-----------|----------|--------|------------|
| v1.0 | ~3 | 3 | Bootstrap GSD, deploy pipeline GitHub Pages |
| v1.1 | ~3 | 3 | Restyling bento grid, design token |
| v1.2 | 1 | 1 | Prima milestone single-phase; checkpoint human-verify come piano dedicato |

### Cumulative Quality

| Milestone | Tests | Coverage | Zero-Dep Additions |
|-----------|-------|----------|-------------------|
| v1.0 | build gate | — | 0 (zero-JS) |
| v1.1 | build gate | — | 0 (zero-JS) |
| v1.2 | build + grep gates | — | 0 (zero-JS) |

### Top Lessons (Verified Across Milestones)

1. I commit locali vanno pushati manualmente — verificare `git status -sb` a fine milestone (v1.0: 24 commit non pushati; v1.2: 21+ in attesa)
2. Il vincolo zero-JS + CSS custom properties regge bene anche i restyling: nessuna dipendenza aggiunta in tre milestone
