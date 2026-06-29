# Plan slajdów — Warsztaty „Eskadra Bielika – Misja 2"

Dokument slajd-po-slajdzie do przeniesienia treści do szablonu `Eskadra Bielika - Misja 2 - Szablon Slajdów.pptx`.

**Legenda akcji:**
- **KEEP** — zostaw jak jest (ew. drobna korekta).
- **FILL** — uzupełnij placeholder konkretną treścią.
- **NEW** — dodaj nowy slajd (treść poniżej).
- **CUT** — usuń z finalnej talii (to puste wzorce układów).

**Talia po zmianach (kolejność prezentacyjna):** 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 → **8a (RAG)** → **8b (Architektura)** → 9 → 10 → 11 → 12 → 13 → 14 → 15 → 16 → 17 → 18 (Agenda — najlepiej przesuń na początek, patrz uwaga) → [31–34 jako załącznik].

> **Uwaga o agendzie:** slajd 18 (Agenda) logicznie pasuje na **początek** (po slajdzie 4 „Warsztaty"). Sugeruję przenieść go tam podczas składania talii.

---

## Slajd 1 — Tytuł · **FILL**

```
Eskadra Bielika
Misja Druga

Lublin, [DATA WARSZTATU]
```

Zamień placeholder „Miasto, data" na `Lublin, [data]`.

---

## Slajd 2 — Prowadzący · **KEEP** (placeholder do uzupełnienia przez Ciebie)

Uzupełnij swoimi danymi:
- Imię i nazwisko
- Krótkie info o trenerze (wykształcenie, doświadczenie zawodowe)
- Aktywność społeczna
- Informacja o certyfikacji Eskadry

---

## Slajd 3 — Hasło przewodnie · **KEEP**

```
Suwerenne i wiarygodne AI
Od dokumentów firmowych do inteligentnej bazy wiedzy
w oparciu o model Bielik i Google Cloud
```

---

## Slajd 4 — „Warsztaty" (przerywnik sekcji) · **KEEP**

---

## Slajd 5 — Repo i autor · **KEEP**

```
github.com/bartoszc/eskadra-bielik-misja2
Instrukcja krok po kroku

Autor: Dawid Ostrowski
Senior Program Manager, Developer Relations, Google
linkedin.com/in/avedave
```

---

## Slajd 6 — Kupony Google Cloud Credits · **FILL**

Wstaw realny link do kuponów zamiast `<trygcp link>`:

```
Kupony Google Cloud Credits
→ [WSTAW LINK trygcp]
```

> Sprawdź ważność linku przed warsztatem.

---

## Slajd 7 — „Kluczowe koncepcje" (przerywnik sekcji) · **KEEP**

---

## Slajd 8 — Co budujemy · **KEEP**

```
Praktyczne warsztaty
• Budowa suwerennego systemu RAG w oparciu o model Bielik i Google Cloud
• Stworzenie inteligentnej bazy wiedzy na podstawie wewnętrznych dokumentów firmowych
```

---

## Slajd 8a — „Jak działa RAG?" · **NEW** ⭐ (kluczowy — tego brakuje w talii)

Nagłówek: **Jak działa RAG? (Retrieval-Augmented Generation)**

Diagram przepływu (odtwórz jako boksy + strzałki w PowerPoincie):

```
   [Pytanie użytkownika]
            │
            ▼
   ┌──────────────────┐     zamiana tekstu na wektor
   │  EmbeddingGemma  │ ───────────────────────────────►
   └──────────────────┘
            │  wektor pytania
            ▼
   ┌──────────────────────────┐   znajdź 3 najbardziej
   │  BigQuery Vector Search  │   podobne fragmenty (COSINE)
   └──────────────────────────┘
            │  pasujący kontekst (fragmenty dokumentów)
            ▼
   ┌──────────────────┐   prompt = kontekst + pytanie
   │   Bielik (LLM)   │
   └──────────────────┘
            │
            ▼
   [Odpowiedź oparta o NASZE dokumenty]
```

Pointa na slajd (1 zdanie): *Model nie „wie" o naszym hotelu — my mu podajemy pasujące fragmenty z bazy, zanim odpowie. Dzięki temu odpowiada faktami, a nie zgaduje.*

---

## Slajd 8b — Architektura rozwiązania · **NEW** ⭐

Nagłówek: **Architektura — wszystko serverless na Google Cloud**

Diagram komponentów (boksy):

```
                      ┌─────────────────────────────┐
   Przeglądarka ───►  │  orchestration-api (FastAPI)│  ◄── publiczna usługa
   (Web UI)           │  Cloud Run                  │      (/, /ask, /ask_direct,
                      └─────────────────────────────┘       /ingest)
                          │              │
              token       │              │  token tożsamości
           tożsamości     │              │  (usługi prywatne)
                          ▼              ▼
            ┌──────────────────┐   ┌──────────────────┐
            │  embedding-gemma │   │      bielik      │
            │  Cloud Run (CPU) │   │ Cloud Run (GPU L4)│
            │  EmbeddingGemma  │   │ Bielik 4.5B Q8_0 │
            └──────────────────┘   └──────────────────┘
                          │
                          ▼
            ┌──────────────────────────────┐
            │  BigQuery: rag_dataset       │
            │  tabela hotel_rules          │
            │  (id, content, embedding[])  │
            │  + VECTOR_SEARCH             │
            └──────────────────────────────┘
```

Podpisy/uwagi na slajd:
- 4 niezależnie wdrażane elementy, region `europe-west1`.
- Modele są **prywatne**; orchestrator woła je **tokenem tożsamości** Google (suwerenność i bezpieczeństwo).
- Baza wektorowa to **natywny** BigQuery Vector Search — bez osobnej bazy wektorowej.

---

## Slajd 9 — Bielik · **KEEP**

```
Bielik — suwerenny polski model językowy
• Dostępne różne wersje i wielkości:
  Bielik-1.5B-v3 · Bielik-4.5B-v3 · Bielik-11B-v2.6 · Bielik Minitron 7B
• Architektura Transformers
• Różne modele bazowe (Mistral, Qwen, Nemotron)
• Na warsztacie: Bielik-4.5B-v3 w kwantyzacji Q8_0
```

---

## Slajd 10 — EmbeddingGemma · **KEEP** (popraw literówkę)

```
EmbeddingGemma — model do tworzenia wektorów
• Embeddingi o stałej długości, głęboki kontekst semantyczny
• Idealny dla RAG
• Mniejszy i szybszy niż pełny LLM        ← POPRAWKA: było „M niejszy i szybsze"
• Łatwa integracja np. z BigQuery
• Wielojęzyczność
```

---

## Slajd 11 — Ollama · **KEEP**

```
Ollama — uruchamiaj modele LLM lokalnie
• Repozytorium LLM-ów
• Zarządzanie modelami
• U nas: pakuje Bielika i EmbeddingGemmę w obrazy Dockera na Cloud Run
```

---

## Slajd 12 — BigQuery · **KEEP**

```
BigQuery — hurtownia danych w chmurze
• Automatyczne skalowanie pamięci i mocy obliczeniowej
• Natywny typ danych do przechowywania wektorów / embeddingów
• Wbudowany Vector Search — wyszukiwanie semantyczne
• Generowanie embeddingów i wywoływanie LLM-ów bezpośrednio w SQL
• Ekosystem API: SQL, Python (BigFrames) i in.
```

---

## Slajd 13 — Google Cloud Run · **KEEP**

```
Google Cloud Run — w pełni zarządzana platforma serverless
• Instancje GPU (L4, RTX PRO 6000)
• 2 mln darmowych requestów miesięcznie
• Szybkie autoskalowanie
• Każdy język, każdy framework
```

---

## Slajd 14 — Google Cloud Shell · **KEEP**

---

## Slajd 15 — Cloud Shell Editor + Code Assist · **KEEP**

---

## Slajd 16 — eskadra.bielik.ai · **KEEP**

```
https://eskadra.bielik.ai/
```

---

## Slajd 17 — „Dołącz do…" · **KEEP**

Zaproszenie do społeczności Eskadry Bielika (uzupełnij call-to-action / link/QR jeśli masz).

---

## Slajd 18 — Agenda · **FILL** (i przesuń na początek, po slajdzie 4)

```
Agenda

01  Wstęp i idea suwerennego AI
02  Setup środowiska (Cloud Shell, kupony, repo)
03  Deploy modelu Bielik (start w tle)
04  Kluczowe koncepcje: RAG, Bielik, Gemma, BigQuery, Cloud Run
05  Model embeddingowy + baza wektorowa w BigQuery
06  Orchestrator (API + Web UI)
07  Zasilenie bazy i pierwsze pytania RAG
08  RAG vs surowy model + eksperymenty
```

---

## Slajdy 19–30 — wzorce układów (lorem ipsum) · **CUT**

Usuń z finalnej talii prezentacyjnej. To wzorce układów z szablonu (2 kolumny, opcje z ikonami, features, timeline, miejsca na zdjęcia). Zachowaj plik źródłowy szablonu, gdyby któryś układ się przydał — ale w talii warsztatowej nie pokazujemy lorem ipsum.

Gdybyś chciał wykorzystać któryś układ zamiast usuwać — najlepiej pasują:
- układ „dwie/trzy opcje z ikonami" (slajdy 21–22) → można przerobić na porównanie **RAG vs. surowy model**;
- układ „cztery features" (slajd 24) → można przerobić na **4 komponenty architektury** (alternatywa dla slajdu 8b).

---

## Slajdy 31–34 — Zasoby Eskadry · **KEEP jako załącznik**

Zostaw na końcu, poza główną narracją:
- 31 — „Zasoby Eskadry Bielika do wykorzystania"
- 32–33 — zestawy teł do prezentacji
- 34 — biblioteka ikon Phosphor (`phosphoricons.com`)

Przydatne przy składaniu/brandingu talii, nie pokazujemy na warsztacie.

---

## ✅ Podsumowanie zmian

| Akcja | Slajdy |
|-------|--------|
| FILL | 1, 6, 18 |
| NEW | 8a (diagram RAG), 8b (architektura) |
| KEEP + drobna korekta | 10 (literówka „Mniejszy") |
| KEEP | 2–5, 7–9, 11–17, 31–34 |
| CUT | 19–30 |
