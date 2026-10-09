# M4L2 / Krok 2 — bullet 4: blast radius + zapis artefaktu

Repo: https://github.com/apache/maven  
Cel: dla kluczowych modułów ustalić, kto od nich zależy i co może pęknąć przy zmianie; zapisać syntezę Kroku 2.  
Uruchamiaj jako ostatni prompt kroku 2 (po `m4l2-01`…`m4l2-03` structure).

## Wymagania konieczne

- **Nie używaj skilli 10x** (np. `/10x-repo-map`, `/10x-init`, `/10x-research` ani żadnego innego skillu z paczki 10xDevs). Wykonaj zadanie ręcznie: komendy CLI + interpretacja w tej sesji. Katalog `../../../../context/map` utwórz zwykłym `mkdir` / zapisem pliku — bez `/10x-init`.

```text
Wymagania konieczne: nie używaj skilli 10x (w tym /10x-repo-map, /10x-init i pozostałych). Pracuj ad hoc na CLI i grafie zależności; katalog context/map/ utwórz bez /10x-init.

Pracujemy nadal na apache/maven. Zamykamy mapę struktury.

Pytanie przewodnie: kto zależy od danego modułu i co może pęknąć przy zmianie?

Weź 4–6 load-bearing / kontraktowych celów z poprzednich promptów i z artifact-1-territory (kandydaci: api/maven-api-core, impl/maven-impl, impl/maven-core, ewent. maven-cli, wybrany compat). Dla każdego:

1. Incoming — które inne moduły/pakiety Mavena zależą od niego (jdeps + dependency:tree reactora; odpowiednik „--reaches”).
2. Outgoing — od czego sam zależy w obrębie reactora (odpowiednik „--focus”).
3. Blast radius przy zmianie publicznego kontraktu vs przy zmianie wewnętrznej implementacji (-apionly pomaga rozdzielić).
4. Ryzyko testów wynikające z grafu: gdzie zmiana naturalnie wymaga unit z mocnym mockowaniem, gdzie IT (its/core-it-suite), a gdzie regresja pluginów zewnętrznych jest `unknown` (jdeps tego nie zobaczy).
5. Jedno zdanie caution + lista unknowns (ClassRealm, SPI, japicmp/breaking API, runtime DI).

Opcjonalnie: jeśli jedno sprzężenie jest decyzyjnie kluczowe, możesz wyrenderować TYLKO ten podgraf do SVG (np. `jdeps --dot-output <dir>` + `dot -Tsvg`, albo ręczny podgraf z macierzy krawędzi → Graphviz → `.svg`; fragment `dependency:tree` też OK jako źródło) — nie renderuj całego repo. Format oczekiwany to SVG; DOT jest tylko pośrednim wejściem do renderu. Markdown pozostaje źródłem prawdy.

Następnie zaktualizuj krótkie podsumowanie struktury (bez powtarzania pełnych tabel z poprzednich odpowiedzi):
- kontrakty i granice warstw (co trzyma, co przecieka),
- cienkie wejścia vs głębsze centra,
- cykle / podejrzane zależności (albo ich brak),
- blast radius dla TOP celów,
- miejsca „wygląda groźnie, ale nie jest” (np. gęste ITS, Dependabot POM, mvnup ≠ core),
- lista unknowns do Deep Focus.

Na koniec zapisz wynik całej sesji Kroku 2 jako `context/map/artifact-2-structure.md`.
Jeśli katalog `context/map/` nie istnieje — utwórz go. Artefakt ma być zwięzły, z dowodami z `jdeps` / `mvn dependency:*`, z jawnym zakresem modułów i ograniczeniami analizy statycznej. Nie twórz jeszcze `repo-map.md` ani `artifact-3-contributors.md`.
```
