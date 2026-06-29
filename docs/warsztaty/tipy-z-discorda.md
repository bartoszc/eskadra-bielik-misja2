# Tipy z Discorda trenerów Eskadry Bielika (Misja 2)

> Synteza z kanału `#eskadra-bielika-closed` (serwer SpeakLeash), zakres **1 kwietnia – 27 czerwca 2026**, ~59 warsztatów.
> Źródło: `eskadra-bielika-closed.md`. Dwa kąty: **(A)** jak trenerzy radzili sobie z wyzwaniami, **(B)** supportujące narzędzia / repo / rankingi.

---

## A. Jak trenerzy radzili sobie z wyzwaniami

### 1. Dostępność i quota GPU L4 (najczęstszy problem techniczny)
- **Objaw:** `You do not have quota for using GPUs without zonal redundancy` lub brak L4 w regionie (us-central4 / us-east2). Potrafiło zająć ~40 min (Poznań, 30/04).
- **Co działało:**
  - **Ponowny deploy** „na pałę" — czasem przechodzi za drugim razem (Płock, 14/05).
  - **Profilaktyczne podbicie limitów kwot z wyprzedzeniem** (Damian Siwicki, 01/05): `g.co/cloudrun/gpu-quota`.
  - **Deploy bez GPU na CPU** — wolniejszy, ale działa (Mateusz Jabłoński / LegardT77, 24/04):
    `--cpu 8 --no-cpu-throttling --memory 16Gi --timeout=3600 --concurrency 1 --max-instances 1` + `--set-env-vars OLLAMA_NUM_PARALLEL=1`.
  - **Współdzielony endpoint Bielika od Dawida** (patrz sekcja B).

### 2. Konta billingowe padające masowo (saga 07–26 maja)
- **Objaw:** kupony aktywowane, deploy przechodził, a potem Google zamykał konta billingowe całym salom (Katowice 07/05, Pruszków/Wrocław 09/05).
- **Przyczyna** (Dawid Ostrowski, 15/05 i 19/05): globalny system **antyfraudowy** Google — wstrzymano wydawanie kuponów, przyznane traciły ważność z dnia na dzień.
- **Jak sobie radzili:**
  - Plan B trenera (Mateusz Jabłoński, 12/05): (1) sesja bardziej teoretyczna, (2) własne „trenerskie" endpointy, (3) CPU zamiast GPU.
  - **Sygnał fraudu:** świeżo założone konto Gmail dostające od razu kredyty wygląda podejrzanie → **zachęcać do logowania istniejącym, starym Gmailem** (Dawid, 15–16/05).
  - **Finalne rozwiązanie — konta „Frictionless"** (Dawid, 26/05): gotowe login/hasło drukowane na karteczki i rozdawane. Logowanie **w trybie incognito** na `console.cloud.google.com`, projekt już gotowy, ~20 min do produktywnej pracy. Obowiązują **do końca czerwca** — to aktualny model.

### 3. Uprawnienia: projekt w organizacji vs. prywatny (problem #1)
- **Złota zasada** (Dawid, 16/05): rozdziel dwie rzeczy — **Gmail: używaj istniejącego**; **projekt GCP: zawsze nowy, BEZ organizacji**.
- Projekt w firmowej organizacji → błędy uprawnień (m.in. BigQuery), brak GPU. Doraźnie: ręczne nadanie roli **owner** po włączeniu API (LegardT77, 29/04), albo „zmiana organizacji na brak organizacji" (pawkis, 26/06).

### 4. Cloud Shell — zacięcia i okno autoryzacji
- Cloud Shell potrafił „zaciąć się" i odpowiedzieć po ~30 min (LegardT77, 29/04).
- Okno autoryzacji nie zawsze pojawia się automatycznie (MR, 24/04) — uprzedź uczestników.

### 5. Błąd CUDA / regresja Ollamy
- **Objaw:** `CUDA error: device kernel image is invalid` (ptynecki, 04/06).
- **Rozwiązanie** (Dawid, 05/06): wymusić **`ollama/ollama:0.23.4`** w Dockerfile — nowsze tagi / `:latest` mają regresję. emgiezet (18/06) też łapał błędy Ollamy związane z brakiem aktualnego kernela.

### 6. Model „ucieka" z polskiego
- Bielik po rebase / przez chwilę odpowiadał po **angielsku** (emgiezet, 18/06) i **rosyjsku** (Gdańsk, 18/06) — przejściowe.

