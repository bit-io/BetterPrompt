# BetterPrompt

Biblioteka do budowania interaktywnych promptów w terminalu dla **H#** (menedżer pakietów **Bit**),
inspirowana Pythonowym [`prompt_toolkit`](https://github.com/prompt-toolkit/python-prompt-toolkit).
Zero zależności poza standardową biblioteką H#; działa w skompilowanych, statycznych binarkach.

```h#
use "bit -> BetterPrompt"

fn main() is
    let s: betterprompt::PromptSession = betterprompt::new_session()
    betterprompt::set_message_html(s, "<ansigreen>user</ansigreen><b>❯</b> ")
    betterprompt::add_completer(s, betterprompt::words_completer(["help", "hello", "exit"]))
    s.auto_suggest = 1
    let line: string = betterprompt::prompt(s)
    println("wpisano: " + line)
end
```

## Co potrafi

| Funkcja | prompt_toolkit | BetterPrompt |
|---|---|---|
| Edycja linii (Emacs: Ctrl-A/E/B/F/K/U/W/Y/T, Alt-B/F/D/U/L/C, undo) | ✔ | ✔ |
| Tryb Vi (liczniki, `d`/`c`/`y` + ruchy, `iw`/`aw`, `f`/`t`, `p`, `u`, `J`, `~`, `r`) | ✔ | ✔ (podzbiór) |
| Historia w pamięci i w pliku (format `FileHistory`), strzałki, szukanie po prefiksie | ✔ | ✔ |
| Wyszukiwanie przyrostowe Ctrl-R / Ctrl-S | ✔ | ✔ |
| Uzupełnianie: słowa (+opisy), ścieżki, fuzzy, zagnieżdżone, własne; menu pod kursorem | ✔ | ✔ |
| Auto-sugestie z historii (Prawa strzałka / Ctrl-E / Alt-F) | ✔ | ✔ |
| Walidatory (liczba, regex, niepusty, własny) | ✔ | ✔ |
| Wieloliniowość (Esc+Enter akceptuje), prompt kontynuacji | ✔ | ✔ |
| Hasła (maska), placeholder, `erase_when_done`, `default` | ✔ | ✔ |
| Toolbar na dole (także liczony dynamicznie), `rprompt` | ✔ | ✔ |
| Style, klasy, kolory ANSI / 256 / truecolor z automatycznym obniżaniem | ✔ | ✔ |
| Formatowanie: `HTML(...)`, `ANSI(...)`, `print_formatted_text` | ✔ | ✔ (`html_fragments`, `print_html`, …) |
| Podświetlanie składni (lexer) | Pygments | prosty lexer (słowa kluczowe, liczby, napisy, komentarze) |
| Własne skróty klawiszowe i sekwencje (`c-x c-e`) | ✔ | ✔ |
| Wklejanie (bracketed paste), znaki szerokie CJK, zawijanie linii | ✔ | ✔ |
| Edycja w `$EDITOR` (Ctrl-X Ctrl-E) | ✔ | ✔ |
| `confirm`, `choice`, `message/yes_no/button/input/radiolist/checkboxlist_dialog` | ✔ | ✔ |
| Pasek postępu | ✔ | ✔ (`progress_*`, jedna linia) |
| asyncio, myszka, układy okien (`Layout`, `HSplit`…), pełnoekranowe aplikacje | ✔ | ✘ (poza zakresem) |

## Instalacja

W `Bit.hk` projektu:

```
[dependencies]
-> BetterPrompt => path ../betterprompt
```

(lub `git => https://…/betterprompt`, gdy repozytorium jest opublikowane). Potem `bit build` / `bit run`.
W kodzie: `use "bit -> BetterPrompt"` i wywołania `betterprompt::nazwa(...)`.

Wymagania: H# ≥ 0.9, Linux, terminal zgodny z VT100/xterm, `stty` w `PATH`.

## API — skrót

**Sesja**

```h#
let s: betterprompt::PromptSession = betterprompt::new_session()
betterprompt::set_message(s, "> ")                    // tekst
betterprompt::set_message_html(s, "<b>x</b>> ")       // ze znacznikami
let line: string = betterprompt::prompt(s)
betterprompt::session_interrupted(s)                  // Ctrl-C
betterprompt::session_eof(s)                          // Ctrl-D na pustej linii
betterprompt::ask("Imię: ")                           // jednorazowo
betterprompt::ask_password("Hasło: ")
```

Pola `PromptSession` (ustawiane bezpośrednio): `default_text`, `multiline`, `vi_mode`, `is_password`, `mask`,
`auto_suggest`, `complete_while_typing`, `complete_common`, `enable_history_search`, `history`, `validator`, `lexer`,
`style`, `continuation`, `erase_when_done`, `bracketed_paste`, `menu_height`, `skip_dup_history`, `open_in_editor`,
`cursor_shape`.

Dodatki: `set_bottom_toolbar_html`, `set_bottom_toolbar_fn(s, f)`, `set_rprompt_html`, `set_placeholder_html`,
`add_completer`, `add_binding`.

**Uzupełnianie**: `words_completer(words)`, `words_completer_meta(words, metas)`, `path_completer()`,
`fuzzy_completer(words)`, `nested_completer(["show ip route", …])`, `custom_completer(f)` gdzie
`f: fn(string) -> [string]` dostaje tekst przed kursorem i zwraca kandydatów na ostatnie słowo.
Pola `ignore_case`, `match_middle`, `sentence`, `only_dirs` w strukturze `Completer`.

**Walidacja**: `validator_integer`, `validator_number`, `validator_nonempty`, `validator_regex(wzór, komunikat)`,
`validator_custom(f)` gdzie `f: fn(string) -> string` zwraca `""` dla poprawnych danych albo komunikat błędu.

**Historia**: `new_history()`, `file_history("~/.moja_historia")`, `history_append`.

**Styl i formatowanie**: `new_style()`, `style_add(st, "completion-menu.current", "bg:#00aaaa fg:black bold")`,
`html_fragments`, `ansi_fragments`, `html_escape`, `strip_ansi`, `print_html`, `print_html_style`, `print_ansi`,
`print_fragments`, `clear_screen`, `set_title`.

Znaczniki HTML: `<b> <i> <u> <s> <reverse> <dim> <blink>`, `<ansired>`/`<ansibrightblue>`/… (kolor tekstu),
`<style fg='#ff8800' bg='black' bold='true'>`, dowolna nazwa `<nazwa>` = klasa stylu (`class:nazwa`).
Specyfikacja stylu jak w prompt_toolkit: `bold italic underline reverse strike fg:#rrggbb bg:ansired class:a.b`.
Głębię kolorów wykrywa `COLORTERM`/`TERM`; wymusisz ją przez `BETTERPROMPT_COLOR_DEPTH=1|4|8|24`; `NO_COLOR` wyłącza kolory.

**Skróty**: `confirm(msg, default)`, `choice(msg, opcje, domyślna)`; **dialogi**: `message_dialog`, `yes_no_dialog`,
`button_dialog`, `input_dialog` → `DialogResult{ok,text}`, `radiolist_dialog`, `checkboxlist_dialog` → `DialogResult{ok,selected}`;
**postęp**: `progress_new`, `progress_advance`, `progress_set`, `progress_finish`.

**Własne skróty**

```h#
fn wstaw_date(b: betterprompt::Buffer) -> int is
    betterprompt::buffer_insert(b, "2026-01-01")
    return 1                       // 0 nieobsłużone, 1 obsłużone, 2 akceptuj, 3 przerwij, 4 EOF
end
betterprompt::add_binding(s, "c-t", wstaw_date)   // nazwy jak w prompt_toolkit: c-a, m-b, f2, up, s-tab, "c-x c-e"…
```

## Domyślne skróty klawiszowe

| Klawisz | Działanie | Klawisz | Działanie |
|---|---|---|---|
| Ctrl-A / Home | początek linii | Ctrl-K | utnij do końca linii |
| Ctrl-E / End | koniec linii (przyjmuje sugestię) | Ctrl-U | utnij do początku linii |
| Ctrl-B / ← , Ctrl-F / → | znak wstecz / naprzód | Ctrl-W | utnij słowo wstecz |
| Alt-B / Ctrl-←, Alt-F / Ctrl-→ | słowo wstecz / naprzód | Alt-D, Alt-Backspace | utnij słowo naprzód / wstecz |
| Ctrl-Y, Alt-Y | wklej / rotuj kill ring | Ctrl-T | zamień znaki |
| Alt-U / Alt-L / Alt-C | wielkie / małe / kapitalizacja słowa | Ctrl-_ , Ctrl-X Ctrl-U | cofnij |
| ↑ / Ctrl-P, ↓ / Ctrl-N | historia (lub linie w trybie wieloliniowym) | Alt-< , Alt-> | pierwszy / ostatni wpis historii |
| Ctrl-R / Ctrl-S | szukaj wstecz / naprzód | Alt-. | ostatni argument z historii |
| Tab / Shift-Tab | uzupełnij / poprzednia opcja | Esc | zamknij menu |
| Enter | akceptuj (w trybie wieloliniowym: nowa linia) | Esc Enter (Alt-Enter) | akceptuj wieloliniowy |
| Ctrl-C | przerwij (status 1) | Ctrl-D | EOF na pustej linii, inaczej kasuj znak |
| Ctrl-L | wyczyść ekran | Ctrl-X Ctrl-E | edytuj w `$EDITOR` |

W trybie Vi (`s.vi_mode = 1`): `Esc` → tryb normalny; `h l w b e W B E 0 ^ $ gg G f t F T ; ,`, `d c y` + ruch/`iw`/`aw`,
`dd cc yy`, `x X D C s S p P u Ctrl-R J ~ r i a I A o O`, liczniki (`3dw`), `/` szuka w historii.

## Struktura repozytorium

```
Bit.hk                manifest pakietu
src/lib.h#            publiczne API (wrappery betterprompt::…)
src/utf8.h#           UTF-8, szerokości znaków
src/term.h#           tryb surowy (stty), nieblokujący odczyt (read(2)), dekodowanie klawiszy
src/style.h#          style, kolory, HTML/ANSI
src/buffer.h#         bufor, undo/redo, kill ring
src/vi.h#             tryb Vi
src/history.h# complete.h# validate.h# lexer.h#
src/render.h#         płótno: zawijanie, kursor
src/session.h#        pętla zdarzeń, renderer, menu, szukanie
src/shortcuts.h# dialogs.h# progress.h#
examples/             4 przykłady (hello, repl, form, dialogs)
tests/unit/           73 testy jednostkowe w H#     (cd tests/unit && bit run)
tests/pty/            43 testy integracyjne w PTY   (cd tests/pty/app && bit build && cd .. && python3 run_tests.py)
```

## Testy

```
cd tests/unit && bit run                        # passed=73 failed=0
cd tests/pty/app && bit build && cd .. && pip install pyte && python3 run_tests.py   # 43 ok, 0 błędów
```

## Ograniczenia i uwagi o H# 0.9

Biblioteka została napisana tak, żeby omijać błędy kompilatora LLVM-backend H# 0.9 napotkane w trakcie prac
(przydatne, jeśli rozwijasz ją dalej):

* literały całkowite ≥ 128 wpisane bezpośrednio w wyrażeniu z `int` (np. `b < 128`, `n > 200`) są obcinane do i8 —
  używamy stałych `const` albo argumentów funkcji (`in_rng(v, lo, hi)`);
* `push` na tablicy przekazanej jako parametr funkcji kończy się segfaultem — tablice mutujemy tylko jako pola struktur;
* enumy z danymi i porównanie `==` enumów dają złe wyniki w `match`/`if`; klawisze są stringami, tryby intami;
* wskaźnik funkcji zwracający `bool` zwraca -1/0 (callbacki używają `int`/`string`/`[string]`); tablica wskaźników
  funkcji i `let f = s.pole` nie działają; domknięcia z przechwytywaniem nie są zaimplementowane;
* `arr.remove()` powoduje segfault, `.pop()` nie jest zaimplementowane; `to_string(now_unix())` bez zmiennej pośredniej się wywraca;
* `end`, `bytes` są słowami zastrzeżonymi.

Pozostałe ograniczenia BetterPrompt:

* pojedynczy Esc rozpoznawany jest po 100 ms bez kolejnych bajtów (jak w readline/`ttimeoutlen`);
* menu uzupełniania jest kolumnowe i pojawia się pod całym wejściem (nie pływa pod kursorem w środku wieloliniowego tekstu);
* przy zmniejszeniu okna terminal sam zawija stare wiersze — bieżący prompt rysowany jest od nowej linii;
* kolorystyka składni to prosty lexer, nie Pygments; brak myszki i układów okien;
* gdy stdin nie jest terminalem, `prompt` czyta zwykłą linię (bez edycji), a `confirm`/`choice` czytają odpowiedź tekstową;
  dialogi w takim trybie zwracają wartości domyślne.

## Licencja

MIT — zob. `LICENSE`.
