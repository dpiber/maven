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

Zachowaj tę ścieżkę rozumowania (można **dopisać** sekcje, nie wolno wyciąć logiki „mapa → dług → ranking → decyzja”):

1. **Czym jest Maven** — analogia dla laika; POM; napięcie Maven 3 vs 4.
2. **Skąd mapa (`repo-map`)** — cel onboardingu; warstwy cli → core → impl → api / compat; ITS ≠ centrum produktu.
3. **Strefy ryzyka z mapy** — szczególnie strefa 1 (ModelBuilder), strefa 2 (lifecycle/reactor), most M3↔M4.
4. **Prior: `model-builder-flow-analysis`** — Deep Focus zostawił pytanie „co naprawiamy?” otwarte.
5. **Czym jest `refactor-opportunities`** — eksploracja → decyzja/plan → implementacja; aktualny `status` z `change.md`.
6. **K1 / K2 / K3 w prostych słowach** + ranking z researchu.
7. **Dlaczego wybrano zakres z planu** (dziś: pełne K2 guardy; jeśli plan się zmieni — opisz *aktualny* wybór i powody).
8. **Co w scope / poza scope** według aktualnego `plan.md` (+ brief).
9. **(Gdy status ≥ implementing / implemented)** Co zaimplementowano — rozbicie na fazy + szczegóły kodu (patrz niżej).
10. **Jak to łączy się z „realizing change”** — małe, uziemione w dowodach kroki; guard-first.
11. **Podsumowanie dla kogoś spoza zespołu** + lista źródeł w repo.
12. **Historia zmian** — tabela wersji PDF / narracji (data, co się zmieniło); aktualizuj przy każdym odświeżeniu.

Na okładce: ramka „W jednym zdaniu” z aktualnym werdyktem (nie kopiuj historycznego, jeśli decyzja się zmieniła).

Tuż po okładce / meta: **klikalny spis treści** (linki wewnętrzne do kotwic sekcji + outline/bookmarks w panelu PDF).

Ton: bezpośredni, konkretny, mało żargonu; żargon (FATAL, twin, personality) wyjaśniaj przy pierwszym użyciu.

---

## Reguły odświeżania (obowiązkowe)

### Nie kompresuj narracji

- Pierwsza wersja PDF miała **~9 stron** pełnej prozy dla laika. Odświeżenie **nie może** skracać sekcji 1–8 / realizing / podsumowania „żeby zmieścić nowość”.
- Nowe treści (np. implementacja) **dokładasz** jako dodatkowe sekcje / strony. Docelowa objętość po dodaniu implementacji: **≥ pierwszej wersji**, typowo **więcej** (np. 11–13 stron), nie mniej.
- **Zakaz:** przepisywanie całej notatki na skróty, łączenie akapitów w „TL;DR”, usuwanie tabel warstw / rankingów / analogii.
- Przy odświeżeniu: bazuj na **kanonicznym layoutcie i treści pierwszej wersji** (commit `4f44e945ca` / generator z tamtej sesji), potem **aktualizuj statusy** i **dopisuj** nowe sekcje.

### Trzymaj jeden layout = pierwsza wersja (+ wyróżnienia hierarchii)

Kanoniczna baza (format, proza, callouty, tabele) = pierwsza wersja. Hierarchia nagłówków i nawigacja — jak niżej (obowiązkowe przy każdym odświeżeniu):

| Element | Wartość |
|---------|---------|
| Format | A4, marginesy ~2 cm |
| Font tekstu PL | Arial Unicode (`ArialUni`) |
| Kolory bazowe | INK `#1a1a1a`, MUTED `#444444`, RULE `#cccccc`, BOX_BG `#f5f5f5`, BOX_BORDER `#333333` |
| Kolor sekcji (H1) | `HEADING1` `#0b3d5c`, **fontSize 15–16**, leading ~20, spaceBefore ~14 |
| Kolor podsekcji (H2) | `HEADING2` `#2c5282`, **fontSize 12**, leading ~16, spaceBefore ~10 |
| Tytuł okładki | INK, fontSize 22 (bez zmiany na HEADING1) |
| Body / bullet | INK, fontSize 10 — **nie** koloruj akapitów jak nagłówki |
| Style | `cover_*` / `h1` / `h2` / `body` / `bullet` / `box` / `caption` / `toc` (link) / `table_*` / `quote` |
| Helpery | `P()`, `bullet()`, `callout(...)`, `simple_table(...)`, `code_block(...)`, `hr()`, `h1(anchor, title)` / `h2(...)` z kotwicą, stopka od s. 2 |
| Okładka | kicker → tytuł → sub → `hr()` → meta → callout „W jednym zdaniu” → **klikalny spis** → treść |

**Sekcje vs podsekcje:** zawsze rozróżnialne **kolorem i rozmiarem** (H1 ≠ H2 ≠ body). Nie zostawiaj wszystkich nagłówków w kolorze INK.

