# Artifact 3 — Contributors (Deep Focus: knowledge map)

| | |
|---|---|
| **Repo** | [apache/maven](https://github.com/apache/maven) (lokalnie @ `ccffd508be`) |
| **Wybrany obszar** | `impl/maven-impl` (model building / API impl — bez `target/` / generated-sources) |
| **Okno** | 2025-10-09 → 2026-10-09 (12 miesięcy; jak artifact-1) |
| **Metoda** | CLI `git log` + `gh` (read-only); bez skilli 10x |
| **Filtry** | Boty jako autor: **0** w module; AI jako autor: **0**; człowiek+AI co-author (gł. Claude): **50/155 (~32%)** — liczone przy człowieku; AI udział osobno |
| **Miara rankingu** | Liczba commitów dotykających ścieżki (nie git blame linii) |

---

## 1. Dlaczego ten obszar

Z artifact-1: stałe centrum (#1 commit-touch ~149–155), hub plikowy `DefaultModelBuilder`, wysoki % fix, unknowns model↔project / profile / CI-friendly.  
Z artifact-2: niski Ca, ale na każdej ścieżce runtime (core/cli/compat/embedder); SPI + qualified exports — wąskie gardło kontraktu.  
Kampanie (`mvnup`, ITS) i Dependabot/root POM świadomie odrzucone jako fałszywe centra.

**TOP 5 kandydatów (nie wybrani dalej):** `maven-impl` · `maven-core` · `maven-api-core` · `maven-support` · `compat/maven-model-builder`.

---

## 2. Koncentracja wiedzy

| Metryka | Wartość |
|---------|---------|
| Commity (human-led) | **155** · **28** autorów |
| TOP 1 | **Guillaume Nodet** — **82 (52.9%)** |
| TOP 3 | Nodet + Tamas Cservenak + Goutam Adwant — **104 (67.1%)** |
| TOP 5 | + Sylwester Lachiewicz + Martin Desruisseaux — **~76%** |
| AI co-authored (trailer) | **~32%** commitów ludzkich |
| Boty w module | **0** |

**Werdykt koncentracji: skupiona (norma ASF/Maven).**  
Przy małej liczbie aktywnych committerów >50% TOP1 nie jest automatycznym alarmem bus-factor — to typowy obraz. Volume jest wokół Nodeta; **tematycznie** jest druga linia (Resolver, hardening, parent-inference features, sources/JPMS), więc nie jest to „jedna głowa na wszystko”.

### Kogo zapytać (punkt wejścia, nie formalny ownership)

| Temat zmiany | Punkt wejścia | Alternatywa / lektura |
|--------------|---------------|------------------------|
| Profile external/consumer, parent, CI-friendly `${revision}`, model pipeline | **Guillaume Nodet** | PR `#13112`, trio `#12317`→revert→`#12323` |
| Trust model repo-resolved POM / walidacja coords | **Sylwester Lachiewicz** | `#12948`; napięcie: zamknięty unmerged `#13030` |
| Resolver pin / validation control / session↔repo | **Tamas Cservenak** | `#12520`, `#11238` |
| Iterative parent / inference / BOM UX / nowe SPI | **Goutam Adwant** | `#13104`, `#12703` |
| Source roots / `targetPath` / path matching / JPMS messaging | Martin Desruisseaux | `#11322`, `#11551` |

**Priorytet rozmowy przed dużą zmianą w modelu:** Nodet; przy hardeningu external POM — Lachiewicz (+ czytać `#13030`).

---

## 3. Ludzie × tematy supportu (skrót)

| Osoba | Powtarzające się tematy | Support przy |
|-------|-------------------------|--------------|
| Guillaume Nodet | `DefaultModelBuilder`, sandbox/external profiles, CI-friendly/parent, mirrors, toolchain, deadlock cache | Decyzje i edge case'y modelu M4 |
| Tamas Cservenak | Resolver 2.x bumps, `MavenValidator`, `RepositoryAwareRequest`, type derive | Integracja Resolver / walidacja artifactów |
| Goutam Adwant | Iterative parents, parent inference, BOM warnings, profile ranges, settings/session SPI | Feature’y M4 i kontrakty API wokół modelu |
| Sylwester Lachiewicz | Restrict repo-resolved model, coords/metadata validation, proxy decrypt | Hardening / trust external POM (odfiltrować port docs APT→MD) |
| Martin Desruisseaux | `DefaultSourceRoot`, `targetPath`↔`.mdo`, FileSelector, automodules | Semantyka sources/paths/JPMS |

Chore (Spotless, site rename) oddzielone od decyzji — nie budują rankingu ekspertyzy.

---

## 4. Lektury przed zmianą (skrót)

**Obowiązkowe:** `#12948` (trust repo-resolved) · `#13112`/`#13164` (sandbox profiles) · `#12274` (model SPI → api-spi) · `#13104` (iterative parents) · `#12520` (validation control).

**Edge / regresje:** `#12317`→revert→`#12323` (CI-friendly SOE × consumer) · `#12446` (deadlock parent cache) · `#12078` (false parent cycle / shade DRP) · `#12809` (Path normalize SOE).

**Opcjonalne / unmerged:** `#13030` (propozycja revert restrict interpolation — napięcie kontraktu) · `#13095` (wczesna ścieżka do `#13084`) · `#13254` (activeByDefault / `-P`, dyskusja).

---

## 5. Powiązanie z terytorium i strukturą

- **Territory:** wysoka aktywność + fix pressure w impl = dużo edge case’ów M4; bez mapy ludzi łatwo powtórzyć revertowany fix (`#12317`) albo złamać hardening (`#12948`).
- **Structure:** zmiana w impl ma blast na core/cli/compat/embedder i SPI; kontekst ludzki + PR-y decyzyjne uzupełniają to, czego jdeps nie widać (profile sandbox, trust model, concurrent cache).

---

## 6. Ograniczenia

- Historia commitów ≠ ownership ani dostępność osoby.
- Osoba mogła ograniczyć aktywność / odejść — ranking 12 mies. nie gwarantuje kontaktu.
- Decyzje z PR mogą być nieaktualne lub w napięciu (np. `#12948` vs `#13030`).
- Brak adresów e-mail w tym artefakcie; brak wniosków o zatrudnieniu / formalnych rolach ASF.
- AI co-author (~32%) zawyża „produktywność” narzędzi, nie zastępuje eksperta domenowego.
- Konsumentów zewnętrznych (pluginy) i pełnej deliberacji w `maven-resolver` nie mapowano.

---

## 7. Unknowns → Deep Focus

1. Czy napięcie `#12948` / `#13030` jest domknięte produktowo (pełna vs ograniczona interpolacja external POM)?
2. Granica **model builder (impl) vs project builder (core)** przy parent/BOM — kto jeszcze trzyma drugą stronę.
3. Binary/behavioral break `maven-api-spi` / api-core dla ekosystemu (japicmp) — poza samym rankingiem commitów.
4. Concurrent lifecycle w **maven-core** — osobny Deep Focus (nie ten artefakt).
5. Obowiązkowość `maven-compat` / embedder po M4 względem zmian w impl.

---

## 8. Werdykt Kroku 3

Dla `impl/maven-impl` wiedza commitowa jest **skupiona** (Nodet ~53%, TOP3 ~67%), co przy zespole Maven/ASF jest **oczekiwane**, nie czerwony alert. Support tematyczny ma drugą linię (Cservenak, Adwant, Lachiewicz, Desruisseaux). Przed większą zmianą: przeczytać lektury z §4 i wejść przez Nodeta (model/profile/parent) lub Lachiewicza (trust external POM).

Następny krok mapowania: Deep Focus na unknowns (§7) / inne centra (`maven-core`, `api-core`) — **nie** pełny `repo-map.md` w tym kroku.
