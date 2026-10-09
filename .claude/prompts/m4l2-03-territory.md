# M4L2 / Krok 1 — bullet 3: aktywność × wrażliwe warstwy

Repo: https://github.com/apache/maven  
Cel: wskazać, gdzie aktywność z historii przecina się z runtime, danymi, publicznym API, integracjami albo buildem.  
Uruchamiaj po `m4l2-01` i `m4l2-02`.

## Wymagania konieczne

- **Nie używaj skilli 10x** (np. `/10x-repo-map`, `/10x-init`, `/10x-research` ani żadnego innego skillu z paczki 10xDevs). Wykonaj zadanie ręcznie: komendy CLI + interpretacja w tej sesji.

```text
Wymagania konieczne: nie używaj skilli 10x (w tym /10x-repo-map, /10x-init i pozostałych). Pracuj ad hoc na CLI i historii gita.

Pracujemy nadal na apache/maven. Masz już ranking aktywności i sygnały współzmian z poprzednich promptów.

Przeciąć te wyniki z wrażliwymi warstwami typowymi dla Mavena i odpowiedzieć: gdzie ostatnia aktywność realnie przecina się z runtime, danymi/konfiguracją, publicznym API, integracjami albo buildem — a gdzie „gorąco” to tylko testy/docs bez blast radius na produkt.

Mapuj obszary aktywności na kategorie (obszar może trafić do kilku):

1. Runtime / lifecycle — wykonywanie builda, lifecycle, mojo execution, classrealm/plugin realm, session/project state w maven-core / maven-embedder.
2. Dane / konfiguracja użytkownika — settings, toolchains, profiles, credentials/mirror/proxy (maven-settings*, user/global settings); nie szukaj sekretów w treści plików, tylko ścieżek i tematyki commitów.
3. Publiczne API / kontrakty — maven-plugin-api, publiczne modele POM (maven-model*), API używane przez pluginy i embedder; zmiany, które łamią kompatybilność pluginów.
4. Integracje — Maven Resolver / artifact resolution, wagon/transport (jeśli w historii), CI, ITS (integration test suite) jako sygnał integracji zewnętrznych, kompatybilność z Maven 3 (maven-compat).
5. Build samego projektu — root/parent POM, `.mvn/`, pluginManagement, release, enforcer, generowanie distribution (apache-maven), wrapper.

Dla każdej kategorii wypisz:
- które aktywne ścieżki z rankingu/co-change w nią wpadają,
- dowód z gita (liczniki / przykładowe tematy commitów lub PR),
- czy to wygląda na stałe centrum, kampanię, czy szum testowy,
- ostrzeżenie przed zmianą (jedno zdanie) albo `n/a`,
- `unknown`, jeśli z samej historii nie da się rozstrzygnąć (np. czy zmiana API jest breaking).

Zaznacz wprost miejsca, które wyglądają groźnie, ale nie są (np. gęste poprawki w `its/` przy stabilnym core, albo chore w parent POM).

Format: Markdown — sekcja per kategoria + krótka lista „najwyższa wrażliwość przy zmianie”. Nie zapisuj jeszcze artifact-1-territory.md.
```
