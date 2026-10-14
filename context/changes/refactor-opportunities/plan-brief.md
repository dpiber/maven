# K2 twin personality guards — Plan Brief

> Full plan: `context/changes/refactor-opportunities/plan.md`
> Research: `context/changes/refactor-opportunities/research.md`

## What & Why

Zamykamy lukę characterization na świadomym kontrakcie twin M3↔M4 dla explicit relativePath GA mismatch (FATAL w M4 vs WARNING w compat / przy `maven.maven3Personality`). Cel: guard przed fałszywym „fix twin”, nie przebudowa semantyki.

## Starting Point

Impl ma unit FATAL+hint, compat ma MNG-5146 WARNING; brak unit demotion personality i brak paired fixture. Ranking researchu proponował K2 → K1 → K3; wywiad wybrał wyłącznie pełne K2.

## Desired End State

Kontrakt udokumentowany (komentarz + notatka w change folder) i pokryty testami: impl FATAL (default) / WARNING (personality); compat WARNING. Produkcja bez zmiany semantyki.

## Key Decisions Made

| Decision | Choice | Why (1 sentence) | Source |
| -------- | ------ | ---------------- | ------ |
| Która opportunity | Pełne K2 (nie K1/K3) | Najniższy blast przy wysokim koszcie błędu „fix twin” | Plan (ranking = propozycja) |
| Głębokość K2 | Demotion + paired + lokalny kontrakt | Kontrpytanie odrzucone — cieńszy guard nie zamyka paired | Plan |
| Wykluczenia | Bez unifikacji severity, bez extractów, bez Jacoco/SPI/mapy | Świadome ograniczenia i poza ścieżką tej zmiany | Plan + Research |
| Unknowns | NOT doing / follow-up; zero discovery mid-impl | Unika scope creep (configProperties/ClassRealm) | Plan |
| Kolejność faz | Kontrakt → demotion → paired; ITS opcjonalny smoke | Guard-first, najtańsze najpierw | Plan |

## Scope

**In scope:**
- Komentarz przy `mismatchRelativePathAndGA`
- `contract-twin-relativepath.md` w folderze zmiany
- Unit demotion personality w `ParentCycleDetectionTest`
- Paired fixture layout impl↔compat (skopiowany, bez shared test-jar)

**Out of scope:**
- Unifikacja FATAL↔WARNING
- Extract K1 / K3
- Nowe SPI, migracja deprecated ProjectBuilder
- Jacoco jako cel; ClassRealm/`ModelTransformer` research
- Korekty `repo-map.md`

## Architecture / Approach

Characterization tests + dokumentacja kontraktu na istniejącym hubie ModelBuilder i deprecated compat twin. Brak Strangler / Branch by Abstraction — to nie migracja struktury.

## Phases at a Glance

| Phase | What it delivers | Key risk |
| ----- | ---------------- | -------- |
| 1. Contract note | Komentarz + notatka kontraktu | Zbyt ogólny wording → mylący „fix” |
| 2. Demotion unit | WARNING przy personality | System/configProperties wiring + cleanup; userProperties anti-pattern under ApiRunner |
| 3. Paired fixture | Ten sam layout po obu stronach twin | Drift layoutu między modułami |

**Prerequisites:** Zweryfikowany `research.md`; istniejący FATAL unit + MNG-5146  
**Estimated effort:** ~1–2 sesje implementacji (3 fazy, głównie testy)

## Open Risks & Assumptions

- Demotion w unit może wymagać follow-up wiring properties (poza zakresem)
- Adopcja `-Dmaven.maven3Personality` w ekosystemie = unknown follow-up

## Success Criteria (Summary)

- Kontrakt zapisany i zgodny z researchem (nie sync severity)
- Unit: FATAL default + WARNING personality po stronie impl
- Compat WARNING pozostaje; para layoutów dokumentuje twin bez zmiany semantyki
