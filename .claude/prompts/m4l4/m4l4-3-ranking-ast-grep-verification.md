# M4L4 / m4l4-3 — weryfikacja rankingu (`ast-grep`)

Alias kursowy dla weryfikacji z kroku 2 — pełna wersja ze checklistą lektury: `m4l4-02.md`.

Repo: https://github.com/apache/maven (Java)  
Raport: `context/changes/refactor-opportunities/research.md`

```text
Zweryfikuj raport context/changes/refactor-opportunities/research.md
(apache/maven, Java — nie Mattermost/Go).

Wypisz z niego twierdzenia STRUKTURALNE, na których stoi ranking (liczby metod / LOC / @Test,
"nadpisuje / implementuje X, ale nie Y", liczność call-site'ów, pary lustrzanych typów
impl↔compat DefaultModelBuilder / dwa DefaultProjectBuilder).

Dla każdego zbuduj wzorzec ast-grep (-l java), wywołaj go i podaj wynik jako:
twierdzenie -> potwierdzone / doprecyzowane / obalone, z plikami i liniami.
Każde zero z ast-grep potwierdź klasycznym grepem / rg.

Po weryfikacji zaktualizuj analizowany raport:
- błędne liczby i numery linii popraw w miejscu, w formacie "150 (raport: 145)" —
  tak, żeby ślad korekty został w tekście;
- dodaj sekcję "## Weryfikacja twierdzeń (ast-grep)" z tabelą:
  twierdzenie → werdykt → dowód (plik:linia) → metoda (wzorzec/reguła);
- zaktualizuj frontmatter: last_updated, dopisz tag "verified" i commit weryfikacji;
- sekcji "Refactor opportunities (ranked)" oraz werdyktów intencjonalności NIE zmieniaj.

Jeśli wynik podważa pozycję kandydata, opisz to wyłącznie w sekcji weryfikacji,
z adnotacją "do decyzji na etapie planowania".

Nie zmieniaj kodu źródłowego Mavena.
```
