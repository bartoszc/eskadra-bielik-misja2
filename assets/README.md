# Materiały e-learningowe

Dodatkowe materiały do samodzielnej nauki po warsztatach. Możesz do nich wrócić w dowolnym momencie.

---

## Interaktywne diagramy architektury (`diagrams/`)

Cztery diagramy HTML wizualizujące każdy etap systemu RAG. Działają offline — wystarczy pobrać repo i otworzyć plik `.html` w przeglądarce.

| Plik | Co pokazuje |
|---|---|
| `architektura_interaktywna.html` | Pełny widok systemu — komponenty i przepływ danych |
| `architektura_interaktywna_ingestion.html` | Pipeline ingestion — ładowanie danych do BigQuery |
| `architektura_interaktywna_rag.html` | Pipeline RAG — zapytanie → embedding → BQ → LLM → odpowiedź |
| `architektura_interaktywna_embeddings.html` | Wprowadzenie do embeddingów — od 1D do 768D + cosine similarity |

### Jak uruchomić

**Opcja A — bezpośrednio w przeglądarce (najprostsza):**
```bash
git clone https://github.com/bartoszc/eskadra-bielik-misja2.git
# Otwórz dowolny plik .html z katalogu assets/diagrams/ w przeglądarce
```

**Opcja B — lokalny serwer HTTP (jeśli przeglądarka blokuje pliki lokalne):**
```bash
cd assets/diagrams
bash serve.sh
# Otwórz: http://localhost:8080/architektura_interaktywna.html
```

### Materiały wideo

Nagrania wideo od autora diagramów:
- **Pipeline ingestion** — https://youtu.be/D0qCltR8UJQ
- **Pipeline RAG** — https://youtu.be/D7s8duHl7sQ

---

## Notebook Google Colab (`notebooks/`)

Pełny warsztat w notebooku Jupyter — ten sam materiał co na zajęciach, ale bez potrzeby konta GCP. Działa na darmowym GPU T4 w Google Colab lub Kaggle.

| Element | Wariant GCP (warsztat) | Wariant Colab (notebook) |
|---|---|---|
| LLM / embedding | Cloud Run + Ollama | Ollama lokalnie |
| Vector store | BigQuery VECTOR_SEARCH | ChromaDB lokalnie |
| Checkpointy / certyfikat | ✅ | ✅ (ten sam format) |
| Czas wykonania | ~180 min | ~36 min |

### Jak uruchomić

Kliknij przycisk w notebooku:

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/bartoszc/eskadra-bielik-misja2/blob/main/assets/notebooks/warsztat_colab.ipynb)

> ⚠️ Pamiętaj o włączeniu GPU: `Środowisko wykonawcze → Zmień typ środowiska → GPU (T4)`

---

*Diagramy i notebook opracowane przez [LegardT77](https://github.com/Legard777). Zaadaptowane dla tego repo za zgodą na licencji Apache 2.0.*
