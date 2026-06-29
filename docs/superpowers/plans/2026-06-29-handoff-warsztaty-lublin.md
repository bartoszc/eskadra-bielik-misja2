# Handoff — warsztaty Eskadra Bielika (Misja 2), Lublin

> **Dla kolejnej instancji Claude (drugi komputer):** ten dokument to przekazanie kontekstu z sesji z 2026-06-29. Zakłada zero wcześniejszego kontekstu. Przeczytaj go w całości, zanim zaczniesz cokolwiek robić. Najważniejsze pliki źródłowe są wskazane ścieżkami — otwórz je.

**Cel sesji prowadzącego:** przeprowadzić 2-godzinny warsztat (codelab) „serverless RAG z Bielikiem na GCP" w Lublinie.

---

## 0. Czego NIE ma na drugim komputerze

Te rzeczy żyją w `~/.claude/` pierwszego komputera i **nie przyjdą z repo**:
- **Hook `SessionEnd`** (`~/.claude/settings.json`) wypisujący `claude --resume <id>` + ID + ścieżkę transkryptu na koniec sesji. Jeśli chcesz to samo tutaj — odtwórz z pamięci poniżej.
- **Pamięć projektu** (`~/.claude/projects/<sanitized-cwd>/memory/`), m.in. `sessionend-resume-hook.md`.

Wszystko inne istotne jest w repo (sekcja 2) — pod warunkiem, że zostało wypushowane (sekcja 1).

---

## 1. Stan repo / git (sprawdź NAJPIERW)

Na 2026-06-29 na pierwszym komputerze materiały warsztatowe były **nieśledzone i niewypushowane**:
- `docs/`, `eskadra-bielika-closed.md`, `CLAUDE.md`, szablon `.pptx` — wszystko untracked.
- `origin` = `git@github.com:bartoszc/eskadra-bielik-misja2.git`, branch `main`.

**Zanim ruszysz na drugim komputerze:** `git pull` i sprawdź, czy `docs/warsztaty/` oraz `eskadra-bielika-closed.md` faktycznie są. Jeśli ich nie ma → nie zostały wypushowane z pierwszego komputera; trzeba tam wrócić, zacommitować i wypushować.

---

## 2. Mapa materiałów warsztatowych (`docs/warsztaty/`)

- `plan-slajdow.md` — plan slajd-po-slajdzie z adnotacjami KEEP / FILL / NEW / CUT. Kluczowe nowe slajdy: **8a „Jak działa RAG?"**, **8b „Architektura"**. Do uzupełnienia ręcznie: slajd 1 (tytuł), 6 (kupony — patrz uwaga w sekcji 3!), 18 (agenda).
- `scenariusz-prowadzacego.md` — scenariusz na 2h: 10 bloków z timingiem, komendy per blok, plan awaryjny GPU, checklista prowadzącego.
- `tipy-z-discorda.md` — **synteza z kanału trenerów** (1 kwi–27 cze, ~59 warsztatów): (A) jak radzić sobie z wyzwaniami, (B) narzędzia/repo/rankingi z datami i linkami. **To jest najświeższy wkład tej sesji — przeczytaj.**
- `discord-export.txt` — krótki surowy fragment (22 linie).
- `../superpowers/specs/2026-06-28-warsztaty-prezentacja-scenariusz-design.md` — wcześniejszy spec.
- `../../eskadra-bielika-closed.md` (root repo) — pełny eksport kanału Discord (876 linii), źródło syntezy.

---

## 3. Najważniejsze wnioski operacyjne (stan na koniec czerwca 2026)

To są realia, które MUSZĄ trafić do prowadzenia warsztatu. Pełne uzasadnienia i daty: `docs/warsztaty/tipy-z-discorda.md`.

1. **Kupony GCP NIE działają.** Od końca maja jedzie się na **kontach „Frictionless"**: gotowe login/hasło drukowane na karteczki, logowanie w **incognito** na `console.cloud.google.com`, projekt już gotowy. Obowiązują **do końca czerwca**. → Slajd 6 („kupony") i checklista są pod tym kątem **nieaktualne** — wymagają przeróbki na „konta Frictionless".
2. **Najczęstsza wpadka z kontami:** uczestnik zapomina **wybrać projekt** po zalogowaniu.
3. **Uprawnienia — złota zasada:** Gmail = istniejący (nie nowy); projekt GCP = zawsze **nowy, BEZ organizacji**. Projekt w firmowej organizacji → błędy uprawnień (BigQuery) i brak GPU.
4. **Quota / brak GPU L4:** fallbacki w kolejności — wspólna usługa `bielik` prowadzącego (URL Dawida: `https://bielik-3342661l5038.europe-west1.run.app`, embedding: `https://embedding-gemma-3342661l5038.europe-west1.run.app`) → deploy na CPU (`--cpu 8 --no-cpu-throttling --memory 16Gi --timeout=3600`) → ponowny deploy → **wariant Colab** (`github.com/Legardt777/eskadra-bielik-misja2/blob/main/notebooks/warsztat_colab.ipynb`).
5. **Błąd CUDA:** wymuś `ollama/ollama:0.23.4` w Dockerfile (nowsze tagi mają regresję). Sprawdź, czy to repo ma poprawkę.
6. **QR do Discorda w szablonie prezentacji jest WYGASŁY** — użyj linku ze stopki `bielik.ai` / `eskadra.bielik.ai`.
7. **Dydaktyka:** wspólne tempo sali, plan krok-po-kroku, pokaż self-debug (Cloud Run + BigQuery), obecność na Lumie na starcie, nie wysyłaj własnej ankiety (zabija wypełnialność centralnej).

---

## 4. Co jest OTWARTE (do dokończenia)

- [ ] **Wpleść troubleshooting (sekcja 3) do `scenariusz-prowadzacego.md`** — rozbudować sekcję „Plan awaryjny GPU" w pełny „Troubleshooting i plan B" i zaktualizować checklistę (kupony → konta Frictionless, Ollama 0.23.4, wygasły QR). *Gotowy draft tej edycji powstał w sesji 2026-06-29, ale został wycofany na prośbę użytkownika — decyzję podejmij z nim.*
- [ ] **Zaktualizować slajdy FILL** w `plan-slajdow.md`: 1 (tytuł), 6 (zmienić „kupony" → „konta Frictionless"), 18 (agenda).
- [ ] **Repo z „tablicą wyników" (leaderboard):** w eksporcie Discorda jest tylko wzmianka (greg, Gliwice #51, 19/06) — bez linku do repo. Jeśli potrzebne, poszukać u greg / LegardT77.
- [ ] (opcjonalnie) Próbny przebieg scenariusza pod kątem timingu 2h.

---

## 5. Jak odtworzyć hook „wznów sesję" (jeśli chcesz go na drugim komputerze)

Dodaj do `~/.claude/settings.json` w `hooks.SessionEnd` komendę (pisze na `/dev/tty`, bo stdout SessionEnd bywa nierenderowany; nie używaj `IFS=$'\t'` w stringu JSON — psuje walidator):

```
jq -r '.session_id, .transcript_path' | { read -r sid; read -r tpath; printf '\n\033[1;36m↺ Aby wznowić tę sesję:\033[0m  claude --resume %s\n\033[2m   ID sesji:    %s\n   Transkrypt:  %s\033[0m\n\n' "$sid" "$sid" "$tpath" > /dev/tty; } 2>/dev/null || true
```
