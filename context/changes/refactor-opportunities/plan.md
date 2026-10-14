# K2 twin personality guards — Implementation Plan

## Overview

Zamknąć lukę characterization na świadomym kontrakcie twin M3↔M4 dla explicit relativePath GA mismatch: dokumentacja intencji + unit demotion `maven.maven3Personality` + paired fixture impl↔compat — bez unifikacji severity i bez extractów strukturalnych (K1/K3).

## Current State Analysis

- Hot path M4: `impl/.../DefaultModelBuilder.mismatchRelativePathAndGA` — `warn = defaultPath || maven3Mode`, inaczej FATAL (`DefaultModelBuilder.java:1324–1362`).
- Twin compat: zawsze WARNING przy mismatch (`compat/.../DefaultModelBuilder.java:1072–1074`); brak personality w compat.
- Unit impl: `ParentCycleDetectionTest#testWrongRelativePathInGeneratedPomIsFatalWithMaven3Hint` asertuje FATAL + hint o personality (`ParentCycleDetectionTest.java:336–403`); **brak** unit demotion → WARNING.
- Compat: MNG-5146 WARNING w `DefaultModelBuilderTest` (`:111–229`).
- Stałe: `Constants.MAVEN_MAVEN3_PERSONALITY`, `Features.mavenMaven3Personality`.
- Ranking researchu proponował K2 → K1 → K3; decyzja planu: wyłącznie pełne K2 (guard).

### Key Discoveries:

- Personality jest świadomym ograniczeniem (MNG-5146 / `maven.maven3Personality`), nie bugiem do „naprawy” severity.
- Paired fixture = skopiowany layout POM między modułami; brak shared test-jar.
- `Features.mavenMaven3Personality` czyta `configProperties` sesji; testy mogą używać `userProperties` — jeśli demotion nie zadziała, stop + follow-up (bez discovery ClassRealm/SPI w tej zmianie).

## Desired End State

Dokumentowany i testami scharakteryzowany kontrakt: explicit relativePath GA mismatch → FATAL (M4 default) / WARNING (`maven.maven3Personality`) po stronie impl; compat pozostaje WARNING. Semantyka produkcyjna bez zmian poza 1–3 liniowym komentarzem kontraktu. Implementer wie, że nie wolno syncować severity twinów.

## What We're NOT Doing

- Unifikacja severity FATAL↔WARNING między twinami
- Extract package-private stage’ów z hubu `DefaultModelBuilder` (K1)
- Extract / characterization strukturalny `BuildSession` / `DefaultProjectBuilder` (K3)
- Nowe publiczne SPI pipeline
- Migracja deprecated `project.DefaultProjectBuilder` na api ProjectBuilder
- Jacoco % jako cel / baseline w tej zmianie
- Badanie wiring ClassRealm / zewnętrznych `ModelTransformer`
- Korekty `context/map/repo-map.md`
- Improvisacja `configProperties` / SPI w trakcie implementacji, jeśli `userProperties` nie wystarczy do demotion — wtedy follow-up, nie scope creep

## Implementation Approach

Guard-first / characterization przed jakąkolwiek migracją struktury. Kolejność: zapis kontraktu → unit demotion → paired fixture. ITS `MavenITmng8294ParentChecksTest` jest opcjonalnym smoke, nie gate demotion. Nie stosujemy Strangler / Branch by Abstraction — to nie jest migracja struktury.

## Phase 1: Contract note

### Overview

Utrwalić w kodzie i w folderze zmiany, że rozjazd FATAL (impl) vs WARNING (compat) jest celowy; nie sync severity.

### Changes Required:

#### 1. Code comment at mismatchRelativePathAndGA

**File**: `impl/maven-impl/src/main/java/org/apache/maven/impl/model/DefaultModelBuilder.java`

**Intent**: Dodać 1–3 linie komentarza przy `mismatchRelativePathAndGA` (lub bezpośrednio nad logiką `warn = defaultPath || maven3Mode`), że compat twin pozostaje WARNING oraz że personality demotuje do WARNING; nie unifikować severity.

**Contract**: Komentarz tylko; brak zmiany warunków, severity, komunikatów ani sygnatury metody.

#### 2. Local contract note in change folder

