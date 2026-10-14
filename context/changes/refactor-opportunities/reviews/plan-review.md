<!-- PLAN-REVIEW-REPORT -->
# Plan Review: K2 twin personality guards

- **Plan**: `context/changes/refactor-opportunities/plan.md`
- **Mode**: Deep
- **Date**: 2026-10-14
- **Verdict**: SOUND (after triage)
- **Findings**: 0 critical 2 warnings 1 observations (all FIXED)

## Verdicts

| Dimension | Verdict |
|-----------|---------|
| End-State Alignment | PASS |
| Lean Execution | WARNING |
| Architectural Fitness | PASS |
| Blind Spots | WARNING |
| Plan Completeness | PASS |

## Grounding

5/5 paths ✓, 7/7 symbols ✓, brief↔plan ~ (Phase 2 risk wording in brief still says `userProperties`; plan already documents System/config + userProperties anti-pattern)

Paths exist: impl/compat `DefaultModelBuilder`, `ParentCycleDetectionTest`, compat `DefaultModelBuilderTest`, ITS `MavenITmng8294ParentChecksTest` (referenced). Symbols: `mismatchRelativePathAndGA` (`:1324`, callers `:1899`/`:2309`), `Constants.MAVEN_MAVEN3_PERSONALITY`, `Features.mavenMaven3Personality` ← configProperties, FATAL unit `:336–403`, MNG-5146 WARNING, no demotion unit yet. Progress↔Phase criteria match. Prior triage fixes (configProperties wiring, WRONG_PARENT SoT) are present in the plan body.

## Findings

### F1 — Phase 2 demotion layout may be thrown away in Phase 3

- **Severity**: ⚠️ WARNING
- **Impact**: 🔎 MEDIUM — real tradeoff; pause to reason through it
- **Dimension**: Lean Execution
- **Location**: Phase 2 — Demotion unit + Phase 3 — Paired fixture
- **Detail**: Phase 2 places the demotion test next to the shade-style FATAL (`dependency-reduced-pom` ↔ `pom.xml`). Phase 3 then declares shade non-canonical for the pair and says „zastąpić lub dodać” WRONG_PARENT from compat. If implementer clones shade for demotion in Phase 2 and Phase 3 chooses replace, the demotion test is rewritten once — avoidable churn. Desired End State only needs FATAL default + WARNING personality on a paired layout, not shade specifically.
- **Fix A ⭐ Recommended**: In Phase 2, build demotion (and keep/adjust FATAL) on the compat WRONG_PARENT shape already named as Phase 3 SoT; Phase 3 then only wires the pair / review, not a second layout choice.
  - Strength: One layout decision; Phase 2 work survives Phase 3; matches „Source of truth = compat MNG-5146 / WRONG_PARENT”.
  - Tradeoff: Phase 2 is slightly larger (layout copy earlier); shade FATAL may stay as extra coverage or be deferred.
  - Confidence: HIGH — both layouts are explicit GA mismatch; hot path does not care which POM shape.
  - Blind spot: Whether to keep shade FATAL as additional regression (GH-13191) vs drop after WRONG_PARENT FATAL lands.
- **Fix B**: Keep Phase 2 on shade; Phase 3 must say **add** (never replace) WRONG_PARENT pair, and demotion must also cover that shape (or only the paired shape needs demotion).
  - Strength: Preserves existing GH-13191 shade characterization untouched in Phase 2.
  - Tradeoff: Two explicit-mismatch layouts on impl; demotion may need a second assertion or move in Phase 3.
  - Confidence: HIGH — plan already allows „dodać”.
  - Blind spot: Duplicate maintenance of two layouts.
- **Decision**: FIXED via Fix A

### F2 — Demotion test must recreate session; @BeforeEach already called createSession

- **Severity**: ⚠️ WARNING
- **Impact**: 🏃 LOW — quick decision; fix is obvious and narrowly scoped
- **Dimension**: Blind Spots
- **Location**: Phase 2 — Demotion unit (impl)
- **Detail**: `ParentCycleDetectionTest` creates `session = ApiRunner.createSession()` in `@BeforeEach` with no personality. Plan correctly forbids `request.userProperties` and points at System property before `createSession` or mutating `configProperties`. If the implementer only `System.setProperty` inside the demotion `@Test` after setup, the existing session’s config map will not see it (ApiRunner copies system props at create time). Class has zero existing System/config patterns to copy.
- **Fix**: In Phase 2 Contract, require demotion test to set `Constants.MAVEN_MAVEN3_PERSONALITY` then call `ApiRunner.createSession()` locally (override shared field) or mutate a mutable config map on the session before `build`; document `@AfterEach` / try-finally cleanup for System property. Do not rely on `@BeforeEach` session alone after a late setProperty.
- **Decision**: FIXED

### F3 — plan-brief Phase 2 risk still says userProperties

- **Severity**: OBSERVATION
- **Impact**: 🏃 LOW — quick decision; fix is obvious and narrowly scoped
- **Dimension**: Plan Completeness
- **Location**: `plan-brief.md` — Phases at a Glance / Open Risks
- **Detail**: Brief key risk for Phase 2 still reads „`userProperties` może nie zasilić feature flag”. Plan body already specifies System/configProperties and marks userProperties as anti-pattern (prior triage Fix A). Brief↔plan drift can mislead a skim reader.
- **Fix**: Update brief Phase 2 risk to „System/configProperties wiring + cleanup; userProperties anti-pattern under ApiRunner”.
- **Decision**: FIXED
