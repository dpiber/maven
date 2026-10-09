# M4L2 / Krok 3 — bullet 4: koncentracja wiedzy + zapis artefaktu

Repo: https://github.com/apache/maven  
Cel: ocenić, czy wiedza o wybranym obszarze jest rozproszona czy skupiona; zapisać syntezę Kroku 3.  
Uruchamiaj jako ostatni prompt kroku 3 (po `m4l2-01`…`m4l2-03` contributors).

## Wymagania konieczne

- **Nie używaj skilli 10x** (np. `/10x-repo-map`, `/10x-init`, `/10x-research` ani żadnego innego skillu z paczki 10xDevs). Wykonaj zadanie ręcznie: komendy CLI + interpretacja w tej sesji. Katalog `../../../../context/map` utwórz zwykłym `mkdir` / zapisem pliku — bez `/10x-init`.

```text
Wymagania konieczne: nie używaj skilli 10x (w tym /10x-repo-map, /10x-init i pozostałych). Pracuj ad hoc na CLI i historii gita; katalog context/map/ utwórz bez /10x-init.

Pracujemy nadal na apache/maven. Zamykamy mapę kontrybutorów dla wybranego obszaru.

Pytanie przewodnie: czy wiedza o tym obszarze wygląda na rozproszoną, czy skupioną wokół kilku osób?

Na podstawie rankingu, tematów i lektur z poprzednich promptów:

1. Policz udział TOP 1 / TOP 3 osób w commitach obszaru (okno 12 miesięcy, po filtrze botów; AI osobno).
2. Oceń koncentrację: skupiona / umiarkowana / rozproszona — z uwzględnieniem wielkości zespołu Maven/ASF (przy 2–3 aktywnych osobach skupienie to norma, nie alarm).
3. Wskaż 1–2 kandydatów „kogo zapytać” dopasowanych tematycznie (nie formalnych ownerów — punkt wejścia do rozmowy / lektury PR).
4. Zapisz ograniczenia: historia ≠ właścicielstwo; osoba mogła odejść; decyzje z PR mogą być nieaktualne; brak e-maili i brak wniosków o zatrudnieniu.
5. Krótko powiąż z artifact-1 (aktywność/fixy) i artifact-2 (blast radius / kontrakt) — dlaczego właśnie ten obszar wymaga kontekstu ludzkiego.

Następnie zaktualizuj krótkie podsumowanie Kroku 3 (bez powtarzania pełnych tabel):
- wybrany obszar i dlaczego,
- kluczowi ludzie × tematy supportu,
- lektury PR / edge case'y,
- koncentracja wiedzy + kogo zapytać,
- unknowns do Deep Focus.

Na koniec zapisz wynik całej sesji Kroku 3 jako `context/map/artifact-3-contributors.md`.
Jeśli katalog `context/map/` nie istnieje — utwórz go. Artefakt ma być zwięzły, z dowodami z `git`/`gh`, z jawnym oknem 12 miesięcy i filtrami (boty, AI). Nie twórz jeszcze `repo-map.md`.
```
