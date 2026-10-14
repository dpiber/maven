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
- `Features.mavenMaven3Personality` czyta `configProperties` sesji (`DefaultModelBuilder` hot path); w `ApiRunner` user props nie zasilają config mapy — demotion w teście przez System property + recreate `ApiRunner.createSession()`, nie `request.userProperties(...)`.

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
- Improvisacja ClassRealm / SPI, jeśli System property / session `configProperties` też nie demotuje — wtedy follow-up, nie scope creep

## Implementation Approach

Guard-first / characterization przed jakąkolwiek migracją struktury. Kolejność: zapis kontraktu → unit demotion → paired verification gate. ITS `MavenITmng8294ParentChecksTest` jest opcjonalnym smoke, nie gate demotion. Nie stosujemy Strangler / Branch by Abstraction — to nie jest migracja struktury.

## Phase 1: Contract note

### Overview

Utrwalić w kodzie i w folderze zmiany, że rozjazd FATAL (impl) vs WARNING (compat) jest celowy; nie sync severity.

### Changes Required:

#### 1. Code comment at mismatchRelativePathAndGA

**File**: `impl/maven-impl/src/main/java/org/apache/maven/impl/model/DefaultModelBuilder.java`

**Intent**: Rozszerzyć istniejący komentarz przy `warn = defaultPath || maven3Mode` (już opisuje WARNING/FATAL + personality) o 1–2 linie: compat twin pozostaje WARNING; nie unifikować severity. Nie dokładać drugiego równoległego bloku.

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

#### 1. Personality demotion test (WRONG_PARENT layout)

**File**: `impl/maven-impl/src/test/java/org/apache/maven/impl/model/ParentCycleDetectionTest.java`

**Intent**: **Source of truth = compat MNG-5146 / WRONG_PARENT** (REAL_PARENT / WRONG_PARENT + child/../pom.xml) — ten sam kształt weryfikowany później w Phase 3 paired gate. Dodać FATAL (default) + demotion WARNING (personality) na tym layoutcie (stringi / lokalne pliki w module). Istniejący shade-style FATAL (`testWrongRelativePathInGeneratedPomIsFatalWithMaven3Hint` / dependency-reduced-pom) **nie** jest bazą demotion — może zostać jako dodatkowa regresja GH-13191 albo zostać nietknięty; nie klonować go pod demotion.

**Contract**: Włączyć personality przez System property `Constants.MAVEN_MAVEN3_PERSONALITY`, potem **lokalnie** wywołać `ApiRunner.createSession()` (nadpisać shared `session` z `@BeforeEach`) — ApiRunner kopiuje system props w momencie create; late `setProperty` po `@BeforeEach` nie zasila istniejącej config mapy. SoT = System property + recreate session; nie mutować mapy z `getConfigProperties()` (unmodifiable view). Cleanup System property w `@AfterEach` / try-finally. **Anti-pattern**: `request.userProperties(...)` (wzorzec z innych flag w `DefaultModelBuilderTest`) — user props nie trafiają do config mapy w `ApiRunner`. Jeśli System/config też nie demotuje — **stop**, zapisać follow-up; nie badać ClassRealm/SPI ani nie mutować produkcyjnego wiring w tej zmianie.

**Asercja demotion**: Jak shade FATAL w tym samym teście — catch `ModelBuilderException` / problemy z builda; wymagaj mismatch WARNING (message / relativePath). **Nie** wymagaj sukcesu builda ani globalnego braku FATAL: bez `ModelResolver` (`ParentProvidingResolver` z MNG-5146) ścieżka może nadal kończyć się Non-resolvable parent FATAL po demotion mismatch. Para z compat to severity/layout, nie „successful fallback build”. Nie dodawać resolvera pod ApiRunner w tej fazie.

### Success Criteria:

#### Automated Verification:

- `mvn test -pl impl/maven-impl -am -Dtest=ParentCycleDetectionTest` przechodzi
- Nowy test (WRONG_PARENT): FATAL mismatch bez personality oraz WARNING przy personality (nie wymaga sukcesu builda / braku FATAL po demotion)
- Istniejący shade FATAL+hint nadal przechodzi

#### Manual Verification:

