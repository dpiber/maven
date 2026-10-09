# M4L2 / Krok 2 — bullet 1: kontrakty między warstwami

Repo: https://github.com/apache/maven  
Cel: znaleźć kontrakty używane między warstwami (API ↔ impl ↔ compat ↔ CLI) na podstawie grafu zależności.  
Uruchamiaj po `context/map/artifact-1-territory.md` (masz już ranking aktywności).  
Stack: Java / Maven multi-module — preferuj `jdeps` (JDK) i `mvn dependency:*`; nie zakładaj dependency-cruiser.

## Wymagania konieczne

- **Nie używaj skilli 10x** (np. `/10x-repo-map`, `/10x-init`, `/10x-research` ani żadnego innego skillu z paczki 10xDevs). Wykonaj zadanie ręcznie: komendy CLI + interpretacja w tej sesji.

```text
Wymagania konieczne: nie używaj skilli 10x (w tym /10x-repo-map, /10x-init i pozostałych). Pracuj ad hoc na CLI i grafie zależności.

Pracujemy na sklonowanym repozytorium apache/maven (https://github.com/apache/maven). Masz `context/map/artifact-1-territory.md`.

Zanim cokolwiek instalujesz: sprawdź toolchain (java, jdeps, mvn), layout modułów (api/, impl/, compat/, apache-maven/, its/), istniejące skrypty/CI oraz czy w `*/target/*.jar` są już zbudowane artefakty. Potwierdź, że `jdeps` jest w PATH (powinien pochodzić z JDK). Jeśli brak JAR-ów potrzebnych do analizy aktywnych modułów, zbuduj tylko to, co konieczne (np. `mvn -DskipTests package` na wskazanych modułach) — nie instaluj dependency-cruiser ani narzędzi Node tylko dlatego, że lekcja je wspomina. Uzupełniająco możesz użyć `mvn dependency:tree` / `dependency:analyze`, ale główny sygnał struktury ma pochodzić z `jdeps` na artefaktach produkcyjnych (nie na test-fixtures w target/test-classes).

Zrób top 3 pomysłów, co `jdeps` (i ewentualnie Maven Dependency Plugin) może dać do mapy struktury tego repo, potem przejdź do analizy kontraktów.

Pytanie przewodnie: gdzie widać kontrakty używane między warstwami?

Skup się na aktywnych obszarach z artifact-1-territory (m.in. impl/maven-impl, impl/maven-core, impl/maven-cli, api/maven-api-*, compat/*). Sprawdź, czy układ warstw jest przewidywalny, np.:
- api/maven-api-* jako fundament kontraktów (publiczne API Maven 4),
- impl/* zależy od api/*, a nie odwrotnie,
- compat/* jako most do Maven 3 / legacy (maven-compat, maven-model*, embedder),
- CLI / embedder jako cienka warstwa wejścia nad core/impl,
- brak niedozwolonych importów „w górę” (impl → api OK; api → impl = alarm).

Użyj m.in.:
- `jdeps -summary` / `jdeps -verbose:package` na JAR-ach aktywnych modułów,
- `jdeps -apionly` tam, gdzie chcesz oddzielić zależność z publicznego API od zależności implementacyjnych,
- filtruj JDK i zewnętrzne biblioteki (interesuje nas sprzężenie między modułami Mavena, nie graf Guice/SLF4J).

Importy typów / generowany kod / zależności tylko testowe wypisz osobno — nie traktuj ich jak złamania granic warstw. Jawnie zapisz ograniczenia `jdeps`: nie widzi refleksji, `Class.forName`, DI/Plexus/Guice wiring, SPI ładowanego w runtime ani ClassRealm pluginów — to `unknown`, nie „brak zależności”.

Zinterpretuj wyniki w kontekście aktywności z artifact-1-territory (szczególnie model builder, core/lifecycle, api-core, compat↔impl).

Format odpowiedzi:
- Nie generuj grafu SVG (ani pośredniego DOT) na tym etapie — to dopiero opcjonalnie w `m4l2-04` (format końcowy: SVG).
- Markdown: najpierw 3–5 obserwacji, potem tabela z kolumnami:
  - Sprawdzana granica / kontrakt
  - Wynik
  - Dowód (jdeps / dependency:tree)
  - Dlaczego to ważne przy zmianie
  - Związek z artifact-1-territory.md
  - Co sprawdzić dalej
Nie zapisuj jeszcze artifact-2-structure.md.
```
