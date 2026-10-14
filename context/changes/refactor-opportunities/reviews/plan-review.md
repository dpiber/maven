<!-- PLAN-REVIEW-REPORT -->
# Plan Review: K2 twin personality guards

- **Plan**: context/changes/refactor-opportunities/plan.md
- **Mode**: Deep
- **Date**: 2026-10-14
- **Verdict**: SOUND (after triage; was REVISE)
- **Findings**: 0 critical 1 warnings 1 observations

## Verdicts

| Dimension | Verdict |
|-----------|---------|
| End-State Alignment | PASS (was WARNING; F1 fixed) |
| Lean Execution | PASS (F2 fixed) |
| Architectural Fitness | PASS |
| Blind Spots | PASS |
| Plan Completeness | PASS (was WARNING; F1 fixed) |

## Grounding

Grounding: 5/5 paths ✓, 5/5 symbols ✓, brief↔plan ✓

Deep verification confirmed: System property → ApiRunner → configProperties → Features.mavenMaven3Personality; userProperties anti-pattern; post-demotion Non-resolvable parent FATAL without ParentProvidingResolver; WRONG_PARENT SoT only in compat; mismatchRelativePathAndGA callers local only.

## Findings

### F1 — FATAL default na WRONG_PARENT nie jest w Success Criteria / Progress

- **Severity**: ⚠️ WARNING
- **Impact**: 🔎 MEDIUM — real tradeoff; pause to reason through it
- **Dimension**: End-State Alignment / Plan Completeness
- **Location**: Phase 2 — Changes Required vs Success Criteria / Progress 2.2; Phase 3 — paired coverage 3.3
- **Detail**: Phase 2 Changes Required każe dodać FATAL (default) + demotion WARNING na layoutcie WRONG_PARENT (SoT = MNG-5146). Success Criteria i Progress 2.2 wymagały tylko: nowy test asertuje WARNING + „istniejący FATAL+hint (shade) nadal przechodzi”. Phase 3 mówi wprost, że shade jest poza kanoniczną parą, a para ma pokrywać „impl FATAL (default) + WARNING (personality)” — bez nowego FATAL na WRONG_PARENT implementer mógł domknąć checkboxy, a kanoniczna para pozostałaby dziurawa.
- **Fix A ⭐ Recommended**: Dopisać do Phase 2 Success Criteria + Progress krok asertujący FATAL mismatch na WRONG_PARENT bez personality (obok demotion WARNING).
  - Strength: Zamyka lukę między Changes / Desired End State / Phase 3 a toolingiem `/10x-implement`; para staje się jednoznaczna.
  - Tradeoff: Jeden dodatkowy test (lub dwa scenariusze) zamiast samego demotion.
  - Confidence: HIGH — layout i asercja FATAL już istnieją po stronie shade i compat; kod demotion potwierdzony (DefaultModelBuilder.java:1361–1362).
  - Blind spot: None significant.
- **Fix B**: Osłabić Phase 2 Changes / Phase 3 wording — kanoniczna para = shade FATAL + WRONG_PARENT WARNING + compat MNG-5146.
  - Strength: Mniej kodu w Phase 2; shade już jest.
  - Tradeoff: Sprzeczne z własnym „shade poza kanoniczną parą” i ze SoT = WRONG_PARENT; para porównuje różne layouty.
  - Confidence: LOW — podważa decyzję planu o SoT.
  - Blind spot: Czy review/research nadal akceptuje mieszaną parę.
- **Decision**: FIXED via Fix A

### F2 — Komentarz personality już istnieje przy warn=

- **Severity**: 💡 OBSERVATION
- **Impact**: 🏃 LOW — quick decision; fix is obvious and narrowly scoped
- **Dimension**: Lean Execution
- **Location**: Phase 1 — Code comment at mismatchRelativePathAndGA
- **Detail**: Przy `warn = defaultPath || maven3Mode` jest już komentarz (DefaultModelBuilder.java:1358–1360) o WARNING przy defaultPath / personality i FATAL przy explicit mismatch. Brakuje tylko half „compat twin = WARNING / nie unifikować severity”.
- **Fix**: Rozszerzyć istniejący blok o twin/compat + „do not unify”, zamiast dokładać drugi równoległy komentarz.
- **Decision**: FIXED

## Triage summary

- Fixed: F1 (Fix A), F2
- Skipped: —
- Accepted: —
- Dismissed: —
- Verdict after fixes: REVISE → SOUND