- Nazwa i asercje nowego testu jasno komunikują „demotion, not unification”

**Implementation Note**: After completing this phase and all automated verification passes, pause here for manual confirmation from the human that the manual testing was successful before proceeding to the next phase.

---

## Phase 3: Paired verification gate

### Overview

Human/verification gate po Phase 2: potwierdzić, że para impl↔compat dokumentuje ten sam explicit-mismatch przy celowo różnych severity. **Zero nowego kodu** — layout WRONG_PARENT i demotion są już SoT z Phase 2; MNG-5146 po stronie compat bez zmian.

### Changes Required:

#### 1. Review paired layout (already on WRONG_PARENT from Phase 2)

**File**: `impl/maven-impl/src/test/java/org/apache/maven/impl/model/ParentCycleDetectionTest.java` (i ewentualnie lokalne zasoby testowe w tym module)

**Intent**: Layout WRONG_PARENT jest już SoT z Phase 2 (FATAL default + WARNING personality). Phase 3 **nie** wybiera ponownie między shade a WRONG_PARENT i **nie** zastępuje demotion — tylko weryfikuje parę. Shade FATAL (jeśli nadal obecny) pozostaje poza kanoniczną parą.

**Contract**: Zero drugiej decyzji o layoutcie; nie wspólny classpath resource między `maven-impl` a `maven-model-builder`; zero nowych testów / layoutów w tej fazie.

#### 2. Compat side paired assertion (existing MNG-5146 — no code)

**File**: `compat/maven-model-builder/src/test/java/org/apache/maven/model/building/DefaultModelBuilderTest.java`

**Intent**: Istniejący MNG-5146 jest kanoniczną stroną compat; tylko re-run / review asercji WARNING. Nie zmieniać kształtu POM ani kodu testu.

**Contract**: Compat nadal asertuje WARNING; zero zmian w produkcji compat builder; **zero przebudowy layoutu MNG-5146**.

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
- Compat WARNING explicit / default relativePath (MNG-5146 — bez przebudowy layoutu)

### Integration Tests:

- ITS MNG-8294 — opcjonalny smoke ścieżki FATAL, nie gate demotion personality

### Manual Testing Steps:

1. Przejrzeć komentarz + `contract-twin-relativepath.md` pod kątem „nie sync severity”
2. Potwierdzić, że diff produkcji to tylko komentarz
3. Opcjonalnie odpalić ITS MNG-8294 jako smoke

## Migration Notes

Brak migracji danych / API. Revert = revert PR (testy + komentarz + notatka).

## Open Risks & Assumptions

- System property / session `configProperties` powinny zasilić demotion; jeśli mimo to nie — follow-up (ClassRealm/SPI poza zakresem), nie improvisacja w tej zmianie; pamiętać o cleanup System property w teście
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

- [x] 1.1 Contract note file exists with FATAL/WARNING/compat rules — 9d643a181c
- [x] 1.2 Production diff is comment-only in DefaultModelBuilder — 9d643a181c

#### Manual

- [x] 1.3 Contract wording matches research intentionality verdict — 9d643a181c

### Phase 2: Demotion unit (impl)

#### Automated

- [x] 2.1 ParentCycleDetectionTest suite passes — ebcd0c331b
- [x] 2.2 New demotion test asserts mismatch WARNING on WRONG_PARENT (not build success / no-FATAL) — ebcd0c331b
- [x] 2.4 New FATAL default assertion on WRONG_PARENT without personality; existing shade FATAL still passes — ebcd0c331b

#### Manual

- [x] 2.3 Test naming/assertions communicate demotion not unification — ebcd0c331b

### Phase 3: Paired verification gate

#### Automated

- [x] 3.1 ParentCycleDetectionTest suite passes after pairing — b666e22dc7
- [x] 3.2 Compat DefaultModelBuilderTest suite passes — b666e22dc7
- [x] 3.3 Paired coverage: impl FATAL+WARNING personality, compat WARNING — b666e22dc7

#### Manual

- [x] 3.4 Optional ITS MNG-8294 smoke (not demotion gate) — b666e22dc7
- [x] 3.5 Review confirms no shared test-jar and no severity unification — b666e22dc7
