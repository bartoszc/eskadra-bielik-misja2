# Warsztaty „Eskadra Bielika – Misja 2" — prezentacja + scenariusz prowadzącego

**Data:** 2026-06-28
**Kontekst:** Warsztaty w Lublinie (najbliższy wtorek) w ramach projektu Eskadra Bielika. Repozytorium to gotowy codelab budujący suwerenny system RAG (Bielik + EmbeddingGemma + BigQuery Vector Search + FastAPI) na Google Cloud.

## Cel

Dostarczyć dwa dokumenty markdown, które prowadzący wykorzysta do poprowadzenia 2-godzinnych warsztatów:

1. **Scenariusz prowadzącego** — ramowy timing, narracja, komendy, checkpointy, pułapki.
2. **Plan slajdów** — treść slajd-po-slajdzie (KEEP/FILL/NEW/CUT) do przeniesienia do szablonu `.pptx`.

## Założenia (ustalone na brainstormingu)

- **Czas:** ~2h, jeden blok.
- **Audytorium:** mieszane (część zna chmurę/Pythona, część nie) — trzeba tłumaczyć podstawy RAG i `gcloud`.
- **Format dostarczenia:** dokumenty markdown w repo (prowadzący sam przenosi treść do `.pptx`).
- **Model warsztatu:** **Opcja B — Hybryda.** Ciężki deploy Bielika (GPU L4, budowa obrazu Ollama + pobranie wag ~5 GB) startuje na początku i pracuje w tle, gdy omawiana jest teoria. Uczestnicy samodzielnie wykonują części szybkie i pouczające: `setup_env.sh`, deploy embeddingu (CPU), `init_db.py`, deploy orchestratora, `/ingest`, `/ask` vs `/ask_direct`, modyfikacja promptu.

## Deliverable 1 — Scenariusz prowadzącego

Plik: `docs/warsztaty/scenariusz-prowadzacego.md`

Struktura: tabela ramowa + sekcja na każdy blok. Każdy blok zawiera:
- **Co mówić** — skrót narracji (nie słowo w słowo).
- **Komendy** — dokładne polecenia do pokazania/wykonania.
- **Checkpoint** — pytanie kontrolne do sali („wszyscy mają zielony status usługi X?").
- **Pułapki** — typowe błędy (re-source `setup_env.sh` w nowym terminalu, quota GPU, kolejność wdrożeń).

Ramowy timing (120 min):

| Czas | Blok | Treść | Slajdy |
|------|------|-------|--------|
| 0:00–0:10 | Wstęp | Powitanie, bio, idea suwerennego AI | 1–3 |
| 0:10–0:17 | Setup | Cloud Shell, kupony, klon repo, `source setup_env.sh`, enable services + IAM | 5–6, 14–15 |
| 0:17–0:22 | 🚀 Deploy Bielika W TLE | `llm/cloud_run.sh` — zostawiamy działający | — |
| 0:22–0:47 | Teoria (gdy GPU się buduje) | RAG, Bielik, EmbeddingGemma, Ollama, BigQuery Vector Search, Cloud Run | 7–13 |
| 0:47–0:57 | Embedding + baza | `embedding_model/cloud_run.sh` + `init_db.py` | — |
| 0:57–1:07 | Test modeli + orchestrator | `llm_test1.sh`, deploy `orchestration-api` | — |
| 1:07–1:22 | Zasilenie + 1. pytanie | `/ingest` hotel_rules.csv, pierwsze `/ask`, Web UI | — |
| 1:22–1:37 | RAG vs surowy model | `/ask` ↔ `/ask_direct`, dyskusja | 8 |
| 1:37–1:50 | Ćwiczenie kreatywne | Modyfikacja promptu (pirat/ekspert IT) + własny CSV | — |
| 1:50–2:00 | Podsumowanie + Q&A | eskadra.bielik.ai, zasoby, społeczność | 16–17 |

Buforowanie: timing jest napięty; przy opóźnieniach ciąć blok „Ćwiczenie kreatywne" (1:37–1:50) jako pierwszy.

## Deliverable 2 — Plan slajdów

Plik: `docs/warsztaty/plan-slajdow.md`

Dokument slajd-po-slajdzie z oznaczeniem akcji i gotową treścią:

- **1** FILL — tytuł → „Lublin, [data]".
- **2** KEEP — bio prowadzącego (placeholder do uzupełnienia).
- **3–8** KEEP — drobne dopieszczenie.
- **8a NEW** — „Jak działa RAG?" — diagram przepływu: pytanie → embedding → BigQuery Vector Search (top 3) → kontekst + prompt → Bielik → odpowiedź. Diagram opisany jako ASCII + treść boxów.
- **8b NEW** — Architektura rozwiązania — 4 komponenty na Cloud Run (Bielik GPU, EmbeddingGemma CPU, orchestrator FastAPI) + BigQuery; przepływ między nimi i uwierzytelnianie tokenem tożsamości.
- **9–15** KEEP — koncepcje; korekty literówek (np. „M niejszy" → „Mniejszy" na slajdzie 10).
- **16–17** KEEP.
- **18** FILL — Agenda → 8 realnych punktów zgodnych ze scenariuszem.
- **19–30** CUT — szablonowe układy lorem ipsum (z notatką, że to wzorce układów do ewentualnego użycia).
- **31–34** KEEP jako załącznik — zasoby Eskadry (tła, ikony Phosphor).

Diagramy (8a, 8b) opisane w markdown jako ASCII/struktura + dokładna treść boxów, aby łatwo odtworzyć w PowerPoincie.

## Poza zakresem (YAGNI)

- Bezpośrednia edycja pliku `.pptx` (ryzyko zepsucia XML/grafiki).
- Generowanie obrazków/diagramów jako plików graficznych.
- Zmiany w kodzie aplikacji RAG.

## Kryteria sukcesu

- Prowadzący może poprowadzić warsztat z dokumentu scenariusza bez zaglądania do kodu w trakcie.
- Plan slajdów daje gotową treść do wklejenia; brak lorem ipsum w finalnej talii.
- Timing mieści się w 2h z jasnymi punktami cięcia przy opóźnieniach.
- Treść spójna z faktycznym kodem repo (nazwy usług, komendy, endpointy, model `SpeakLeash/bielik-4.5b-v3.0-instruct:Q8_0`).