**File**: `context/changes/refactor-opportunities/contract-twin-relativepath.md`

**Intent**: Krótka notatka kontraktu z odwołaniem do MNG-5146 / personality (`Constants.MAVEN_MAVEN3_PERSONALITY`) i do istniejącego FATAL unit + luką demotion zamykaną w fazach 2–3.

**Contract**: Markdown w folderze zmiany (nie ADR w upstream `docs/` — go nie ma). Linki do plików/testów z researchu.

### Success Criteria:

#### Automated Verification:

- Plik `context/changes/refactor-opportunities/contract-twin-relativepath.md` istnieje i wymienia FATAL (impl default), WARNING (personality), WARNING (compat)
- Diff produkcyjny ogranicza się do komentarza w `DefaultModelBuilder.java` (brak zmian logiki — weryfikacja review / `git diff`)

#### Manual Verification:

- Komentarz i notatka są zgodne z werdyktem researchu (świadome ograniczenie, nie bug)

**Implementation Note**: After completing this phase and all automated verification passes, pause here for manual confirmation from the human that the manual testing was successful before proceeding to the next phase. Phase blocks use plain bullets — the corresponding `- [ ]` checkboxes for these items live in the `## Progress` section at the bottom of the plan.

---

## Phase 2: Demotion unit (impl)

### Overview

Dodać unit: explicit relativePath GA mismatch + `maven.maven3Personality=true` → WARNING (nie FATAL); zachować istniejący FATAL bez flagi.

### Changes Required:

#### 1. Personality demotion test

**File**: `impl/maven-impl/src/test/java/org/apache/maven/impl/model/ParentCycleDetectionTest.java`

**Intent**: Nowy test obok `testWrongRelativePathInGeneratedPomIsFatalWithMaven3Hint`, ten sam rodzaj scenariusza explicit mismatch, z włączoną personality; asercja WARNING zamiast FATAL. Istniejący FATAL test bez zmian semantycznych.

**Contract**: Użyć `Constants.MAVEN_MAVEN3_PERSONALITY` / wzorca `userProperties` jak w innych testach `DefaultModelBuilderTest`. Jeśli flaga nie demotuje w `ApiRunner` sesji — **stop**, zapisać follow-up; nie badać ClassRealm/SPI ani nie mutować produkcyjnego wiring w tej zmianie.

### Success Criteria:

#### Automated Verification:

- `mvn test -pl impl/maven-impl -am -Dtest=ParentCycleDetectionTest` przechodzi
- Nowy test asertuje WARNING przy personality; istniejący FATAL+hint bez flagi nadal przechodzi

#### Manual Verification:

- Nazwa i asercje nowego testu jasno komunikują „demotion, not unification”

**Implementation Note**: After completing this phase and all automated verification passes, pause here for manual confirmation from the human that the manual testing was successful before proceeding to the next phase.

---

## Phase 3: Paired fixture

### Overview

Ujednolicić layout POM (explicit mismatch) między impl i compat tak, by para testów dokumentowała ten sam scenariusz przy celowo różnych severity (impl FATAL default / WARNING+personality; compat WARNING).

### Changes Required:

#### 1. Align or add paired layout on impl side

**File**: `impl/maven-impl/src/test/java/org/apache/maven/impl/model/ParentCycleDetectionTest.java` (i ewentualnie lokalne zasoby testowe w tym module)

**Intent**: Dopasować layout scenariusza explicit mismatch do kontraktu paired (ten sam kształt co compat MNG-5146 / WRONG_PARENT), bez shared test module.

**Contract**: Skopiowany layout (stringi / struktura katalogów), nie wspólny classpath resource między `maven-impl` a `maven-model-builder`.

#### 2. Compat side paired assertion

**File**: `compat/maven-model-builder/src/test/java/org/apache/maven/model/building/DefaultModelBuilderTest.java`

**Intent**: Upewnić się, że istniejący MNG-5146 explicit-WARNING (lub lekko dopasowany layout) stanowi drugą stronę pary; dodać test tylko jeśli layout trzeba zsynchronizować z impl.

**Contract**: Compat nadal asertuje WARNING; zero zmian w produkcji compat builder.

### Success Criteria:

#### Automated Verification:

