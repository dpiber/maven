# Artifact 2 — Structure (layer contracts & blast radius)

| | |
|---|---|
| **Repo** | [apache/maven](https://github.com/apache/maven) (lokalnie @ `4d906620bf`) |
| **Zakres** | Moduły aktywne z artifact-1: `api/maven-api-*`, `impl/{maven-impl,maven-core,maven-cli,maven-support,maven-di,…}`, powiązane `compat/*` (model, model-builder, artifact, compat, embedder, resolver-provider). Bez ITS jako węzła produktowego. |
| **Metoda** | `jdeps` (`-summary`, `-verbose:package`/`class`, `-apionly`) na JAR-ach `*/target/*-4.1.0-SNAPSHOT.jar` (+ kopie bez `module-info` dla JPMS); `mvn dependency:tree -Dscope=compile`. Bez skilli 10x / dependency-cruiser. |
| **Surowce robocze** | `context/map/.work/jdeps/module-edges.tsv` (172 krawędzie); opcjonalny podgraf SVG `focus-chain.svg` (źródło: `focus-chain.dot`) |
| **Ograniczenia** | Statyczny bytecode — **nie** widać Guice/Plexus/DI, refleksji, ClassRealm pluginów, SPI ładowanego w runtime. Zewnętrzni konsumenci API (pluginy ekosystemu) = **unknown**. |

---

## 1. Kontrakty i granice warstw

| Granica | Werdykt | Skrót dowodu |
|---------|---------|--------------|
| `api/*` ↛ `impl` / `compat` | **Trzyma** | `jdeps -verbose:class` na api-*: tylko `org.apache.maven.api.*` |
| `maven-impl` → `api/*`, ↛ `core`/`cli`/`compat` | **Trzyma** | summary + `dependency:tree`; JPMS `provides` SPI + qualified exports do core/cling/compat/embedder |
| `core` / `cli` → `api` + `impl` | **Oczekiwane** | DAG w dół |
| `core`/`cli` → `compat` (model, artifact, plugin-api, model-builder…) | **Przeciek / most M3** | widać też w `-apionly` dla core — typy legacy w powierzchni API |
| `compat` / `embedder` → `core`+`impl` | **Most w górę** (liść) | nie cykl; Ca=0 dla `maven-compat` / `embedder` |

Kierunek warstw: **`api ← impl ← core ← cli`**, z równoległym mostem legacy przez `compat/*` i `maven-support` (v4 IO współdzielone z modelem M3).

---

## 2. Cienkie wejścia vs głębsze centra

| Klasa | Moduły | Ca / Ce (jdeps, reactor) |
|-------|--------|---------------------------|
| **Shallow** | `maven-cli`, `maven-api-cli`, `maven-embedder`, `maven-compat` | cli Ca=1 Ce=21; api-cli Ca=2 Ce=3; embedder/compat Ca=0, wysoki Ce |
| **Deep / load-bearing** | `maven-api-xml` (Ca=18), `maven-support` (14), `maven-api-annotations` (13), `maven-api-core` (11), `maven-api-model` (10), legacy `maven-model`/`artifact` (7) | wysoki fan-in typów |
| **Deep (ścieżka runtime)** | `maven-impl` (Ca=4), `maven-core` (Ca=3) | niski Ca, ale na każdej ścieżce startu (cli/compat/embedder) |
| **Hub mylący** | `mvnup` w cli; `its/core-it-suite` | wysoki buzz w territory, niski/zerowy fan-in produktowy |

---

## 3. Cykle i podejrzane zależności

- **Cykle modułów: brak** (jdeps 172 edges + POM direct deps) — wynik pozytywny.
- **Cykle pakietowe:** gęste *wewnątrz* `maven-core` (lifecycle ↔ project ↔ execution ↔ plugin) i `maven-model-builder` (`model.building` jako hub).
- **Podejrzane mosty:** split package `org.apache.maven.plugin` (plugin-api / core / compat); `model.building` ↔ `model.plugin` ze klasami w model-builder **i** core (`DefaultLifecycleBindingsInjector`); cli → `impl.model` / lifecycle (głębiej niż czysty facade).
- **Nie alarm:** JDK, Guice „not found”, Resolver zewnętrzny, intra-api (`api`↔`api.services`).

---

## 4. Blast radius — TOP cele

### `api/maven-api-core` (kontrakt publiczny M4)

| | |
|---|---|
| **Incoming** | Ca=11: api-cli, api-spi, impl, core, cli, compat, embedder, jline, logging, resolver-provider, settings-builder |
| **Outgoing** | Ce=6: tylko inne `api-*` (annotations, model, plugin, settings, toolchain, xml) — `-apionly` = to samo |
| **Zmiana publicznego kontraktu** | Szeroki blast w reactorze + **unknown** pluginy zewnętrzne / japicmp |
| **Zmiana „wewnętrzna”** | Prawie nie istnieje — cały JAR to API |
| **Testy** | Unit u konsumentów (mock `Session`/services); IT w `its/`; regresja ekosystemu pluginów = unknown |
| **Caution** | Każde usunięcie/zmiana sygnatury w `api.services` / SPI = breaking dla M4. |
| **Unknowns** | japicmp, konsumenci spoza reactora, codegen `.mdo` |

### `impl/maven-impl` (model building / API impl)

| | |
|---|---|
| **Incoming** | Ca=4: **core, cli, compat, embedder** (cała ścieżka runtime) |
| **Outgoing** | Ce=12: prawie cały `api-*` + di + support (`-apionly` bez zewnętrznego Resolver) |
| **Kontrakt publiczny** | Qualified exports + `provides` SPI — zmiana friends/SPI boli core/cli/compat |
| **Implementacja wewnętrzna** | Bezpieczniejsza *jeśli* nie rusza eksportowanych pakietów / SPI; territory: `DefaultModelBuilder` = hub zmian |
| **Testy** | Unit z mockami SPI/resolver; mocne IT model/profile; pluginy = unknown |
| **Caution** | Niski Ca myli — to wąskie gardło pod wszystkimi wejściami. |
| **Unknowns** | RootDetector/ModelObjectProcessor runtime; ClassRealm nie dotyczy wprost, ale wiring DI tak |

### `impl/maven-core` (lifecycle / project / plugins)

| | |
|---|---|
| **Incoming** | Ca=3: cli, compat, embedder |
| **Outgoing** | Ce=22 — api\*, impl, **oraz** compat model/model-builder/artifact/plugin-api/settings…; `-apionly` nadal trzyma compat + impl |
| **Kontrakt publiczny** | Typy M3 w powierzchni + gęste pętle pakietowe → max blast produktowy |
| **Implementacja** | Concurrent lifecycle / realm — lokalne zmiany i tak często tykają project/plugin |
| **Testy** | Unit trudny (mocne mockowanie session/realm); **IT obowiązkowe**; pluginy zewnętrzne = unknown |
| **Caution** | Hub dwuświatowy M4+M3 — „mały” refaktor lifecycle rzadko zostaje w jednym pakiecie. |
| **Unknowns** | Guice/Sisu, ClassRealm, concurrent DAG vs klasyczny lifecycle |

### `impl/maven-support` (wspólny v4 IO)

| | |
|---|---|
| **Incoming** | Ca=14 — impl, core, cli, embedder/compat **oraz** maven-model, model-builder, settings\*, plugin-api, resolver-provider, toolchain\* |
| **Outgoing** | Ce=6: api model/settings/toolchain/metadata/plugin/xml — `-apionly` = to samo |
| **Kontrakt** | Zmiana readers/writers v4 = jednoczesny hit M4 impl i legacy compat jars |
| **Implementacja** | Część generowana — ryzyko driftu ze `.mdo` |
| **Testy** | Unit serializacji + IT modelu; mniej „plugin unknown” niż api-core |
| **Caution** | Wyższy fan-in niż `maven-impl` — cichy load-bearing kernel. |
| **Unknowns** | Udział codegen vs ręczny kod |

### `impl/maven-cli` (wejście)

| | |
|---|---|
| **Incoming** | Ca=1: tylko embedder |
| **Outgoing** | Ce=21; `-apionly` ≈ api\* + **maven-core** (cieńsze niż pełny graf) |
| **Kontrakt publiczny** | Głównie przez `maven-api-cli`; sam cli ma mały fan-in |
| **Implementacja / mvnup** | Contained (nikt poza cli nie importuje `mvnup`); część ścieżek sięga `impl.model` / `impl.standalone` |
| **Testy** | Unit invokerów; IT CLI; mvnup ≠ obowiązkowy IT core |
| **Caution** | Bezpieczniejsze do lokalnych zmian niż core — byle nie psuć bootstrapu/embeddera. |
| **Unknowns** | ClassWorlds start, zewnętrzne tooly na api-cli |

### `compat/maven-model-builder` (most legacy modelu)

| | |
|---|---|
| **Incoming** | Ca=4: core, compat, embedder, resolver-provider |
| **Outgoing** | Ce=7: api-model/xml/annotations, model, artifact, builder-support, **support**; `-apionly` ≈ annotations + builder-support + model |
| **Kontrakt** | Pętle `model.building`↔szprychy + most do `model.plugin` w core |
| **Implementacja** | Zmiana profili/parent resolution = co-change z impl (territory ~88%) |
| **Testy** | Unit builder pipeline + IT modelu; resolver-provider jako sąsiad |
| **Caution** | „Tylko deprecated compat” jest mylące — core nadal zależy w `-apionly`. |
| **Unknowns** | Harmonogram usuwania vs obowiązek w distro |

---

## 5. Wygląda groźnie, ale nie jest

| Sygnał | Dlaczego nie jest centrum struktury |
|--------|-------------------------------------|
| Gęste `its/core-it-suite` | Lustro regresji, nie fan-in produktowy |
| Root POM / Dependabot | Szum territory, poza grafem runtime |
| `mvnup` (wysoki git buzz) | Contained w cli (+ facade api-cli); Ca cli = 1 |
| `maven-compat.jar` (Ca=0) | Liść / deprecated bridge — load-bearing są `support` / `model` / `artifact` / `model-builder` |
| Niski Ca `maven-impl`/`maven-core` | Wąskie gardło ścieżki, nie „mało ważne” |

---

## 6. Unknowns → Deep Focus

1. **Binary/behavioral break** `api/maven-api-*` dla pluginów (japicmp + ekosystem).
2. **Granica model building (impl) vs project builder (core)** przy parent/BOM/CI-friendly — w tym pętla `model.building`↔`model.plugin`.
3. **Credentials / settings ↔ resolver** (poza czystym grafem modułów).
4. **Concurrent lifecycle DAG** vs klasyczne gwarancje (pakiety w core splątane).
5. **Przyszłość obowiązkowego `maven-compat` / embedder** w distro.
6. **Runtime wiring:** ClassRealm, Guice/Sisu, SPI — niewidoczne dla jdeps.
7. **Resolver pinning / transport** (reverty) przez `resolver-provider` ↔ support/model-builder.

---

## 7. Werdykt Kroku 2

Struktura apache/maven w aktywnym obszarze jest **przewidywalnym DAG-iem warstw** z czystym `api/` i czystym `maven-impl`, oraz **świadomymi mostami M3** (`compat` typy w core, `maven-support` jako wspólny IO). Blast radius przy zmianie kontraktu jest największy dla **`maven-api-core` / api-xml / maven-support`**; przy zmianie runtime — dla **`maven-core` + `maven-impl`** mimo niskiego Ca. Wejścia (`cli`, embedder) są cienkie w fan-in; gorące kampanie (`mvnup`, ITS) nie są centrami importów.

Następny krok mapowania: Deep Focus na unknowns powyżej — nie pełny `repo-map.md` ani artifact-3.
