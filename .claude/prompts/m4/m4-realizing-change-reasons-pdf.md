# Prompt: PDF „dlaczego refactor-opportunities”

Użyj tego pliku, gdy chcesz **wygenerować od zera** albo **odświeżyć** notatkę PDF:

`.claude/notes/m4/m4-realizing-change-reasons.pdf`

PDF tłumaczy laikowi, *dlaczego* robimy zmianę `refactor-opportunities`, zaczynając od mapy projektu i idąc do decyzji planu (K2).

---

## Kiedy odświeżać

Odśwież PDF, gdy zmieni się którykolwiek z poniższych źródeł (albo gdy użytkownik poprosi o refresh):

| # | Źródło | Rola w narracji |
|---|--------|-----------------|
| 1 | `context/map/repo-map.md` | Punkt startu: warstwy, strefy ryzyka, most M3↔M4 |
| 2 | `context/changes/model-builder-flow-analysis/change.md` (+ `research.md` jeśli istnieje i jest aktualny) | Prior: Deep Focus / dług ścieżki ModelBuildera |
| 3 | `context/changes/refactor-opportunities/change.md` | Intencja i etapy zmiany |
| 4 | `context/changes/refactor-opportunities/research.md` | Ranking K1/K2/K3, intencjonalność, odrzucone |
| 5 | `context/changes/refactor-opportunities/plan.md` | Zakres realizacji (obecnie pełne K2) |
| 6 | `context/changes/refactor-opportunities/plan-brief.md` | Skrót decyzji |
| 7 | `context/changes/refactor-opportunities/reviews/plan-review.md` | Werdykt review (jeśli istnieje) |

Jeśli status w frontmatter `change.md` lub zakres planu się zmienił (np. K1 weszło w scope), **zaktualizuj narrację** — nie kopiuj ślepo starego PDF.

---

## Zadanie dla agenta

1. Przeczytaj wszystkie źródła z tabeli powyżej (pomiń brakujące, ale zanotuj w PDF, że źródło było nieobecne).
2. Zsyntetyzuj **szczegółową notatkę po polsku, jak dla laika**: pełny kontekst od `repo-map`, bez zakładania wiedzy o Mavenie / 10x / M4L*.
3. **Nadpisz** plik `.claude/notes/m4/m4-realizing-change-reasons.pdf` (utwórz katalog, jeśli trzeba).
4. Nie commituj ani nie pushuj, chyba że użytkownik explicite o to poprosi.
5. Na koniec podaj: ścieżkę PDF, liczbę stron, krótkie TL;DR co się zmieniło względem poprzedniej wersji (jeśli da się porównać).

---

## Wymagana struktura treści PDF

Zachowaj tę ścieżkę rozumowania (można dopisać sekcje, nie wolno wyciąć logiki „mapa → dług → ranking → decyzja”):

1. **Czym jest Maven** — analogia dla laika; POM; napięcie Maven 3 vs 4.
2. **Skąd mapa (`repo-map`)** — cel onboardingu; warstwy cli → core → impl → api / compat; ITS ≠ centrum produktu.
3. **Strefy ryzyka z mapy** — szczególnie strefa 1 (ModelBuilder), strefa 2 (lifecycle/reactor), most M3↔M4.
4. **Prior: `model-builder-flow-analysis`** — Deep Focus zostawił pytanie „co naprawiamy?” otwarte.
5. **Czym jest `refactor-opportunities`** — eksploracja → decyzja/plan → implementacja; aktualny `status` z `change.md`.
6. **K1 / K2 / K3 w prostych słowach** + ranking z researchu.
7. **Dlaczego wybrano zakres z planu** (dziś: pełne K2 guardy; jeśli plan się zmieni — opisz *aktualny* wybór i powody).
8. **Co w scope / poza scope** według aktualnego `plan.md` (+ brief).
9. **Jak to łączy się z „realizing change”** — małe, uziemione w dowodach kroki; guard-first.
10. **Podsumowanie dla kogoś spoza zespołu** + lista źródeł w repo.

Na okładce: ramka „W jednym zdaniu” z aktualnym werdyktem (nie kopiuj historycznego, jeśli decyzja się zmieniła).

Ton: bezpośredni, konkretny, mało żargonu; żargon (FATAL, twin, personality) wyjaśniaj przy pierwszym użyciu.

---

## Jak technicznie wygenerować PDF (receptura użyta pierwotnie)

### Narzędzia

- Python 3 + **reportlab** (`pip install reportlab` jeśli brakuje).
- Opcjonalnie **pypdf** do weryfikacji liczby stron i obecności polskich znaków.
- Skill PDF (Cursor/Claude): generowanie przez reportlab Platypus, nie przez „wydruk HTML”.

### Polskie znaki

Standardowe fonty Helvetica w reportlab **nie mają** polskich glifów. Zarejestruj TTF z Unicode, np. na macOS:

```text
/System/Library/Fonts/Supplemental/Arial Unicode.ttf
```

```python
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.ttfonts import TTFont
pdfmetrics.registerFont(TTFont("ArialUni", "/System/Library/Fonts/Supplemental/Arial Unicode.ttf"))
```

Używaj `fontName="ArialUni"` we wszystkich stylach Paragraph. Nie używaj Unicode sub/superscript w reportlab (czarne prostokąty) — tu i tak niepotrzebne.

### Layout

- Format **A4**, marginesy ~2 cm.
- `SimpleDocTemplate` + `Paragraph` / `Spacer` / `Table` / `HRFlowable` / `PageBreak`.
- Style własne (np. `H1PL`, `BodyPL`) — unikaj kolizji z domyślnymi nazwami reportlab (`Bullet`, `Normal`, …).
- Numer strony w stopce od strony 2; tytuł/metadata w dokumencie PDF.
- Język treści: **polski**.

### Weryfikacja po zapisie

- Plik istnieje pod `.claude/notes/m4/m4-realizing-change-reasons.pdf`.
- Liczba stron sensowna (pierwsza wersja: ~9; po zmianach może być inna).
- W tekście PDF (extract) są polskie znaki (`ą`, `ę`, `ł`, …) oraz kluczowe tokeny: `refactor-opportunities`, `repo-map`, `K2` (lub aktualny wybór planu).

### Przykładowy szkic wywołania

```bash
mkdir -p .claude/notes/m4
python3 <<'PY'
# 1) wczytaj/zsyntezuj treść ze źródeł context/...
# 2) zbuduj story (Paragraphs) jak wyżej
# 3) doc.build(...) → .claude/notes/m4/m4-realizing-change-reasons.pdf
PY
```

Nie commituj skryptu generującego do repo Apache Maven, chyba że użytkownik tego chce — receptura ma żyć w tym prompcie / lokalnych notatkach `.claude/`.

---

## Prompt one-liner (do wklejenia)

```text
Odśwież PDF według .claude/prompts/m4/m4-realizing-change-reasons-pdf.md:
przeczytaj aktualne źródła context/map/repo-map.md oraz context/changes/{model-builder-flow-analysis,refactor-opportunities}/,
wygeneruj na nowo .claude/notes/m4/m4-realizing-change-reasons.pdf (PL, dla laika, reportlab + Arial Unicode), nadpisz stary plik.
```

---

## Historia

| Data | Uwaga |
|------|--------|
| 2026-10-14 | Pierwsza wersja PDF + ten prompt. Narracja: mapa → prior ModelBuilder → ranking K2→K1→K3 → plan pełne K2 (guard twin/personality), status `plan_reviewed`. |