- `mvn test -pl impl/maven-impl -am -Dtest=ParentCycleDetectionTest` przechodzi
- `mvn test -pl compat/maven-model-builder -am -Dtest=DefaultModelBuilderTest` przechodzi
- Para pokrywa explicit mismatch: impl FATAL (default) + WARNING (personality); compat WARNING

#### Manual Verification:

- Opcjonalny smoke (nie gate): po `mvn install -pl apache-maven -am -DskipTests`, `mvn verify -Prun-its -pl its/core-it-suite -Dits.test=MavenITmng8294ParentChecksTest`
- Review potwierdza brak shared test-jar i brak unifikacji severity

**Implementation Note**: After completing this phase and all automated verification passes, pause here for manual confirmation from the human that the manual testing was successful before proceeding to the next phase.

---

## Testing Strategy

### Unit Tests:

- Impl FATAL bez personality (istniejący)
- Impl WARNING z personality (nowy)
- Compat WARNING explicit / default relativePath (MNG-5146, ewentualnie dopasowany)

### Integration Tests:

- ITS MNG-8294 — opcjonalny smoke ścieżki FATAL, nie gate demotion personality

### Manual Testing Steps:

1. Przejrzeć komentarz + `contract-twin-relativepath.md` pod kątem „nie sync severity”
2. Potwierdzić, że diff produkcji to tylko komentarz
3. Opcjonalnie odpalić ITS MNG-8294 jako smoke

## Migration Notes

Brak migracji danych / API. Revert = revert PR (testy + komentarz + notatka).

## Open Risks & Assumptions

- `userProperties` w teście unit może nie zasilić `Features.mavenMaven3Personality` (czyta `configProperties`) — wtedy follow-up, nie improvisacja w tej zmianie
- Adopcja `-Dmaven.maven3Personality` w ekosystemie pozostaje unknown (follow-up)
- Jacoco %, ClassRealm/`ModelTransformer`, harmonogram migracji deprecated PB — poza zakresem

## Follow-ups (nie w tej zmianie)

- K1 extract package-private stages
- K3 characterization + extract `BuildSession`
- Pomiar Jacoco hubów
- Research ClassRealm / zewnętrznych transformerów
- Adopcja personality w ekosystemie

## References

- Related research: `context/changes/refactor-opportunities/research.md`
- Prior debt: `context/changes/model-builder-flow-analysis/research.md`
- Impl mismatch: `impl/maven-impl/src/main/java/org/apache/maven/impl/model/DefaultModelBuilder.java:1324–1362`
- Compat WARNING: `compat/maven-model-builder/src/main/java/org/apache/maven/model/building/DefaultModelBuilder.java:1072–1074`
- FATAL unit: `impl/maven-impl/src/test/java/org/apache/maven/impl/model/ParentCycleDetectionTest.java:336–403`
- Compat MNG-5146: `compat/maven-model-builder/src/test/java/org/apache/maven/model/building/DefaultModelBuilderTest.java:111–229`

## Progress

> Convention: `- [ ]` pending, `- [x]` done. Append ` — <commit sha>` when a step lands. Do not rename step titles. See `references/progress-format.md`.

### Phase 1: Contract note

#### Automated

- [ ] 1.1 Contract note file exists with FATAL/WARNING/compat rules
- [ ] 1.2 Production diff is comment-only in DefaultModelBuilder

#### Manual

- [ ] 1.3 Contract wording matches research intentionality verdict

### Phase 2: Demotion unit (impl)

#### Automated

- [ ] 2.1 ParentCycleDetectionTest suite passes
- [ ] 2.2 New demotion test asserts WARNING; existing FATAL test still passes

#### Manual

- [ ] 2.3 Test naming/assertions communicate demotion not unification

### Phase 3: Paired fixture

#### Automated

- [ ] 3.1 ParentCycleDetectionTest suite passes after pairing
- [ ] 3.2 Compat DefaultModelBuilderTest suite passes
- [ ] 3.3 Paired coverage: impl FATAL+WARNING personality, compat WARNING

#### Manual

- [ ] 3.4 Optional ITS MNG-8294 smoke (not demotion gate)
- [ ] 3.5 Review confirms no shared test-jar and no severity unification
