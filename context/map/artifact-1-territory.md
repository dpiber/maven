# Artifact 1 — Territory (Wide Scan)

| | |
|---|---|
| **Repo** | [apache/maven](https://github.com/apache/maven) (analiza na lokalnym `master` @ `61f92e956c`) |
| **Okno** | 2025-10-09 → 2026-10-09 (12 miesięcy) |
| **Historia** | Pełna (nie shallow); `git rev-parse --is-shallow-repository` → `false` |
| **Metoda** | Tylko CLI (`git`, `gh`); bez skilli 10x; bez czytania kodu źródłowego pod analizę |
| **Commity w oknie** | 719–720 łącznie → **~494–496** po filtrach szumu użytych w rankingu |

---

## 1. Audyt szumu

| Kategoria | Występowanie / wzorce | Wpływ na ranking | Po odfiltrowaniu |
|-----------|----------------------|------------------|------------------|
| **Boty / automatyzacja** | ~**215** Dependabot + 1 `github-actions[bot]`; subjects `build: bump …` | **Duży** — bez filtra `root/pom.xml` = #1 (186 touche), `.github/workflows` zawyżone (62→22) | Centra produktowe **bez zmiany** (impl/core/its/cli zostają TOP) |
| **Bump wersji / multi-POM chore** | ~223 subjectów bump (głównie boty); multi-pom naraz ~3 ludzkie | Zawyża root/IT pom.xml, nie logikę runtime | Wniosek o centrum **bez zmiany** |
| **Format / license headers** | ~7 (Spotless, ATR license headers `#12613`) | Niski w module-rank; mega-commit headers wycięty z co-change | Bez zmiany |
| **Rename / move bez zachowania** | ~4 (`Move testing…`, `Move model SPI…`, site rename, IT JUnit4 move) | Pomijalne | Bez zmiany |
| **Lockfile’y** | **Brak** w drzewie / historii okna (Java/Maven) | n/a | — |
| **`target/` / generated-sources** | **Nie commitowane** w oknie (brak ścieżek w `git log --name-only`) | n/a | — |
| **Lokalizacje / i18n** | Brak istotnych `messages_*.properties` w oknie | n/a | — |
| **Grafiki** | 1× `.idea/icon.png` | Zerowy | — |
| **LICENSE/NOTICE** | ~6 commitów (NOTICE year, distro templates, fixture NOTICE) | Pomijalny | — |
| **Snapshoty / golden / IT fixtures** | `its/core-it-suite/src/test/resources/**` w **~63** kept commitach (~438 file-touches); bulk ≥30 plików: ~4 | Zawyża **plikowy** ruch w `its/`; module-rank IT nadal sensowny jako sygnał integracji | IT zostaje „gorące”, ale **nie** jako blast radius produktu |
| **Mega-commity** (co-change) | 14 wyciętych (JPMS multi-module, APT→MD, NIO2 IT 749 plików, license headers, parent bump…) | Bez wycięcia wiązałyby wszystko ze wszystkim | Co-change czytelne dopiero po filtrze |
| **AI agents** | **Nie szum** — **163/496 (~33%)** kept: Claude co-authored-by (~157), Copilot trailer (~5), 1× `copilot-swe-agent` autor | Liczone w aktywności; udział osobno | — |

**Korekta względem surowego TOP:** jedyna istotna pomyłka bez filtra to uznanie **root `pom.xml` / workflows** za centrum produktu. Po filtrze: centrum = `impl/maven-impl`, `impl/maven-core`, `its/`, `impl/maven-cli`.

---

## 2. Synteza terytorium

### Stałe centra vs kampanie vs wygasające

| Rodzaj | Obszary |
|--------|---------|
| **Stałe centra** | `impl/maven-impl` (**model building** — `DefaultModelBuilder`), `impl/maven-core` (project/reactor, plugin realm, lifecycle), `its/core-it-suite` jako lusterko regresji |
| **Kampanie (gł. 2026-Q2–Q3)** | **`mvnup`** w `impl/maven-cli` (~51 commitów); concurrent lifecycle (`BuildPlanExecutor`); consumer POM; polish profili/CI-friendly revisions; API M4 (accessors, SPI, `.mdo`) |
| **Wygasające / drugorzędne** | Klasyczne `compat/maven-model`, settings↔embedder w parach (~0); `.mvn/` (~3); layout starych root-`maven-*` już nie wraca w oknie |
| **Historyczne (nie na HEAD)** | `impl/maven-executor` → osobny projekt [apache/maven-executor](https://github.com/apache/maven-executor) (`#12004`, 2026-05) |

Aktywność miesięczna nierówna: spokój 2026-Q1, skok V–IX 2026 (Maven 4 polish). 2026-Q4 w danych = ~9 dni — nie czytać jako wygasanie.

### TOP sprzężenia co-change (warte uwagi)

- `maven-impl` ↔ `its` (~35% lesser) · `maven-core` ↔ `impl` (~25%) · `core` ↔ `its` (~28%)
- `api/maven-api-core` ↔ impl/core (~46–49% zmian API)
- `compat/maven-model-builder` ↔ `impl` (**~88%**) — legacy builder idzie z nowym modelem
- `compat/maven-embedder` ↔ `cli` (~81%); resolver-provider ↔ impl (~86%)
- Trójka decyzyjna: **api-core + core + impl**
- Hub ręczny: **root `pom.xml`** (breadth ~18 modułów; ~70% zmian ręcznych, nie generator)
- `mvnup` słabo sprzężony z core/model — kampania lokalna + czasem IT

### Przecięcia z wrażliwymi warstwami

| Warstwa | Gdzie aktywność jest realna | Uwaga |
|---------|----------------------------|--------|
| **Runtime** | core lifecycle / project / plugin realm; CLI `LookupInvoker`; embedder M3 | Wysoki blast; Q3 kampania concurrent lifecycle |
| **Dane / config** | settings (`DefaultSettingsBuilder`, credentials provenance `#13265`); toolchains; **profile via model builder** | Nie szukano sekretów w treści — tylko ścieżki/tematy |
| **Public API** | `api/maven-api-*`, `.mdo`; usunięcia z API (`PathTranslator` MNG-8749) | Breaking dla pluginów = **unknown** bez japicmp |
| **Integracje** | Resolver bridge; `maven-compat`; CI; ITS | ITS często **tylko sygnał testowy** (~41 IT-only) |
| **Build repo** | root POM, `apache-maven/` distro, workflows | Niski dramat produktowy; `mvnup` ≠ build Mavena |

### Wygląda groźnie, ale nie jest

- Gęste **fixy w `its/`** i wysoki % fix w IT = domykanie M4, nie rozpad core  
- **Root POM / Dependabot** bez filtra = fałszywe centrum  
- **`mvnup` + „toolchain”** w nazwach plików ≠ codzienny runtime toolchain manager  
- Mega docs/license/NIO2 IT refactors  

### Unknowns → Deep Focus

1. Które zmiany w `api/maven-api-*` / `.mdo` są **binary/behavioral breaking** dla pluginów (japicmp, plugin ecosystem)?  
2. Granica **model building vs project builder** przy parent/BOM/CI-friendly — gdzie dokładnie pęka kontrakt?  
3. **Credentials provenance / mirror merge** — pełny model zaufania settings↔resolver.  
4. Concurrent **lifecycle DAG** — kompletność gwarancji względem klasycznego lifecycle.  
5. Stan i przyszłość **`maven-compat` / embedder** po M4 (co zostaje obowiązkowe).  
6. Transport/resolver pinning (reverty 2.0.24) — wpływ na użytkowników enterprise repo.

---

## 3. Ranking skrót (po filtrze szumu)

**Moduły (commit-touch):** `impl/maven-impl` (149) → `impl/maven-core` (121–122) → `its/core-it-suite` (96–97) → `impl/maven-cli` (87) → `api/maven-api-core` (38) → root `pom.xml` (35) → …

**Pliki:** `DefaultModelBuilder.java` → root `pom.xml` → `PluginUpgradeStrategy.java` → testy modelu / mvnup → `DefaultProjectBuilder` / `DefaultConsumerPomBuilder`.

**Fix pressure (TOP5):** impl ~46% fix, core ~41%, IT ~45%, cli ~49% (mvnup), api-core ~32%; reverty nieliczne (model CI-friendly, `maven.config` quoting, API path quoting). Tematy fixów = edge-case’y M4 / IT, nie flailing CI jako werdykt „psuje się”.

**Forge (`gh`, read-only):** ~322 PR closed-unmerged (12 mies.); ~580 open issues; koncentracja dyskusji: model/profile, lifecycle, realm/classloader, resolver.

---

## 4. Werdykt

Stałe centrum apache/maven w tym oknie to **implementacja modelu + core runtime (reactor/lifecycle/plugins)**, domykane przez **ITS**, z nakładką kampanii **Maven 4** (`mvnup`, API, concurrent lifecycle, consumer POM). Szum botów/POM-bumpów potrafi ukraść TOP — po filtrze obraz jest stabilny. Następny krok mapowania: Deep Focus na unknowns (API break, model↔project, settings/authz, lifecycle DAG), nie na `its/` ani Dependabot.