### Klikalny spis treści (obowiązkowy)

- Spis na początku (po calloucie okładki), przed sekcją 1.
- Każdy wpis = link wewnętrzny (`<link href="#sec-N">…</link>` albo równoważne destination reportlab) do kotwicy przy H1.
- Przy każdym H1: `<a name="sec-N"/>` (lub `bookmarkPage` + outline).
- Dodaj **PDF outline / bookmarks** (panel nawigacji w czytniku) dla sekcji H1; opcjonalnie H2.
- Wizualnie: wpisy TOC w kolorze `HEADING1`, czytelne prowadzenie (numer + tytuł); nie zwykły czarny tekst bez linku.
- Weryfikacja: w PDF są klikalne linki / destinations (np. `pypdf` outlines lub ręczny smoke w Preview).

### Historia zmian (obowiązkowa sekcja w PDF)

- Ostatnia sekcja (lub przed stopką źródeł): **„Historia zmian”**.
- Tabela: Data | Wersja / trigger | Co się zmieniło w narracji lub zakresie (1–2 zdania).
- Wiersze minimum: pierwsza wersja (plan_reviewed) → odświeżenia (implemented, layout, TOC, …).
- Synchronizuj z tabelą **Historia** na dole **tego** pliku promptu — PDF ma być czytelnym skrótem dla laika, prompt trzyma pełniejszy log agentski.

### Formatowanie zmian w kodzie

Gdy pokazujesz dodany/zmieniony kod (sekcja implementacji / fazy):

- **Nie** wklejaj snippetów jako zwykłego akapitu `body` ani samego `<font face='Courier'>…</font>` w środku prozy.
- Użyj bloku **`code_block(title, lines)`**:
  - etykieta nad blokiem (ścieżka pliku + faza / „dodano” / „zmieniono”),
  - tło `#f0f0f0`, ramka + **lewy accent** 3 pt `#333333`,
  - font **Courier** (kod ASCII/Java), rozmiar ~8–8.5, leading ~10–11,
  - każda linia osobno; wcięcia przez spacje niełamliwe (`&nbsp;`) albo zachowane w `Preformatted`,
  - escape XML (`&`, `<`, `>`),
  - krótkie snippety (kilka–kilkanaście linii); dłuższe diffy streszczaj listą metod + 1–2 bloki reprezentatywne.
- Obok bloku: 1–2 zdania po polsku *co to chroni* (nie tylko „dodano test”).

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

Używaj `fontName="ArialUni"` we wszystkich stylach Paragraph (proza). Do **bloków kodu** — Courier. Nie używaj Unicode sub/superscript w reportlab (czarne prostokąty).

### Layout

- Format **A4**, marginesy ~2 cm — szczegóły w tabeli „Trzymaj jeden layout” powyżej.
- `SimpleDocTemplate` + `Paragraph` / `Spacer` / `Table` / `HRFlowable` / `PageBreak` + helpery pierwszej wersji.
- Style własne (`H1PL`, `BodyPL`, …) — unikaj kolizji z domyślnymi nazwami reportlab.
- Numer strony w stopce od strony 2; tytuł/metadata w dokumencie PDF.
- Język treści: **polski**.

### Weryfikacja po zapisie

- Plik istnieje pod `.claude/notes/m4/m4-realizing-change-reasons.pdf`.
- Liczba stron: **nie mniejsza** niż pierwsza wersja (~9), gdy dodano implementację — **więcej**.
- Extract zawiera polskie znaki (`ą`, `ę`, `ł`, …) oraz tokeny: `refactor-opportunities`, `repo-map`, `K2` (lub aktualny wybór planu).
- Jeśli jest sekcja implementacji: w extract widać nazwy plików/metod **oraz** fragmenty z bloków kodu (np. `warn =`, nazwy testów).
- Nagłówki H1/H2 wizualnie inne niż body (kolor + rozmiar); spis ma linki; sekcja „Historia zmian” obecna.

### Przykładowy szkic wywołania

```bash
mkdir -p .claude/notes/m4
python3 <<'PY'
# 1) wczytaj/zsyntezuj treść ze źródeł context/...
# 2) zbuduj story jak pierwsza wersja (ten sam layout), aktualizując statusy
# 3) dopisz sekcje (np. implementacja + code_block), nie kompresuj 1–8
# 4) doc.build(...) → .claude/notes/m4/m4-realizing-change-reasons.pdf
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
| 2026-10-14 | Odświeżenie po implementacji (skompresowane — **odrzucone** jako wzorzec): status `implemented`, ale za krótko vs v1. |
| 2026-10-14 | Reguły: nie kompresuj; layout = v1; `code_block` dla snippetyów. PDF przebudowany z pełnej prozy v1 + sekcja implementacji fazami. |
| 2026-10-14 | Reguły: H1/H2 kolorem+rozmiarem; klikalny TOC + bookmarks; sekcja Historia zmian. PDF przegenerowany. |
