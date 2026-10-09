# M4L2 / Krok 2 — bullet 2: cienkie wejścia vs głębsze centra

Repo: https://github.com/apache/maven  
Cel: odróżnić cienkie wejścia (CLI, facade, thin adapters) od głębszych centrów zależności (wysoki fan-in / load-bearing).  
Uruchamiaj po `m4l2-01-structure.md`.

## Wymagania konieczne

- **Nie używaj skilli 10x** (np. `/10x-repo-map`, `/10x-init`, `/10x-research` ani żadnego innego skillu z paczki 10xDevs). Wykonaj zadanie ręcznie: komendy CLI + interpretacja w tej sesji.

```text
Wymagania konieczne: nie używaj skilli 10x (w tym /10x-repo-map, /10x-init i pozostałych). Pracuj ad hoc na CLI i grafie zależności (jdeps / Maven Dependency Plugin).

Pracujemy nadal na apache/maven. Masz artifact-1-territory.md oraz wnioski o kontraktach warstw z poprzedniego prompta.

Pytanie przewodnie: które pliki/moduły są cienkimi wejściami, a które wyglądają jak głębsze centra?

Na podstawie `jdeps` (summary + verbose:package, ewentualnie -apionly) oraz drzewa Maven (`dependency:tree` między modułami reactora) sklasyfikuj aktywne obszary z terytorium:

1. Cienkie wejścia (shallow) — mało własnej logiki w grafie, głównie delegacja dalej: np. CLI invoker, facade API, cienki bridge compat→impl. Wysokie Ce przy niskim Ca albo „tylko woła dalej”.
2. Głębsze centra (deep / load-bearing) — dużo zależnych (wysoki fan-in / Ca): inne moduły Mavena importują ten pakiet/JAR; zmiana kontraktu ma szeroki blast radius. Kandydaci z terytorium: maven-impl (model building), maven-core (lifecycle/project/plugin realm), api/maven-api-core.
3. Huby pośrednie — wysoki ruch w gicie, ale w grafie głównie orkiestracja (np. mvnup w maven-cli): nie myl aktywności terytorium z centralnością w grafie importów.

Dla każdej pozycji podaj:
- evidence z jdeps / dependency:tree (kto zależy / od kogo zależy; na poziomie pakietu lub modułu),
- inference (shallow vs deep / load-bearing vs contained),
- caution przed zmianą (jedno zdanie),
- unknowns (refleksja, ClassRealm, SPI, codegen .mdo — jeśli graf może kłamać).

Nie oceniaj jeszcze jakości ani nie proponuj refaktoryzacji. Nie renderuj grafu SVG na tym etapie — Markdown wystarczy (opcjonalny podgraf SVG dopiero w `m4l2-04`). Jeśli graf jest zbyt gęsty, zwiń do poziomu modułu/pakietu (odpowiednik --collapse), nie do setek klas.

Format: Markdown — sekcje `Cienkie wejścia`, `Głębsze centra`, `Huby mylące (aktywne ≠ centralne)`, tabela porównawcza, 3–5 wniosków decyzyjnych. Nie zapisuj jeszcze artifact-2-structure.md.
```
