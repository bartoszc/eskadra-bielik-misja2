# Scenariusz prowadzącego — Warsztaty „Eskadra Bielika – Misja 2"

**Czas trwania:** ~2h (jeden blok)
**Audytorium:** mieszane (część zna chmurę/Pythona, część nie)
**Model warsztatu:** Hybryda — ciężki deploy Bielika (GPU) startuje w tle podczas teorii, uczestnicy samodzielnie robią szybkie i pouczające kroki.
**Repo:** `github.com/avedave/eskadra-bielik-misja2`

---

## ⏱️ Ramowy timing

| Czas | Blok | Slajdy |
|------|------|--------|
| 0:00–0:10 | 1. Wstęp i idea suwerennego AI | 1–3 |
| 0:10–0:17 | 2. Setup środowiska | 5–6, 14–15 |
| 0:17–0:22 | 3. 🚀 Deploy Bielika W TLE | — |
| 0:22–0:47 | 4. Teoria (gdy GPU się buduje) | 7–13 |
| 0:47–0:57 | 5. Embedding + baza wektorowa | — |
| 0:57–1:07 | 6. Test modeli + deploy orchestratora | — |
| 1:07–1:22 | 7. Zasilenie bazy + pierwsze pytanie | — |
| 1:22–1:37 | 8. RAG vs surowy model | 8 |
| 1:37–1:50 | 9. Ćwiczenie kreatywne | — |
| 1:50–2:00 | 10. Podsumowanie + Q&A | 16–17 |

> **Plan B na czas:** timing jest napięty. Przy opóźnieniach tnij w tej kolejności: (1) blok 9 „Ćwiczenie kreatywne" → pokaż jako demo zamiast hands-on; (2) skróć dyskusję w bloku 8; (3) blok 7 — pokaż `/ask` tylko z UI, pomiń `curl`.

---

## 🔑 Zasada przewodnia warsztatu

Cała sztuczka czasowa polega na jednym: **deploy Bielika (GPU L4) trwa kilkanaście minut** (budowa obrazu Ollama + pobranie wag ~5 GB). Dlatego odpalamy go **na samym początku** (blok 3) i zostawiamy pracujący w tle, a czas oczekiwania wypełniamy teorią (blok 4). Gdy wracamy do konsoli (blok 6), Bielik jest już gotowy.

---

## Blok 1 — Wstęp i idea suwerennego AI (0:00–0:10) · slajdy 1–3

**Co mówić:**
- Powitanie, krótkie bio (slajd 2).
- Hasło: *„Suwerenne i wiarygodne AI — od dokumentów firmowych do inteligentnej bazy wiedzy"*.
- Po co to wszystko: chcemy asystenta, który odpowiada **na podstawie naszych dokumentów** (regulaminy, procedury), działa na **polskim modelu Bielik**, a dane nie wychodzą poza nasz projekt w chmurze.
- Zapowiedź: w 2h każdy zbuduje działający system RAG end-to-end i porówna „goły" model z modelem wspartym własną wiedzą.

