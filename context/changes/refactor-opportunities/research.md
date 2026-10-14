---
date: 2026-10-14T10:04:20+0000
researcher: Dominik Piber
git_commit: 9789a04220cfa0c5e59fc38cfc9f0c7c9a43ed20
research_commit: 023575b599375d1c012d60e1f56d3a223cb1eb3f
branch: docs/refactor-opportunities
repository: apache/maven
topic: "Refactor opportunities from model-builder-flow-analysis debt"
tags: [research, codebase, model-builder, refactor-opportunities, maven-impl, maven-core, compat, verified]
status: complete
last_updated: 2026-10-14
last_updated_by: Dominik Piber (ast-grep verify)
---

# Research: Refactor opportunities (ModelBuilder path)

**Date**: 2026-10-14T10:04:20+0000  
**Researcher**: Dominik Piber  
**Git Commit**: `9789a04220cfa0c5e59fc38cfc9f0c7c9a43ed20` (verify; research: `023575b599375d1c012d60e1f56d3a223cb1eb3f`)  
**Branch**: `docs/refactor-opportunities`  
**Repository**: apache/maven  

## Research Question

Na podstawie `context/changes/model-builder-flow-analysis/research.md` (prior długu / ryzyk strukturalnych ścieżki budowania modelu POM): wypisać każdy odnotowany problem, sklasyfikować kandydatów refaktoru strukturalnego vs wejścia do kosztu, zbadać kandydatów (obecny kształt / intencjonalność / wykonalność) i zamknąć rankingiem 2–3 najmocniejszych opportunity — bez decyzji produktowej i bez zmian w kodzie.

## Summary

Trzy problemy z prioru są **kandydatami strukturalnymi**: (K1) hub `impl…DefaultModelBuilder` (~3451 LOC), (K2) connascence / drift twin M3↔M4 (w tym relativePath personality), (K3) koncentracja orkiestracji reactora w `project.DefaultProjectBuilder`. Luki testowe, Jacoco %, luki mapy/docs i unknown SPI/ClassRealm **nie** są kandydatami — wchodzą w koszt i osłony.

Intencjonalność: K2 i K3 to **świadome ograniczenia** (guard, nie przebudowa kontraktu). K1 to **świadomy orchestrator** z **akumulacyjnym** rozmiarem pliku. Najmocniejsza ścieżka inkrementalna: najpierw guardy K2, potem package-private extract stage’ów K1, potem characterization + extract w `BuildSession` (K3). Unifikacja severity twinów = redesign domeny (poza zakresem; M4L5).

## Lista problemów z prioru i klasyfikacja

Źródło: `context/changes/model-builder-flow-analysis/research.md` (Technical debt §1–5, Detailed Findings, Open Questions, Architecture Insights). Ustalenia prioru przyjęte; poniżej tylko klasyfikacja pod refaktor struktury.

