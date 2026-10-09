# M4L2 / Krok 1 — bullet 4: filtr szumu + zapis artefaktu

Repo: https://github.com/apache/maven  
Cel: jawnie doprecyzować, co było/jest szumem w analizie terytorium, i zapisać syntezę sesji.  
Uruchamiaj jako ostatni prompt kroku 1 (po `m4l2-01`…`m4l2-03`).

## Wymagania konieczne

- **Nie używaj skilli 10x** (np. `/10x-repo-map`, `/10x-init`, `/10x-research` ani żadnego innego skillu z paczki 10xDevs). Wykonaj zadanie ręcznie: komendy CLI + interpretacja w tej sesji. Katalog `../../../../context/map` utwórz zwykłym `mkdir` / zapisem pliku — bez `/10x-init`.

```text
Wymagania konieczne: nie używaj skilli 10x (w tym /10x-repo-map, /10x-init i pozostałych). Pracuj ad hoc na CLI i historii gita; katalog context/map/ utwórz bez /10x-init.

Pracujemy nadal na apache/maven. Zanim zamkniemy mapę terytorium, zrób jawny audyt szumu z dotychczasowej analizy (ostatnie 12 miesięcy) i popraw rankingi/wnioski, jeśli coś wcześniej wpadło do TOP przez pomyłkę.

Wypisz i sklasyfikuj, co trzeba odfiltrować / osobno oznaczyć jako szum:

- lockfile'y i ekwivalent (jeśli występują),
- snapshoty testowe, golden files, duże fixtury w `its/` zmieniane hurtowo,
- pliki generowane / `target/` / generated-sources (jeśli kiedykolwiek commitowane),
- masowe formatowanie (Spotless, import order, Checkstyle-only, license header bumps),
- lokalizacje / tłumaczenia (jeśli nie niosą logiki produktu),
- commity botów i automatyzacji (Dependabot, Renovate, ASF bots, release automation),
- masowe przenosiny / rename bez zmiany zachowania,
- czyste chore wersji we wszystkich `pom.xml` naraz.

Dla każdej kategorii szumu podaj: przykładowe ścieżki lub wzorce commitów, szacunek wpływu na ranking (o ile zawyżały TOP), oraz czy po odfiltrowaniu zmienia się wniosek o „stałe centra” projektu.

Następnie zaktualizuj krótkie podsumowanie terytorium (bez powtarzania pełnych tabel z poprzednich odpowiedzi):
- stałe centra vs kampanie vs obszary wygasające,
- top sprzężenia co-change warte uwagi,
- przecięcia z runtime / API / settings / resolver / build,
- lista `unknowns` do Deep Focus,
- miejsca „wygląda groźnie, ale nie jest”.

Na koniec zapisz wynik całej sesji Kroku 1 jako `context/map/artifact-1-territory.md`.
Jeśli katalog `context/map/` nie istnieje — utwórz go. Artefakt ma być zwięzły, z dowodami z komend gita/`gh`, z jawnym oknem czasowym i listą odfiltrowanego szumu. Nie twórz jeszcze `repo-map.md` ani pozostałych artifact-*.md.
```