**Checkpoint:** sala wie, co to RAG w jednym zdaniu? („Model + wyszukiwanie w naszych dokumentach przed odpowiedzią").

---

## Blok 2 — Setup środowiska (0:10–0:17) · slajdy 5–6, 14–15

**Co mówić:** zanim cokolwiek wdrożymy, przygotowujemy projekt Google Cloud i Cloud Shell. Cloud Shell to gotowy terminal w przeglądarce z `gcloud` — nic nie instalujemy lokalnie.

**Komendy (uczestnicy wykonują razem):**

```bash
# Weryfikacja konta i projektu
gcloud auth list
gcloud config get project

# Klon repo i wejście do katalogu
git clone https://github.com/avedave/eskadra-bielik-misja2
cd eskadra-bielik-misja2

# Wczytanie zmiennych środowiskowych
source setup_env.sh

# Włączenie potrzebnych usług
gcloud services enable run.googleapis.com
gcloud services enable cloudbuild.googleapis.com
gcloud services enable artifactregistry.googleapis.com
gcloud services enable bigquery.googleapis.com

# Uprawnienie do wywoływania prywatnych usług Cloud Run
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member=user:$(gcloud config get-value account) \
  --role='roles/run.invoker'
```

**Checkpoint:** „Wszyscy widzą swój `PROJECT_ID` po `echo $PROJECT_ID`?"

**⚠️ Pułapki:**
- **Konta Frictionless** (kartki z loginem/hasłem) — upewnij się PRZED warsztatem, że masz wydrukowany i pocięty PDF z kontami. Każdy uczestnik loguje się w trybie incognito na console.cloud.google.com i **wybiera istniejący projekt z listy**.
- Najczęstszy błąd: zalogowanie się, ale zapomnienie o wybraniu projektu — sprawdź checkpoint po konfiguracji środowiska.
- **Edge case kont:** jeśli uczestnik nie może zalogować się na żadne z przydzielonych kont (błąd „Couldn't sign in" lub „Verify it's you" na każdym kolejnym) — inna przeglądarka/incognito/sieć nie pomoże. Rozwiązanie: uczestnik pracuje w parze z kimś, kto ma działające konto, i próbuje samodzielnie w domu.
- Mylenie **nazwy projektu** z **ID projektu** — to nie zawsze to samo.
- `enable services` potrafi chwilę trwać — można puścić w tle.

---

## Blok 3 — 🚀 Deploy Bielika W TLE (0:17–0:22) · bez slajdu

**Co mówić:** „Teraz odpalamy najcięższy element — model Bielik na karcie GPU. To zajmie kilkanaście minut, więc **uruchamiamy go TERAZ i zostawiamy w tle**, a w międzyczasie omówimy, jak to wszystko działa."

**Komendy:**

```bash
cd llm
./cloud_run.sh
```

**Co się dzieje (warto powiedzieć):** Cloud Build buduje obraz z `ollama/ollama`, wewnątrz pobierane są wagi modelu `SpeakLeash/bielik-4.5b-v3.0-instruct:Q8_0`, a usługa `bielik` startuje na GPU **nvidia-l4** z 32 GB RAM.

**Checkpoint:** „Wszyscy widzą, że deploy ruszył (logi budowania lecą)? Zostawiamy okno i przechodzimy do teorii."

**⚠️ Pułapki:**
- **Quota GPU** — na świeżych kontach L4 może wymagać zwiększenia limitu. Jeśli ktoś dostaje błąd quota → patrz „Plan awaryjny GPU" na końcu dokumentu.
- Nie zamykać terminala z deployem; jeśli ktoś zamknie — ponowny `source ../setup_env.sh` i `./cloud_run.sh`.

---

## Blok 4 — Teoria, gdy GPU się buduje (0:22–0:47) · slajdy 7–13

To główny blok teoretyczny. Tłumacz każdą koncepcję krótko, wiążąc ją z elementem, który zaraz wdrożymy.

**Co mówić (po slajdach):**
- **Slajd 8 — Czym jest RAG:** pokaż diagram przepływu (pytanie → embedding → wyszukanie w bazie wektorowej → kontekst → LLM → odpowiedź). Podkreśl: model nie „wie" o naszym hotelu — my mu *podajemy* pasujące fragmenty.
- **Slajd 9 — Bielik:** polski model, różne wielkości; my używamy `4.5B-v3` w kwantyzacji `Q8_0` (kompromis jakość/rozmiar). Dobre rozumienie polskiego kontekstu.
- **Slajd 10 — EmbeddingGemma:** zamienia tekst na wektory; mały i szybki, idealny do RAG, integruje się z BigQuery.
- **Slajd 11 — Ollama:** silnik do uruchamiania LLM-ów; w repo opakowuje i Bielika, i EmbeddingGemmę w obrazy Dockera.
- **Slajd 12 — BigQuery:** nasza baza wiedzy z **wbudowanym Vector Search** — wyszukiwanie semantyczne SQL-em, bez osobnej bazy wektorowej.
- **Slajd 13 — Cloud Run:** serverless, GPU na żądanie, autoskalowanie, płatność za użycie — tu mieszkają oba modele i orchestrator.

**Checkpoint:** zerknij na terminal — czy deploy Bielika się kończy? Jeśli tak, zapowiedz, że za chwilę go przetestujemy.

---

## Blok 5 — Embedding + baza wektorowa (0:47–0:57) · bez slajdu

**Co mówić:** „Teraz dwa szybkie elementy: model osadzający (CPU, więc buduje się szybko) i tabela w BigQuery na nasze wektory."

**Komendy:**

```bash
# (jeśli ktoś jest jeszcze w katalogu llm/)
cd ..

# Deploy modelu embeddingowego (CPU — szybki)
cd embedding_model
./cloud_run.sh
cd ..

# Inicjalizacja bazy wektorowej w BigQuery
cd vector_store
pip install google-cloud-bigquery
python init_db.py
cd ..
```

**Co się dzieje:** `init_db.py` tworzy dataset `rag_dataset` i tabelę `hotel_rules` ze schematem `id`, `content`, `embedding` (`FLOAT64 REPEATED`) — kolumna wektorowa.

**Checkpoint:** „Wszyscy mają komunikat `Utworzono tabelę: ...hotel_rules` (lub `już istnieje`)?"

**⚠️ Pułapki:**
- Trzeba mieć wczytane zmienne (`source setup_env.sh`) — `init_db.py` bez `PROJECT_ID` od razu o tym przypomni.

---

## Blok 6 — Test modeli + deploy orchestratora (0:57–1:07) · bez slajdu

**Co mówić:** „Bielik powinien być już gotowy — sprawdźmy go bezpośrednio, a potem wdrożymy aplikację spinającą całość."

**Komendy:**

```bash
# Szybki test Bielika (zadaje pytanie o chlor w basenie)
cd llm
./llm_test1.sh
cd ..

# (opcjonalnie) test embeddingu — zwraca wektor liczb
cd embedding_model
./embedding_test1.sh
cd ..

# Deploy aplikacji FastAPI (orchestrator) — sam dociąga URL-e modeli
cd orchestration
./cloud_run.sh
cd ..

# Zapis adresu usługi do zmiennej
export ORCHESTRATION_URL=$(gcloud run services describe orchestration-api \
  --region $REGION --format="value(status.url)")
echo $ORCHESTRATION_URL
```

**Co się dzieje:** `orchestration/cloud_run.sh` sam wykonuje `gcloud run services describe` dla `bielik` i `embedding-gemma`, wstrzykuje ich URL-e jako `LLM_URL`/`EMBEDDING_URL` i wdraża jedyną **publiczną** usługę (`--allow-unauthenticated`).

**Checkpoint:** „`llm_test1.sh` zwrócił sensowną odpowiedź po polsku? `echo $ORCHESTRATION_URL` pokazuje adres `https://...`?"

**⚠️ Pułapki:**
- Jeśli Bielik jeszcze się NIE wdrożył — poczekajcie na niego tutaj; to naturalny punkt synchronizacji sali.
- Orchestrator wymaga, by `bielik` i `embedding-gemma` już istniały (skrypt to sprawdza i krzyczy, jeśli nie).

---

## Blok 7 — Zasilenie bazy + pierwsze pytanie (1:07–1:22) · bez slajdu

**Co mówić:** „Mamy pustą bazę. Wgrajmy do niej regulamin hotelu — aplikacja zamieni każdą regułę na wektor i zapisze w BigQuery. Potem zadamy pierwsze pytanie RAG."

**Komendy:**

```bash
# Zasilenie bazy przykładowymi regułami hotelowymi
curl -X POST "$ORCHESTRATION_URL/ingest" \
     -F "file=@vector_store/hotel_rules.csv"

# Pierwsze pytanie RAG
curl -X POST "$ORCHESTRATION_URL/ask" \
     -H "Content-Type: application/json" \
     -d '{"query": "Jak często powinien być mierzony poziom chloru w basenie?"}'
```

Potem otwórz **Web UI**: wklej `$ORCHESTRATION_URL` w przeglądarkę.

**Co się dzieje przy `/ask`:** pytanie → embedding → `VECTOR_SEARCH` w BigQuery (top 3, COSINE) → zbudowanie promptu z kontekstem → Bielik → odpowiedź + lista użytych fragmentów.

**Checkpoint:** „Odpowiedź na pytanie o chlor mówi o **co 3 godziny**? (To reguła nr 12 z CSV)."

**⚠️ Pułapki:**
- Indeksowanie do Vector Search może chwilę potrwać — dane tekstowe są od razu, ale wyszukiwanie wektorowe może „złapać" z lekkim opóźnieniem.
- CSV musi mieć kolumny `id` i `text` (w bazie zapisywane jako `content`).

---

## Blok 8 — RAG vs surowy model (1:22–1:37) · slajd 8

**Co mówić:** „Najważniejszy moment — po co w ogóle ten cały RAG. W UI mamy dwie odpowiedzi obok siebie: po lewej Bielik wsparty naszą bazą (`/ask`), po prawej **goły** Bielik bez kontekstu (`/ask_direct`)."

**Demo w UI — zadaj pytania, na które odpowiedź jest TYLKO w naszym CSV:**
- „Ile kosztuje parking hotelowy?" → RAG: **40 PLN/dzień** (reguła 5); goły model: zmyśli lub powie, że nie wie.
- „O której godzinie podawane jest śniadanie?" → RAG: **7:00–10:00** (reguła 1).
- „Do której otwarty jest basen?" → RAG: **8:00–22:00** (reguła 3).

**Dyskusja (zaangażuj salę):**
- Dlaczego goły model zmyśla? (Halucynacje — odpowiada „z głowy", bez dostępu do naszych danych.)
- Skąd RAG wie? Pokaż sekcję **użytego kontekstu** w UI — to fragmenty z BigQuery.
- Suwerenność: nasze dokumenty zostają w naszym projekcie, model działa na Cloud Run.

**Checkpoint:** „Widać różnicę? Goły model halucynuje, RAG cytuje regułę."

---

## Blok 9 — Ćwiczenie kreatywne (1:37–1:50) · bez slajdu

**Co mówić:** „Teraz najlepsza część — pokażę, jak łatwo zmienić zachowanie asystenta i czym go karmimy."

**Ćwiczenie A — zmiana persony (edycja promptu):**
Otwórz `orchestration/main.py`, znajdź prompt w endpoincie `/ask` (zmienna `prompt`, ok. linii 137) i każ Bielikowi odpowiadać np. jak pirat albo ekspert IT. Po zmianie:

```bash
cd orchestration
./cloud_run.sh
cd ..
```

Zadaj to samo pytanie i porównaj ton odpowiedzi.

**Ćwiczenie B — własne dokumenty:**
Stwórz własny mały CSV (kolumny `id,text`) z dowolną „wiedzą firmową" i zasil bazę:

```bash
curl -X POST "$ORCHESTRATION_URL/ingest" -F "file=@moj_plik.csv"
```

Zapytaj o coś z własnych danych.

**Checkpoint:** „Komuś udało się zmienić personę / wgrać własne dane?"

**⚠️ Pułapki:**
- Po każdej zmianie `main.py` trzeba redeployować orchestrator (`./cloud_run.sh`).
- To blok do cięcia jako pierwszy, jeśli goni czas — wtedy pokaż jako demo na swoim ekranie.

---

## Blok 10 — Podsumowanie + Q&A (1:50–2:00) · slajdy 16–17

**Co mówić:**
- Co zbudowaliśmy: suwerenny RAG end-to-end na polskim modelu — embedding, baza wektorowa, LLM, API i UI.
- Kluczowy wniosek: RAG zamienia ogólny model w eksperta od *naszych* dokumentów, bez dotrenowywania i bez wynoszenia danych.
- **eskadra.bielik.ai** (slajd 16) — zachęta do dołączenia do społeczności (slajd 17).
- Materiały: repo + README jako instrukcja krok po kroku do powtórzenia w domu.

**Q&A.**

**⚠️ Sprzątanie po warsztacie (wspomnij!):** usługi GPU kosztują — przypomnij, by po warsztacie skasować usługi Cloud Run (`bielik`, `embedding-gemma`, `orchestration-api`) i dataset BigQuery, żeby nie palić kredytów.

---

## 🔧 Plan awaryjny GPU (quota / brak L4)

Jeśli u części sali deploy `bielik` blokuje quota GPU:
- **Opcja 1 (zalecana):** prowadzący ma **wcześniej wdrożoną wspólną usługę `bielik`** w swoim projekcie z `--allow-unauthenticated` lub nadanym `run.invoker` dla uczestników; podajesz jej URL jako `LLM_URL` przy deployu orchestratora.
- **Opcja 2:** uczestnicy z problemem pracują w parach z osobą, której deploy się udał.
- **Opcja 3:** poproś o zwiększenie quota L4 (`IAM & Admin → Quotas`) — ale to bywa wolne, więc lepiej ograć Opcją 1.

---

## 📋 Checklista prowadzącego (przed warsztatem)

- [ ] PDF z kontami Frictionless wydrukowany i pocięty (dokładna liczba uczestników = z Lumy).
- [ ] Sprawdzona dostępność quota GPU L4 w regionie `europe-west1` (lub gotowa wspólna usługa `bielik`).
- [ ] Przetestowany cały przepływ na czysto (od `git clone` do `/ask`).
- [ ] Slajdy uzupełnione (patrz `plan-slajdow.md`), w tym diagram RAG (8a) i architektury (8b).
- [ ] Link do repo gotowy do wyświetlenia; PDF z kontami rozdany przy wejściu.
