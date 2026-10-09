# M4L2 / Krok 2 — bullet 3: cykle i podejrzane zależności

Repo: https://github.com/apache/maven  
Cel: wykryć cykle oraz podejrzane zależności w grafie modułów/pakietów aktywnych obszarów.  
Uruchamiaj po `m4l2-01` i `m4l2-02` structure.

## Wymagania konieczne

- **Nie używaj skilli 10x** (np. `/10x-repo-map`, `/10x-init`, `/10x-research` ani żadnego innego skillu z paczki 10xDevs). Wykonaj zadanie ręcznie: komendy CLI + interpretacja w tej sesji.

```text
Wymagania konieczne: nie używaj skilli 10x (w tym /10x-repo-map, /10x-init i pozostałych). Pracuj ad hoc na CLI i grafie zależności (jdeps / Maven Dependency Plugin).

Pracujemy nadal na apache/maven. Masz artifact-1-territory oraz dotychczasowe wnioski o kontraktach i centrach.

Pytanie przewodnie: czy graf pokazuje cykle albo podejrzane zależności?

Zakres: najaktywniejsze obszary z artifact-1-territory — m.in. impl/maven-impl, impl/maven-core, impl/maven-cli, api/maven-api-*, oraz powiązane compat/* jeśli wchodzą w graf z impl/core. Nie interesuje mnie pełna lista wszystkiego w repo ani ITS jako „cykl produktowy”.

Zrób:
1. Cykle między modułami reactora (Maven zwykle ich zabrania — jeśli build przechodzi, potwierdź brak cykli modułowych; jeśli coś wygląda na cykl pakietowy wewnątrz/między JAR-ami, wyłap to jdepsem).
2. Cykle / wzajemne zależności na poziomie pakietów (jdeps -verbose:package między parami aktywnych JAR-ów; szukaj A→B i B→A).
3. Podejrzane zależności: api → impl, warstwa wyższa ciągnąca compat legacy bez potrzeby, zależności test-only wplecione w main, niespodziewane mosty CLI↔model, „wszystko zależy od X”.
4. Oddziel sygnał architektoniczny od szumu: zależności tylko na typy z -apionly vs pełne; JDK; zewnętrzne liby; generowane klasy — drugie nie są problemem granic.

Dla każdego cyklu lub podejrzenia podaj najmniejszą realną pętlę (nie rozdmuchuj przez łańcuch typów) i prostym językiem, dlaczego to utrudnia zmianę w legacy. Jeśli cyklu nie ma — napisz to wprost jako wynik pozytywny, nie szukaj na siłę.

Ograniczenia jdeps zapisz jako unknowns: refleksja, DI, ClassRealm pluginów, dynamiczne ładowanie — graf statyczny może nie pokazać realnego sprzężenia runtime.

Format odpowiedzi:
- Bez grafu SVG (ani pośredniego DOT) na tym etapie — format SVG dopiero opcjonalnie w `m4l2-04`.
- Markdown: 3–5 najważniejszych obserwacji, potem tabela:
  - Obszar
  - Co znalazłeś (cykl / podejrzana zależność / brak)
  - Dowód (jdeps / mvn)
  - Dlaczego to ważne przy zmianie
  - Związek z artifact-1-territory.md
  - Co sprawdzić dalej
Nie zapisuj jeszcze artifact-2-structure.md.
```
