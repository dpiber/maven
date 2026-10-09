# M4L2 / Krok 1 — bullet 2: współzmiany (co-change)

Repo: https://github.com/apache/maven  
Cel: znaleźć katalogi/pliki, które często zmieniają się razem w tych samych commitach.  
Uruchamiaj po `m4l2-01-territory.md` (masz już ranking aktywności).

## Wymagania konieczne

- **Nie używaj skilli 10x** (np. `/10x-repo-map`, `/10x-init`, `/10x-research` ani żadnego innego skillu z paczki 10xDevs). Wykonaj zadanie ręcznie: komendy CLI + interpretacja w tej sesji.

```text
Wymagania konieczne: nie używaj skilli 10x (w tym /10x-repo-map, /10x-init i pozostałych). Pracuj ad hoc na CLI i historii gita.

Pracujemy nadal na apache/maven. Na podstawie tego samego okna ostatnich 12 miesięcy (i rankingu z poprzedniego prompta):

Jakie pary lub trójki katalogów/modułów najczęściej pojawiają się w tych samych commitach? Wyszukaj sprzężenia (co-change) i krótko podsumuj wnioski dla top 3 obszarów z rankingu aktywności. Pomiń commity, które dotykają naraz dużej części repo (np. release chore, masowy reformat, bump wersji we wszystkich pom.xml) — one wiążą wszystko ze wszystkim. Przy każdej parze podaj: w ilu commitach wystąpiła razem oraz jaki to procent commitów mniej aktywnego z dwóch obszarów.

Szczególnie interesują mnie typowe dla Mavena granice, jeśli widać je w historii, np.:
- maven-model / maven-model-builder ↔ maven-core
- maven-plugin-api ↔ maven-core / maven-compat
- maven-settings / maven-settings-builder ↔ maven-embedder / maven-core
- resolver / artifact ↔ core
- its/ ↔ konkretny moduł produkcyjny

Jeszcze dwie rzeczy przy okazji tych współzmian:

1. Czy jest jakiś pojedynczy plik, który zmienia się razem z wieloma różnymi obszarami naraz? Myślę o wspólnym mianowniku całego repo — np. root `pom.xml`, `.mvn/`, parent BOM, plik generowany, wspólna konfiguracja. Jeśli tak, sprawdź, czy ktoś go edytuje ręcznie, czy zmienia się automatycznie (generator, release plugin, build) — to dwa różne koszty zmiany.
2. Sprawdź, czy pliki/moduły, które wyszły jako mocno sprzężone, na pewno nadal są w repo na HEAD. To historia: coś mogło dużo się zmieniać, a potem zostać usunięte, przeniesione albo wydzielone do osobnego projektu — nie chcę później opierać analizy na ścieżce, której już nie ma. Jeśli coś zniknęło, oznacz jako historyczne sprzężenie.

Format: Markdown — tabela par/trójek + 3–5 wniosków decyzyjnych (co przy zmianie w A zwykle rusza B). Nie zapisuj jeszcze artifact-1-territory.md.
```