### 7. Dydaktyka / prowadzenie sali
- **Plan krok po kroku** jest kluczowy — bez niego uczestnicy się gubią (Jakub, 22/04).
- **Wspólne tempo** całej sali eliminuje większość drobnych problemów (Marcin Rozmus, 16/05).
- **Pokaż self-debug:** gdzie w Cloud Run widać kontenery i gdzie w BigQuery dane (Marcin Rozmus, 16/05).
- **Catering zamawiaj na konkretną godzinę przed startem** (Bart #46, 17/06).
- **Obecność na Lumie** na starcie (do certyfikatów); QR do skanowania obecności działa dobrze (Grzegorz, 30/04).
- **Nie wysyłaj własnej ankiety** równolegle z centralną — wypełnialność centralnej spada z ~30% do <10% (MartaM, 19/06).

> ⚠️ **Aktualne na dziś:** kupony **nie działają** — jedziesz na **kontach „Frictionless"** (login/hasło na karteczkach, incognito). Najczęstsza wpadka: ludzie zapominają **wybrać projekt** po zalogowaniu (emgiezet, 24/06). QR do Discorda w szablonie prezentacji **jest wygasły** — działający link jest w stopce `bielik.ai` / `eskadra.bielik.ai`.

---

## B. Supportujące narzędzia, repo i rankingi (autor · data · link)

| Co to | Autor | Data | Link |
|---|---|---|---|
| **Repo bazowe warsztatu** „RAG na Bielik + GCP" | Dawid Ostrowski | 10/04 | `github.com/avedave/eskadra-bielik-misja2` |
| **„Szpergał warsztatowy"** — drzewko decyzyjne błędów RAG (Cloud Shell, BILLING_DISABLED, 403…) | emgiezet | 30/04 | `docs.google.com/document/d/1zfQDNfqRAVKK_HQEOMDHFHsXM-XarOfhAux_zHN2w/edit` |
| **Raport benchmarkowy / „ranking" modeli** — Bielik v3.4.5b vs Bielik-Minitron-7B (LLM-as-judge: Claude Opus 4.7, 50 pytań) | Damian Siwicki | 01/05 | `ylia.pl/eskadra-bielika-2.0-raport.html` |
| Benchmark na własnym RAG-u ubezpieczeniowym (30 promptów prawniczych) + rozszerzona prezentacja | emgiezet | 01/05 | Google Slides/Docs „…Poznań" (link w wątku) |
| **Harness ewaluacyjny na bazie RAGAS** (~161 promptów, surowy JSON do analizy) | emgiezet | 02/05 | (oferta udostępnienia w wątku) |
| **Materiały edukacyjne:** 2 filmy YT (Grzegorz Wasilewski) — „RAG ingestion steps" i „RAG query steps" + **interaktywne diagramy HTML** | LegardT77 | 06/05 | `github.com/Legardt777/eskadra-bielik-misja2/tree/main/assets/diagrams` |
| **Wariant Colab** całego warsztatu (T4, ChromaDB zamiast BigQuery, ~36 min, ten sam certyfikat) — kluczowy plan B | LegardT77 | 20/05 | `colab.research.google.com/github/Legardt777/eskadra-bielik-misja2/blob/main/notebooks/warsztat_colab.ipynb` |
| **Współdzielona tablica Excalidraw „Bielik2"** (diagramy do kopiowania/przeróbki) | Oleg Żero | 17–18/05 | `link.excalidraw.com/l/8DWzyEraOpF/i6GBDQOeK9Sh` |
| Skrypt weryfikacji uczestników na LinkedIn (Node.js + Playwright, CSV → dev/nie-dev) | emgiezet | 23/04 | (w wątku) |
| **Fallback — wspólny Bielik na GPU** (Cloud Run, bez auth) | Dawid Ostrowski | 12/05 | `https://bielik-3342661l5038.europe-west1.run.app` |
| **Fallback — Bielik + EmbeddingGemma** (oba publiczne) | Dawid Ostrowski | 19/05 | `https://embedding-gemma-3342661l5038.europe-west1.run.app` + jw. |
| Doc ze zbiorczymi pomysłami na plan B (collab) | Dawid Ostrowski | 19/05 | `docs.google.com/presentation/d/1hpJBpTuVjWOl9NdJAunSIidIgIdQrVinbcD1UBBwbGI/edit` |

**Aplikacje uczestników/trenerów (gotowe podglądy RAG):** `bielik-workshop.mma3.pl` (emgiezet, 18/06), `eskadra.bielik.ai`, `bielik.ai`.

### O „rankingach" (leaderboard)
Dwa różne znaczenia:
1. **Ranking modeli** = raport benchmarkowy Damiana Siwickiego (`ylia.pl/...`, 01/05) — gotowy „ranking" Bielik vs Minitron + porównania z Gemma 4B / Qwen 3.5-4B.
2. **„Tablica wyników" w samym warsztacie** — wzmianka przy Gliwicach (greg, **#51, 19/06**): „power userzy próbowali hackować tablicę wyników". W kanale **NIE ma osobnego linku do repo** tej tablicy — najpewniejsze źródło to fork greg lub LegardT77.
