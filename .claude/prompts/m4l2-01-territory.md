# M4L2 / Krok 1 — bullet 1: stale aktywne obszary

Repo: https://github.com/apache/maven  
Cel: ustalić, które katalogi/moduły i pliki były stale aktywne w ostatnich miesiącach (Wide Scan / terytorium).  
Nie czytaj jeszcze kodu źródłowego — pracuj na historii gita i wynikach CLI.

## Wymagania konieczne

- **Nie używaj skilli 10x** (np. `/10x-repo-map`, `/10x-init`, `/10x-research` ani żadnego innego skillu z paczki 10xDevs). Wykonaj zadanie ręcznie: komendy CLI + interpretacja w tej sesji.

```text
Wymagania konieczne: nie używaj skilli 10x (w tym /10x-repo-map, /10x-init i pozostałych). Pracuj ad hoc na CLI i historii gita.

Pracujemy na sklonowanym repozytorium apache/maven (https://github.com/apache/maven).

Korzystając z historii gita, w zakresie ostatnich 12 miesięcy, pokaż TOP 10 najczęściej modyfikowanych:

a) folderów lub modułów Maven (np. maven-core, maven-model, maven-compat, maven-embedder, maven-plugin-api, apache-maven, its/, …)
b) plików

Odfiltruj szum: lockfile'y, snapshoty, generowane pliki (target/, generated-sources), LICENSE/NOTICE w masowych update'ach, grafiki, raporty RAT, pliki lokalizacyjne jeśli nie niosą logiki. Pomiń commity botów (Dependabot, Renovate, ASF automation, automatyczne podbicia wersji) i masowe zmiany typu formatowanie (Spotless/Checkstyle-only) czy czyste przenosiny plików — ale nie odrzucaj commita tylko dlatego, że jest duży: duży feature zostaje. Commity, których autorem jest agent AI, licz jako aktywność i podaj ich udział osobno. Jeśli jakiś obszar zmienił w tym czasie nazwę albo lokalizację, połącz starą ścieżkę z nową.

Jeśli historia w oknie 12 miesięcy jest niepełna (shallow clone), powiedz to i zaproponuj `git fetch --unshallow` albo weź dostępną historię.

Możesz zejść poziom niżej, jeśli pierwsza seria wyników da zbyt ogólne rezultaty jak samo `maven-core` albo `src/` — chcemy realne obszary aktywności hands-on (np. lifecycle, plugin realm, model building, settings). Przy każdym obszarze dopisz jednym zdaniem, za jaką funkcję produktu Maven odpowiada (np. budowanie POM, rozwiązywanie zależności, API pluginów, CLI/embedder, kompatybilność wsteczna).

Potem podziel te same dane na kwartały (albo na miesiące, jeśli aktywność jest nierównomierna) — chcę zobaczyć, jak zmieniał się nacisk pracy. Oznacz każdy obszar z TOP jako: stały, rosnący, wygasający albo sezonowy; przy wyraźnym skoku nazwij kampanię z opisów commitów (np. Maven 4, model v4, resolver).

Dla 5 najaktywniejszych obszarów policz, jaka część commitów to poprawki (fix, revert, hotfix), a reverty podaj osobno. Zanim uznasz obszar za „psujący się”, przeczytaj tematy poprawek: w CI, ITS (integration tests) i drobnych poprawkach dokumentacji naprawianie metodą prób i błędów to zwykły tryb pracy. Jeden duży revert przypisz do obszaru, którego dotyczył.

Jeśli masz skonfigurowane GitHub CLI (`gh`) przeciwko apache/maven, dołóż PR-y zamknięte bez merge'a i otwarte zgłoszenia błędów w tych obszarach. Tylko odczyt: niczego nie zmieniaj na GitHubie.

Format: Markdown, rankingi + krótkie wnioski (co jest stałym centrum vs kampanią). Nie generuj jeszcze artifact-1-territory.md — to na końcu serii.
```
