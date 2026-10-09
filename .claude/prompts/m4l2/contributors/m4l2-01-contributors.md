# M4L2 / Krok 3 — bullet 1: kto pracował przy wybranym obszarze

Repo: https://github.com/apache/maven  
Cel: na podstawie artifact-1-territory i artifact-2-structure wybrać obszar centralny/podejrzany i ustalić, kto najczęściej przy nim pracował w ostatnich 12 miesiącach.  
Uruchamiaj po `../../../../context/map/artifact-1-territory.md` oraz `../../../../context/map/artifact-2-structure.md`.

## Wymagania konieczne

- **Nie używaj skilli 10x** (np. `/10x-repo-map`, `/10x-init`, `/10x-research` ani żadnego innego skillu z paczki 10xDevs). Wykonaj zadanie ręcznie: komendy CLI + interpretacja w tej sesji.

```text
Wymagania konieczne: nie używaj skilli 10x (w tym /10x-repo-map, /10x-init i pozostałych). Pracuj ad hoc na CLI i historii gita (`git log` / opcjonalnie `gh`).

Pracujemy na sklonowanym repozytorium apache/maven (https://github.com/apache/maven). Masz:
- `context/map/artifact-1-territory.md`
- `context/map/artifact-2-structure.md`

Zapoznaj się z obu artefaktami, a następnie zidentyfikuj top 5 obszarów (modułów/pakietów), które mogą wymagać kontaktu z kontrybutorami przed większą zmianą — łącz sygnał z terytorium (aktywność, fixy, co-change) i ze struktury (load-bearing, kontrakty, blast radius, cykle). Kandydaci typowi dla Mavena: impl/maven-impl (model), impl/maven-core (lifecycle/plugin realm), impl/maven-cli (mvnup/CLI), api/maven-api-*, wybrany compat/*.

Potem wybierz JEDEN główny obszar do dalszej analizy w tym kroku (możesz krótko uzasadnić wybór względem pozostałej czwórki).

Pytanie przewodnie: kto najczęściej pracował przy wybranym obszarze w ostatnich 12 miesiącach?

Dla wybranego obszaru (ścieżki źródłowe, bez `target/` i generated-sources) pokaż ranking kontrybutorów z ostatnich 12 miesięcy (liczba commitów / linii lub plików — podaj, którą miarę używasz). Odfiltruj boty i automatyzacje (Dependabot, Renovate, ASF bots, release automation). Commity, których autorem jest agent AI (Claude, Codex, Copilot, Cursor Agent itd.) odfiltruj z rankingu ludzi — ale podaj ich udział osobno jako aktywność AI. Commit z człowiekiem jako autorem i agentem jako współautorem zostaw przy człowieku — to człowiek prowadził zmianę.

Podawaj same nazwy autorów, bez adresów e-mail. Jeśli ta sama osoba występuje pod kilkoma aliasami/adresami, połącz ją w jedną pozycję.

Weź pod uwagę wielkość zespołu ASF/Maven: przy małej liczbie aktywnych committerów skupienie wiedzy w jednej–dwóch głowach to norma, a nie automatyczne „ryzyko bus-factor” — na razie tylko ranking i kontekst, bez diagnozy koncentracji (to kolejny prompt).

Format: Markdown — krótki wybór obszaru + tabela TOP kontrybutorów + udział botów/AI. Nie zapisuj jeszcze artifact-3-contributors.md.
```
