# M4L2 / Krok 3 — bullet 3: PR-y, decyzje i edge case'y do przeczytania

Repo: https://github.com/apache/maven  
Cel: wskazać konkretne PR-y, decyzje lub edge case'y warte przeczytania przed zmianą w wybranym obszarze.  
Uruchamiaj po `m4l2-01` i `m4l2-02` contributors.

## Wymagania konieczne

- **Nie używaj skilli 10x** (np. `/10x-repo-map`, `/10x-init`, `/10x-research` ani żadnego innego skillu z paczki 10xDevs). Wykonaj zadanie ręcznie: komendy CLI + interpretacja w tej sesji.

```text
Wymagania konieczne: nie używaj skilli 10x (w tym /10x-repo-map, /10x-init i pozostałych). Pracuj ad hoc na CLI (`git log`, opcjonalnie `gh`); tylko odczyt — niczego nie zmieniaj na GitHubie.

Pracujemy nadal na apache/maven. Masz wybrany obszar, ranking ludzi i mapę tematów z poprzednich promptów oraz artifact-1 / artifact-2.

Pytanie przewodnie: które PR-y, decyzje albo edge case'y warto przeczytać przed zmianą?

Dla wybranego obszaru zbierz krótką listę lektur (cel: 5–10 pozycji, nie dump historii):

1. PR-y / commity z wyraźną decyzją projektową (API break/compat, nowy kontrakt, zmiana lifecycle, migracja Maven 3→4, resolver, model v4).
2. Edge case'y i trudne poprawki (revert, hotfix, „fixes … after …”, ITS flaky związane z obszarem, regression).
3. Dyskusje / PR-y zamknięte bez merge'a albo długo dyskutowane — jeśli `gh` jest dostępne przeciwko apache/maven; jeśli nie — zaznacz jako unknown i oprzyj się na treściach commitów.
4. Powiąż każdą pozycję z osobą/tematem z poprzedniego prompta („dlaczego to czytać przed X”).

Filtry: pomiń czyste chore wersji, Spotless-only, masowe license headers. Preferuj rzeczy, które tłumaczą *dlaczego* kod wygląda jak wygląda.

To nie jest git blame linii — mapa kontrybutorów ma wskazać kontekst i typ problemów, nie „ostatniego edytora pliku”.

Format odpowiedzi:
- Markdown
- sekcje: `Lektury obowiązkowe`, `Edge case'y / regresje`, `Opcjonalne / unknowns`
- w tabelach: Tytuł/ID | Link lub hash | Obszar ścieżek | Dlaczego przed zmianą | Kogo to dotyczy (osoba/temat)
Nie zapisuj jeszcze artifact-3-contributors.md.
```
