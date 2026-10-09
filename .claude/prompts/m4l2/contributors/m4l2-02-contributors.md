# M4L2 / Krok 3 — bullet 2: powtarzające się tematy u kontrybutorów

Repo: https://github.com/apache/maven  
Cel: dla kluczowych osób z wybranego obszaru sklasyfikować powtarzające się tematy pracy — kto oferuje support przy jakim typie problemu.  
Uruchamiaj po `m4l2-01-contributors.md`.

## Wymagania konieczne

- **Nie używaj skilli 10x** (np. `/10x-repo-map`, `/10x-init`, `/10x-research` ani żadnego innego skillu z paczki 10xDevs). Wykonaj zadanie ręcznie: komendy CLI + interpretacja w tej sesji.

```text
Wymagania konieczne: nie używaj skilli 10x (w tym /10x-repo-map, /10x-init i pozostałych). Pracuj ad hoc na CLI i historii gita (tematy commitów / PR-ów).

Pracujemy nadal na apache/maven. Masz wybrany obszar i ranking kontrybutorów z poprzedniego prompta oraz artifact-1-territory / artifact-2-structure.

Pytanie przewodnie: jakie tematy powtarzają się u konkretnych kontrybutorów?

Weź top 3–5 osób z rankingu (ludzie, nie boty/AI) dla wybranego obszaru z ostatnich 12 miesięcy. Dla każdej osoby:

1. Przejrzyj tematy commitów (i jeśli masz `gh` — tytuły/opisy zmerge'owanych PR-ów) w tym obszarze.
2. Sklasyfikuj aktywności pogrupowane tematycznie — np. model builder / POM consumer, lifecycle/plugin realm, resolver/repo, CLI/mvnup, API kontrakty (maven-api-*), compat/legacy bridge, testy ITS, JPMS/DI, poprawki regresji.
3. Wskaż, przy jakim typie problemu ta osoba najpewniej zaoferuje support (jedno–dwa zdania inference na osobę).
4. Oddziel „dużo commitów chore/format” od „decyzje i edge case'y” — interesuje nas kontekst, nie sama liczba.

Nadal: same nazwy autorów, bez e-maili; scal aliasy jednej osoby. Nie traktuj samej liczby commitów jako dowodu eksperckiej wiedzy — tematy i powtarzalność > volume.

Format odpowiedzi:
- Markdown
- najpierw 3–5 obserwacji (kto-za-czym)
- potem tabela: Osoba | Tematy powtarzające się | Dowód (przykładowe commity/PR) | Do czego pytać przed zmianą | Uwagi / unknowns
Nie zapisuj jeszcze artifact-3-contributors.md.
```