| # | Problem (jak w priorze) | Klasyfikacja | ID |
|---|-------------------------|--------------|-----|
| 1 | Hotspot `DefaultModelBuilder` ~3450 LOC — jedna klasa łączy file/effective/parent/profile/CI-friendly | **KANDYDAT** | K1 |
| 2 | Connascence M3↔M4 bez krawędzi kompilacji; asymetria co-change; ręczne uzgodnienie znaczenia | **KANDYDAT** | K2 |
| 3 | Drift behawioralny parent relativePath: compat WARNING vs impl FATAL (+ `maven3Personality`) | **KANDYDAT** (ten sam wątek co #2; osobna soczewka „personality”) | K2 |
| 4 | Luki unit: cienki `DefaultProjectBuilderTest`; brak demotion `maven3Personality`; brak paired twin; Jacoco % unknown | **nie kandydat** — wejście do kosztu / osłon | — |
| 5 | Blast radius: api-core/spi + core ProjectBuilder kruche semantycznie (compile-checked) | **nie osobny kandydat** — opis sprzężenia; oddziałuje na koszt K1/K3 | — |
| 6 | Twin compat + resolver-provider kruche (brak POM→impl) | **KANDYDAT** (składnik K2) | K2 |
| 7 | `DefaultProjectBuilder` + RequestType — granica reactor vs consumer, materializacja `MavenProject` | **KANDYDAT** | K3 |
| 8 | `.mdo` / `maven-support` = tańsze sprzężenie niż twin | **nie kandydat** — kontekst kosztu (preferencja vs ręczny twin) | — |
| 9 | ITS jako lustro (~42% co-change) — nie hotspot produktowy | **nie kandydat** — osłona regresji | — |
| 10 | Korekty mapy (kierunek call-graph; compat nie na każdym `mvn`; dwa `DefaultProjectBuilder`) | **nie kandydat** — dokumentacja / onboarding | — |
| 11 | SPI `ModelParser`/`ModelTransformer` / ClassRealm / ServiceLoader = unknown | **nie kandydat** — lukа badawcza, nie struktura do naprawy w tej zmianie | — |
| 12 | Poza ścieżką: `mvnup`, concurrent lifecycle, gęste fixy w ITS same w sobie | **nie kandydat** — prior §5 | — |

**Kandydaci do badania:** K1, K2, K3.

---

## Detailed Findings

### K1 — Hub `DefaultModelBuilder` (koncentracja pipeline’u)

#### Obecny kształt

- Klasa: `impl/maven-impl/.../impl/model/DefaultModelBuilder.java` — **3451** LOC (`wc -l` na HEAD tej sesji) (**evidence**).
- Implementuje `org.apache.maven.api.services.ModelBuilder`; `@Named` + `@Inject` ctor z 18 collaboratorami SPI (`ModelProcessor`, `ModelValidator`, interpolator, inheritance, profiles, …) oraz `List<ModelTransformer>` = **19** parametrów ctora (**evidence**: `:134–209`; raport: ~18 + List).
- Entry: `ModelBuilderSessionImpl.build` (`:227`) — `BUILD_PROJECT` → `buildBuildPom()`, inaczej `buildEffectiveModel` (**evidence**: `:241–247`; prior).
- Masa logiki w nested `ModelBuilderSessionState` (file load, effective, parent frames, CI-friendly w `doReadFileModel`, import depMgmt, cache) (**evidence**: struktura `:216+`, `:265+`; metody m.in. `mismatchRelativePathAndGA` `:1324`, `buildBuildPom` / `buildEffectiveModel`, `doReadFileModel` — potwierdzone lokalizacje przez worker shape).
- **Istniejące szwy:** stage interfaces w `api/maven-api-spi/.../services/model/` + `Default*` helpery poza hubem; SPI `ModelTransformer` na file/raw/effective; `RequestType` w `ModelBuilderRequest` (**evidence**). Brak potrzeby nowego publicznego SPI do extractu package-private (**inference**).

#### Intencjonalność

- **Werdykt:** świadome ograniczenie co do *jednego orchestratora pipeline’u*; przypadkowa złożoność co do *rozmiaru monolitu pliku*.
- 2009 (`26a35db0c7`): ModelBuilder „hosts most of the logic from the project builder”; ProjectBuilder „simply layered on top” (**evidence**: subject commita).
- Site compat model-builder: hub „manages the steps sequence” — kroki jako osobne komponenty (**evidence**: worker history / site).
- MNG-8084: nowy builder M4 urodzony już jako duży orchestrator; dalszy wzrost LOC = akrecja feature’ów (mixiny, parent checks, CI-friendly, cache) bez ADR „musi zostać jednym plikiem” (**evidence** growth commits; **inference** akrecja; **unknown** czy maintainerzy formalnie akceptują 3.4k LOC).

#### Wykonalność

- **Docelowy kształt (nazwa):** extract package-private pipeline stages (metody session-state → typy w tym samym pakiecie; hub deleguje).
- **Ścieżka:** characterization / gap-fill unit → move method → type → hub thin facade; bez publikacji nowego SPI; revert = revert PR (Mikado + istniejący Branch-by-Abstraction na stage’ach).
- **Osłony:** `DefaultModelBuilderTest` — **40** `@Test` (**evidence**); `ParentCycleDetectionTest`, `ParentTraversalTest`; ITS `MavenITmng8294ParentChecksTest`; root Surefire/Failsafe + profil `jacoco` (opt-in); ITS `-Prun-its` (**evidence** worker feasibility / prior).
- **Pierwszy prerekwizyt:** guard unit na explicit relativePath mismatch → FATAL oraz osobno `maven3Personality` → WARNING (dziś unit sprawdza FATAL+hint, nie demotion — **evidence**: `ParentCycleDetectionTest.java:394–403`; gap prior).
- **Blast radius:** package-private move ≈ głównie `maven-impl` + unit; zmiana powierzchni API/`ModelBuilderRequest` → must-change-together api-core + spi + impl + core + tests (+ często ITS) (**evidence** prior blast; **inference** dla extract lokalnego).
- **Stop M4L5?** Nie — o ile nie zmienia się semantyki parent/profile/CI-friendly.

### K2 — Twin M3↔M4 (connascence + relativePath personality)

#### Obecny kształt

- Twin: `compat/maven-model-builder/.../DefaultModelBuilder.java` — **1465** LOC, `@Deprecated(since = "4.0.0")` (**evidence**: `:99–100`, `wc -l`).
- Hot path reactora woła **tylko** `api.services.ModelBuilder` z `project.DefaultProjectBuilder` (**evidence**: importy `:72–75`, prior; zero importów/`FQN` `model.building.(Default)?ModelBuilder` w maven-core main — prior; **doprecyzowanie**: nadal importy innych typów `model.building.*` np. `ModelProblem` / `ModelSource` `:93–96`).
- relativePath GA mismatch:
  - **impl** `:1324–1362`: `warn = defaultPath || maven3Mode`; inaczej **FATAL**; komentarz „preserve historical lenient behaviour” (**evidence**).
  - **compat** `:1072–1074` (raport: `:1071–1074`): zawsze **WARNING** (**evidence**; `:1071` = `problems.setSource`).
- `maven.maven3Personality` / `Features.mavenMaven3Personality` — po stronie impl; w `compat/maven-model-builder` brak personality (**evidence**: rg).
- Brak krawędzi kompilacji compat→impl; ręczne uzgodnienie semantyki (prior co-change A→B wysokie gdy compat się rusza; reverse niskie) (**evidence** prior; nie przeliczane w tej sesji).

#### Intencjonalność

- **Werdykt:** **świadome ograniczenie** (load-bearing personality / kontrakt M3 vs M4). Guard, nie „naprawa” severity.
- MNG-8294 (`fee69f2f89`): consistency checks parenta — eskalacja FATAL w M4 (**evidence**).
- `#2380` / `5000b5933f`: `maven.maven3Personality` — „behave like … Maven 3” (`Constants`) (**evidence**).
- MNG-5146 (`4309183ac9` / #13086): FATAL tylko przy explicit `<relativePath>` bez personality; compat pozostaje WARNING jako terytorium M3 (**evidence**: subject + kod/komentarz `:1358–1361`).
- Hint w FATAL o `-Dmaven.maven3Personality` jako temporary workaround (`f47804c07e` / worker) (**evidence** message `:1353–1354`).

#### Wykonalność

- **Docelowy kształt (nazwa):** paired characterization tests + personality ADR/doc guard.
- **Ścieżka:** (1) udokumentować celowy rozjazd → (2) paired unit (impl FATAL default / WARNING+personality; compat WARNING — częściowo już w compat test MNG-5146) → (3) opcjonalnie shared fixture / checklist review — **bez** unifikacji severity → revert = usunięcie guardów.
- **Osłony:** compat unit MNG-5146 WARNING; ITS MNG-8294 FATAL path; **brak** unit demotion personality w impl; **brak** paired twin na jednej fixture (**evidence** prior + rg testów).
- **Pierwszy prerekwizyt:** unit impl: explicit mismatch + `maven.maven3Personality=true` → WARNING (oraz bez flagi → FATAL) + krótki zapis „nie sync severity twinów”.
- **Blast radius:** guard-only = testy + docs. Wyrównanie FATAL↔WARNING = zmiana semantyki produktu + ITS/ekosystem — nie inkrementalny refactor struktury.
- **Stop M4L5?** **Tak**, jeśli „naprawa” = unifikacja severity / redesign parent resolution. Guard = w zakresie.

### K3 — `DefaultProjectBuilder` (orkiestracja reactora + materializacja)

#### Obecny kształt

- `impl/maven-core/.../project/DefaultProjectBuilder.java` — **1420** LOC, `@Deprecated`, implements legacy `ProjectBuilder` (**evidence**: `wc -l`, `:112–115` (raport: `:108–115`)).
- Nested `BuildSession`: `ModelBuilderSession`, `projectIndex`, parent frames; `BUILD_PROJECT` → `modelBuilderSession.build` → `initProject` / `finishProjectInitialization` (**evidence**: prior `:665–710`, `:679`; worker shape; także `modelBuilder.newSession()` `:351`).
- Facade `internal.impl.DefaultProjectBuilder` deleguje do legacy builder (**evidence** prior; `builder.build` `:90` / `:94`).
- Call-site reactor: `DefaultProjectsSelector` → `build(files, recursive, …)` (**evidence** worker; `:61`).

#### Intencjonalność

- **Werdykt:** **świadome ograniczenie** — warstwa „na ModelBuilderze” od 2009; nie wciągać materializacji z powrotem do ModelBuilder.
- Ten sam commit `26a35db0c7`: PB layered on top (**evidence**).
- Deprecation API (`8ce8cd0ccc` / worker) nie usuwa roli reactora (**evidence** / **inference** na podstawie nadal żywego call-graph).
- **Unknown:** docelowy moment pełnej migracji reactora na non-deprecated `api.services.ProjectBuilder`.

#### Wykonalność

- **Docelowy kształt (nazwa):** extract package-private `BuildSession` orchestration stages (request assembly vs materializacja vs parent wiring).
- **Ścieżka:** characterization reactor `build(List, recursive)` → wydzielić etapy → hub zostaje fasadą; revert PR.
- **Osłony:** `DefaultProjectBuilderTest` — **3** `@Test`, tylko `extractProjectId` (**evidence**); grubsze: `ProjectBuilderTest`, `DefaultMavenProjectBuilderTest`, `PomConstructionTest`; ITS project-builder-related; te same profile CI (**evidence** worker).
- **Pierwszy prerekwizyt:** characterization unit/IT-level na drzewo `ProjectBuildingResult` / collected projects **przed** move.
- **Blast radius:** extract w `maven-core` umiarkowany; zmiana `RequestType`/drzewa wyniku → api + impl ModelBuilder + ITS.
- **Stop M4L5?** Nie przy extract orchestration. Tak przy zmianie kontraktu reactor↔model lub scaleniu twinów ProjectBuilder.

---

## Refactor opportunities

Ranking = propozycja pod późniejsze planowanie (osobna sesja). Nie jest decyzją ani prośbą o wybór.

### 1. K2 — paired characterization + personality ADR guard

| | |
|--|--|
| **Obecny → docelowy** | Ręcznie utrzymywany twin z celowym FATAL vs WARNING → udokumentowany kontrakt + paired testy (bez unifikacji severity) |
| **Dlaczego #1** | Najniższy koszt zmiany przy wysokim koszcie błędu (fałszywy „fix twin” psuje kontrakt M4/M3). Historia (`4309183ac9`, `5000b5933f`) mówi: guard, nie rebuild. |
| **Blast radius** | Testy + docs; zero zmian semantyki produkcyjnej przy ścieżce guard |
| **Ścieżka** | ADR/komentarz kontraktu → unit demotion personality → paired fixture (impl vs compat) → opcjonalnie checklist PR |
| **Prerekwizyt** | Unit: explicit mismatch + `maven.maven3Personality` → WARNING |

### 2. K1 — extract package-private pipeline stages z hubu ModelBuilder

| | |
|--|--|
| **Obecny → docelowy** | Monolit `DefaultModelBuilder` / `ModelBuilderSessionState` (~3451 LOC) → hub + package-private stage types; istniejące SPI stage’ów bez nowego publicznego API |
| **Dlaczego #2** | Największy lokalny dług strukturalny na hot path; istniejący Branch-by-Abstraction (collaboratorzy SPI) obniża koszt extractu względem „nowej architektury”. Koszt zmiany > K2, ale dług behawioralny przy każdej edycji hubu jest najwyższy (prior §1). |
| **Blast radius** | Przy extract lokalnym: `maven-impl` + unit; przy ruszaniu API: pełny must-change-together z prioru |
| **Ścieżka** | Guard mismatch/personality → extract jednego stage (np. parent mismatch / fragment file-model) → kolejne Mikado steps |
| **Prerekwizyt** | Te same guardy co K2 (characterization przed ruchem kodu) |

### 3. K3 — extract package-private stages z `BuildSession` (ProjectBuilder)

| | |
|--|--|
| **Obecny → docelowy** | Zagnieżdżona orkiestracja request + materializacja `MavenProject` w `DefaultProjectBuilder` → package-private stages; zachowana warstwa „na ModelBuilderze” |
| **Dlaczego #3** | Świadoma, potrzebna warstwa (2009); dług jest w gęstości `BuildSession` i cienkim nazwanym unit, nie w samym istnieniu PB. Niższy priorytet niż K1, dopóki nie ma characterization reactor path. |
| **Blast radius** | Umiarkowany w `maven-core`; wysoki przy zmianie RequestType / drzewa |
| **Ścieżka** | Characterization `build(List, recursive)` → extract request assembly / materialization / parent wiring |
| **Prerekwizyt** | Characterization wokół materializacji (nie tylko `extractProjectId`) |

### Rozważone i odrzucone (jako opportunity refaktoru w tej zmianie)

| Pozycja | Dlaczego odrzucona |
|---------|-------------------|
| Unifikacja severity relativePath (FATAL↔WARNING) między twinami | Celowa personality; prawdziwa „naprawa” = redesign kontraktu domeny → M4L5, nie struktura |
| Samo dołożenie testów / Jacoco jako „kandydat” | Nie zmienia struktury kodu; zostaje prerekwizytem / kosztem |
| Korekty `repo-map.md` / docs call-graph | Dokumentacja, nie refaktor |
| Scalenie materializacji ProjectBuilder z powrotem do ModelBuilder | Odwraca decyzję z 2009; zwiększa hub K1 |
| Nowe publiczne SPI pipeline tylko po to, by „rozbić” hub | Zbędne — stage interfaces już są; zwiększa blast api |
| `mvnup` / concurrent lifecycle / ClassRealm SPI wiring | Poza ścieżką prioru lub unknown badawczy — nie opportunity z tej listy długu |

---

## Code References

- `impl/maven-impl/.../model/DefaultModelBuilder.java:134–209,241–247,1324–1362` — hub, RequestType branch, relativePath personality
- `compat/maven-model-builder/.../DefaultModelBuilder.java:99–100,1072–1074` (raport: `:1071–1074`) — deprecated twin, always WARNING
- `impl/maven-core/.../project/DefaultProjectBuilder.java:72–75,112–115,665–710,679` — reactor → api ModelBuilder → materializacja; `@Deprecated`+class `:112–115` (raport: `:108–115`)
- `impl/maven-impl/.../model/ParentCycleDetectionTest.java:394–403` — FATAL+hint, brak demotion unit
- `impl/maven-impl/.../model/DefaultModelBuilderTest.java` — 40 `@Test`
- `impl/maven-core/.../project/DefaultProjectBuilderTest.java` — 3 `@Test` (`extractProjectId`)
- `its/core-it-suite/.../MavenITmng8294ParentChecksTest.java` — parent ITS
- Commits: `26a35db0c7` (PB/MB split), `fee69f2f89` (MNG-8294), `5000b5933f` (personality), `4309183ac9` (MNG-5146)

## Architecture Insights

- DAG `api ← impl ← core ← cli` z równoległym `compat/*` trzyma się; twin nie jest na hot path `mvn`, ale wymaga ręcznej semantyki.
- Orchestrator ModelBuilder + warstwa ProjectBuilder to decyzja nośna z 2009; refaktor wartościowy to **rozmiar/szwy wewnętrzne**, nie usuwanie warstw.
- Preferuj „guard, nie przebudowa” tam, gdzie git pokazuje personality (K2) lub świadomy podział PB/MB (K3).

## Historical Context (from prior changes)

- `context/changes/model-builder-flow-analysis/research.md` — prior długu, trace, test gaps, blast radius (budowane na, nie wyprowadzane na nowo).
- `context/map/repo-map.md` §3–4 — strefa 1 + most M3↔M4 + co-change priors.
- `context/foundation/lessons.md` — nieobecny.

## Related Research

- `context/changes/model-builder-flow-analysis/research.md`

## Open Questions

1. Jacoco line/branch % dla hubów — nadal **unknown** (profil istnieje, nie uruchamiany).
2. Pełny runtime wiring ClassRealm / zewnętrznych `ModelTransformer` — **unknown** (prior).
3. Ile CI/ekosystemu polega na `-Dmaven.maven3Personality` — **unknown**.
4. Formalny ADR w repo — brak; intencja w commitach/site/javadoc (**evidence** braku `docs/` ADR).
5. Harmonogram migracji reactora z deprecated `project.DefaultProjectBuilder` na api ProjectBuilder — **unknown**.

## Inspected scope / method

- Prior przeczytany w całości; mapa § strefa 1 jako prior.
- Trzech sub-agentów Explore (kształt / historia / wykonalność) na K1–K3; synteza w parent context.
- Spot-check: `mismatchRelativePathAndGA` `:1324–1362`, compat WARNING `:1072–1074` (raport: `:1071–1074`), `wc -l` hubów, liczby `@Test`, commity `4309183ac9` / `5000b5933f` / `fee69f2f89` / `26a35db0c7`.
- Zero zmian w kodzie produkcyjnym; zero buildów/testów uruchomionych.
- Weryfikacja strukturalna (ast-grep + rg) na HEAD `9789a04220` — sekcja poniżej; ranking i werdykty intencjonalności nietknięte.

## Weryfikacja twierdzeń (ast-grep)

Weryfikacja na `9789a04220cfa0c5e59fc38cfc9f0c7c9a43ed20` (research baseline: `023575b599…`). Wzorce `-l java`; każde zero AST potwierdzone `rg`. Sekcji „Refactor opportunities (ranked)” i werdyktów intencjonalności **nie zmieniono**.

| Twierdzenie | Werdykt | Dowód (plik:linia) | Metoda (wzorzec/reguła) |
|-------------|---------|--------------------|-------------------------|
| `impl…DefaultModelBuilder` = **3451** LOC | potwierdzone | `impl/.../model/DefaultModelBuilder.java` (`wc -l` → 3451) | `wc -l` |
| `compat…DefaultModelBuilder` = **1465** LOC | potwierdzone | `compat/.../building/DefaultModelBuilder.java` (`wc -l` → 1465) | `wc -l` |
| `project…DefaultProjectBuilder` = **1420** LOC | potwierdzone | `impl/.../project/DefaultProjectBuilder.java` (`wc -l` → 1420) | `wc -l` |
| Para lustrzana: dwa `DefaultModelBuilder` (impl vs compat) | potwierdzone | impl `:136`; compat `:100` | `rg 'class DefaultModelBuilder'`; ast-grep class-decl na dużym pliku pusty → fallback `rg` |
| Para lustrzana: dwa `DefaultProjectBuilder` (legacy `project` + facade `internal.impl`) | potwierdzone | `project/…:115`; `internal/impl/…:57` | `rg 'class DefaultProjectBuilder'`; `ast-grep -p 'class BuildSession implements AutoCloseable'` (nested) |
| impl `implements ModelBuilder`; `@Named` + `@Inject` ctor | potwierdzone | `@Named` `:134`; class `:136`; `@Inject` ctor `:169–209` | `rg`; parametry: python count ctor = **19** (18 SPI + `List<ModelTransformer>`; raport: ~18 + List) |
| Entry `BUILD_PROJECT` → `buildBuildPom()`, else `buildEffectiveModel` | potwierdzone | `DefaultModelBuilder.java:241–247` (`build` method `:227`) | `rg 'BUILD_PROJECT\|buildBuildPom\|buildEffectiveModel'`; `ast-grep -p 'modelBuilderSession.build($A)'` (call-site’y) |
| `mismatchRelativePathAndGA`: `warn = defaultPath \|\| maven3Mode`; else FATAL | potwierdzone | `:1324–1362` (`warn` `:1361`; ternary Severity `:1362`) | `ast-grep -p 'boolean warn = defaultPath \|\| maven3Mode'`; `ast-grep -p 'add(warn ? Severity.WARNING : Severity.FATAL, $$$A)'`; call-site’y `ast-grep -p 'mismatchRelativePathAndGA($$$)'` → `:1899`, `:2309` |
| compat relativePath mismatch → zawsze WARNING | potwierdzone / doprecyzowane | `:1072–1074` (raport: `:1071–1074`; `:1071` = `setSource`) | `rg 'Severity.WARNING'` + odczyt bloku; ast-grep `problems.add(new ModelProblemCollectorRequest(Severity.WARNING, $$$A))` na pliku — pusty (składnia multi-line) → `rg` |
| `Features.mavenMaven3Personality` po stronie impl; **zero** w compat | potwierdzone | impl `:1326–1327`, `:2007`; `Features.java:39`; compat: **0** hits | `ast-grep -p 'Features.mavenMaven3Personality($$$A)'`; `rg` w `compat/maven-model-builder` → ZERO |
| Hot path: `project.DefaultProjectBuilder` importuje **api** `ModelBuilder` (`:72–75`); **zero** compat `ModelBuilder` w maven-core main | potwierdzone / doprecyzowane | api imports `:72–75`; `rg` ZERO `model.building.(Default)?ModelBuilder` w `maven-core/src/main`; **doprecyzowanie**: inne typy `model.building.*` tak (`:93–96` ModelProblem/Source) — nie twin builder | `rg 'import org.apache.maven.api.services.ModelBuilder'`; `rg` ZERO na twin; **unknown** wiring SPI/ClassRealm (poza AST) |
| `modelBuilderSession.build` na ścieżce reactora | potwierdzone / doprecyzowane | produkcja `:492`, `:679` (+ test `:1733`); `BUILD_PROJECT` request `:674`; `newSession` `:351` | `ast-grep -p 'modelBuilderSession.build($A)'` |
| Nested `BuildSession` → `initProject` / materializacja (`:665–710`) | potwierdzone | `BuildSession` `:317`; `build(File,recursive)` `:665`; `initProject` `:699`/`:773`; `finishProjectInitialization` `:511`/`:783` | `ast-grep -p 'class BuildSession implements AutoCloseable'`; `rg` |
| Facade `internal.impl.DefaultProjectBuilder` deleguje do legacy | potwierdzone | field `org.apache.maven.project.ProjectBuilder builder` `:59`; `builder.build` `:90`, `:94` | `rg` / odczyt |
| Reactor call-site: `DefaultProjectsSelector` → `projectBuilder.build(files, recursive, …)` | potwierdzone | `DefaultProjectsSelector.java:61` | `ast-grep -p 'projectBuilder.build($F, $R, $REQ)'` |
| `DefaultModelBuilderTest` — **40** `@Test` | potwierdzone | `DefaultModelBuilderTest.java` (40× `@Test`) | `rg -c '@Test'` / `^\s*@Test\b` |
| `DefaultProjectBuilderTest` — **3** `@Test`, tylko `extractProjectId` | potwierdzone | `:51`, `:77`, `:101`; refleksja `extractProjectId` | `rg -c '@Test'`; `rg 'extractProjectId'` |
| `ParentCycleDetectionTest:394–403` — FATAL+hint; brak unit demotion personality | potwierdzone | `:394–403`; brak `maven3Personality=true` w unit demotion (`rg` testów impl) | `rg` + odczyt; **unknown** czy ITS pokrywa demotion poza zakresem tej weryfikacji |
| `@Deprecated(since="4.0.0")` na compat MB i legacy PB | potwierdzone / doprecyzowane | compat MB `:99–100`; legacy PB `:112–115` (raport: `:108–115`) | `rg '@Deprecated\|class Default'` |

### Wpływ na ranking (tylko adnotacja)

Żadne twierdzenie ranking-bearing nie zostało **obalone**. Doprecyzowania (linie WARNING/Deprecated, 19 vs „~18” params, dodatkowy call-site `:492`, importy nie-twin `model.building.*` w core) **nie podważają** kolejności K2 → K1 → K3.

Jeśli na etapie planowania ktoś czyta „zero compat w maven-core” zbyt szeroko: zawęzić do **ModelBuilder twin** (inne typy `model.building` zostają) — **do decyzji na etapie planowania** (blast/cost K2/K3, nie zmiana pozycji w tym raporcie).

SPI / ClassRealm / ServiceLoader wiring `ModelTransformer` — nadal **unknown** (nie mylić z „brak wywołań” w AST).
