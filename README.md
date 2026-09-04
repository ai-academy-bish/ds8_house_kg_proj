# house.kg — Statistics & Linear Algebra Seminar

A hands-on seminar project. We take **one real dataset** — real-estate classifieds
from [house.kg](https://www.house.kg) (Kyrgyzstan) — and apply everything from the
course so far: distributions and histograms, central tendency and spread, outliers,
normalization, z-standardization, treating an object as a point, and nearest-neighbour
search. We work in a shared notebook and everyone pushes their own findings to a branch.

The full task programme is in **[PHASES.md](PHASES.md)**.

## The dataset

We use **[`aiacademy-kg/house_kg_full_dataset`](https://huggingface.co/datasets/aiacademy-kg/house_kg_full_dataset)**
(public). It is a full crawl of **25,473 listings** across Kyrgyzstan and is organised
as several linked tables:

| Table | What it is |
|---|---|
| `listings` | **the one we use** — one row per ad: deal, type, city, price, area, rooms, views, condition, coordinates, … |
| `companies` / `complexes` | agencies and residential complexes, with ratings |
| `reviews` / `users` | reviews and their authors |
| `photos` | listing photos (**we do NOT use images in this seminar**) |

`listings` has ~72 fields; we keep a compact subset (price, area, rooms, views,
condition, building series, floor, coordinates). Values stay in the original Russian
(e.g. `condition == "евроремонт"`); field names are English.

> Two facts to know up front: **~92% of listings are in Bishkek**, and **sale and rent
> prices are not comparable** (a sale is a total, a rent is a monthly/daily rate — always
> filter on `deal` / `price_period`). Phase 1 stays inside a single clean slice:
> **Bishkek · sale · apartments**.

## Environment setup (uv)

```bash
# 1. install uv once (https://docs.astral.sh/uv/)
curl -LsSf https://astral.sh/uv/install.sh | sh

# 2. create the environment and install dependencies
uv venv                       # creates .venv with a compatible Python
uv pip install -e .           # core deps from pyproject.toml
uv pip install -e ".[notebook]"   # + Jupyter/ipykernel to run the notebook locally

# 3. activate
source .venv/bin/activate     # Windows: .venv\Scripts\activate
```

On **Google Colab** you don't need uv — just run `!pip install -q datasets` in the
first cell.

## Hugging Face authentication

The dataset is public, so downloads work without a token — but unauthenticated
requests are rate-limited. Authenticate once to avoid throttling:

```bash
uv run huggingface-cli login        # paste a token from huggingface.co/settings/tokens
# or, non-interactive:
export HF_TOKEN=hf_xxx               # Colab: userdata / os.environ["HF_TOKEN"]
```

## Working with the data

```python
import pandas as pd
from huggingface_hub import hf_hub_download

path = hf_hub_download("aiacademy-kg/house_kg_full_dataset",
                       "data/listings.parquet", repo_type="dataset")
df = pd.read_parquet(path)

# Phase 1 working slice: Bishkek apartments for sale
work = df[(df.deal == "sale") & (df.type == "apartment") & (df.city == "Бишкек")].copy()
```

### What you can do with it (examples)

- see the **shape** of prices/areas (right-skewed, heavy-tailed) and how a single
  bad row wrecks the mean and the standard deviation;
- **normalize**: price per m² (removes the size effect) vs z-standardization;
- treat a listing as a **point** and find the *k* most similar apartments — and see why
  the raw Euclidean distance is dominated by price until you standardize;
- (Phase 2) compare **segments** (one-room, rentals, renovation, building series),
  compare **cities** (Bishkek vs Osh) *meaningfully*, or hunt **duplicate ads**.

## Workflow

We all start from `seminar_1.ipynb`. For your own work, **create a branch and push it**
with a short note of what you did and what you found — see [PHASES.md](PHASES.md#how-to-submit).

## Layout

```
seminar_1.ipynb   shared starter notebook (Phase 1 walkthrough)
pyproject.toml    dependencies (managed with uv)
README.md         this file
PHASES.md         the task programme (Phase 1 + Phase 2 topics)
```
