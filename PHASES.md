# Seminar programme — house.kg

Two phases. **Phase 1** is a guided walkthrough we do together on one clean slice
(Bishkek apartments for sale) — it fixes the method. **Phase 2** is a menu of research
topics (`T1…Tn`): pick one, take it deep on your own branch.

Ground rules for the whole seminar:

- **Compute by hand, then verify with a library.** Write the mean / std / distance in
  NumPy, then check against `pandas` / `scikit-learn`.
- **Interpret in words.** Every plot and every number needs a one-sentence conclusion:
  *what does this say about the data?*
- Tools you already have: histograms & KDE, mean / mode / spread (std, range),
  outliers, normalization (a rate, e.g. price per m²), **z-standardization**, an object
  as a **point**, Euclidean and cosine distance, nearest neighbours.
- Not yet introduced (avoid, or flag clearly as a "spoiler"): percentiles, correlation,
  logarithms. The median appears as a *careful spoiler* — a robust centre we will
  formalise later.

---

## Phase 1 — Bishkek apartments for sale (guided)

One slice only: `deal == "sale"`, `type == "apartment"`, `city == "Бишкек"`. No rent,
no other cities. This is the worked reference in `seminar_1.ipynb`.

**Step 0 — Load.** Download `listings`, look at shape, columns, one row. Understand that
a row is an *ad*, not an apartment.

**Step 1 — Focus.** Build the Bishkek · sale · apartment slice. How many ads?

**Step 2 — Clean.**
- Inspect field coverage; drop the many sparse columns irrelevant to an apartment;
  keep a compact set (price, area, rooms, views, condition, series, floor, coords).
- Missing `rooms_n` is **not random**: show that those ads are atypical (very large
  areas, "free layout", "6+ rooms"). Drop them honestly — but know what you drop.

**Step 3 — Look at the shape first.**
- Histograms + KDE of price, area, price-per-m². Describe the shape (right-skewed,
  heavy tail).
- *Spoiler (just for fun):* the same histograms on `log10(price)` and `log10(area)` —
  the tail straightens into a symmetric hump.
- Count of rooms; scatter `area × price` coloured by rooms — the cloud is glued-together
  groups.

**Step 4 — Centre, spread, and an outlier hunt.**
- Compute the mean price — it looks too high. Find the culprit (`nlargest`): a single
  data-entry error in the hundreds of millions.
- Drop that **one** row and recompute: the mean moves a lot and the std collapses by
  ~100×, while the **median** (spoiler: a robust centre) barely moves.
- Mean vs mode vs median, and why the market bunches on round numbers.
- The range `max − min` is useless while garbage remains; set sane bounds **by hand**
  (not by any percentile), and recount the clean slice.

**Step 5 — Normalization (two flavours).**
- **Price per m²** — dividing by a meaningful quantity. Show *visually* (scatter) that
  the size effect disappears: `area × price` climbs, `area × price-per-m²` is flat. Check
  by groups: mean price rises steeply with rooms, mean price-per-m² stays ~flat.
- **z-standardization** — subtract the mean, divide by the std. Do it by hand, verify
  with `StandardScaler` (mean 0, std 1). Say in words how the two normalizations differ.

**Step 6 — Object as a point, and kNN.**
- Pick features (price, area, rooms, views) and one anchor apartment.
- Find its 5 nearest by **raw** Euclidean distance — notice they only match on *price*;
  the large-scale feature dominates.
- Standardize and find the 5 nearest again — now they match on *everything*. This is
  the point of standardization, lived by hand. (Library check: `NearestNeighbors`.)

**Phase 1 deliverable:** run the walkthrough, and for each step add your own one-line
interpretation. You should be able to explain *why raw Euclidean distance failed and
standardization fixed it*.

---

## Phase 2 — Choose a topic (`T1…Tn`)

Pick one direction and take it deep, reusing the Phase 1 toolkit (distributions →
robust centre → normalization → centroids → kNN / comparison). Each topic has a built-in
trap or surprise — find it, quantify it, explain it. Deliver on a branch (see below).

**T1 — One-room segment.** The tightest, most homogeneous sub-market. Describe its
distributions and spread; is price-per-m² more stable here than across all apartments?
Build a "find similar one-rooms" kNN.

**T2 — Large / premium apartments.** 4–5 rooms or the top price tier. This segment is
*heterogeneous* — show that its spread is much larger than the one-room segment. What
makes "premium" hard to summarise with a single number?

**T3 — The rent market.** Switch to `deal == "rent"`, `price_period == "month"`,
Bishkek. Separate populations again: monthly vs daily. Compare the rent distribution to
sale; note that rent-per-m² *falls* with size (economies of scale). Relate the two
markets (how many monthly rents ≈ one sale price?).

**T4 — The renovation premium.** Group by `condition`. Put a number on how much
"евроремонт" adds over "под самоотделку" in price-per-m². Which centroid sits where?

**T5 — The building-series paradox.** Group by `building_series`. Old Soviet series
(хрущевка, 104/105) can be *more* expensive per m² than "элитка". That breaks intuition —
find the hidden variable that explains it (hint: where in the city are they?).

**T6 — Views & popularity.** Treat `views` as its own heavy-tailed feature. Is
popularity linked to price or size? How concentrated is attention (how much of all views
sits in a handful of ads)?

**T7 — Cities: Bishkek vs Osh.** Compare *meaningfully* — **not** "Bishkek has N ads,
Osh has M". Compare price-per-m² distributions and standardized centroids. Osh is a
small sample (~40 rows): how much can you trust its centre? Add Issyk-Kul (resort market)
if you like.

**T8 — Duplicate ads across agencies.** The same apartment is posted by several
agencies. Estimate how many "apartments for sale" are really distinct vs how many are
*ads*. Use near-zero distance in feature space (and shared address) to flag duplicates.

**T9 — A "good deal?" detector.** For a target apartment, find its nearest neighbours in
standardized space (comparable area/rooms/condition) and compare its price to theirs.
Which listings look under- or over-priced relative to their peers?

**T10 — Geography (advanced).** Use `latitude` / `longitude`. Are there price-per-m²
gradients across the city? Cluster listings by location and compare centroids.

You may propose your own `Tn` — clear it first.

---

## How to submit

We all start from `seminar_1.ipynb` on `main`. Do your work on your **own branch** and
push it with a short write-up.

```bash
git checkout -b t7-cities-osh-<yourname>      # e.g. topic + name
# ... work in a copy of the notebook / your own notebook ...
git add .
git commit -m "T7: Bishkek vs Osh price-per-m2 comparison"
git push -u origin t7-cities-osh-<yourname>
```

Each branch must include:

- your notebook (with plots that have a **grid** and readable axes);
- a short **`FINDINGS.md`** in your branch: what you did, what you found, the numbers,
  and — most important — *what it means* and *what surprised you*;
- honest notes on what you dropped/cleaned and why (especially small-sample or
  duplicate caveats).

Grading follows the course model: the analysis on your slice matters, but the
**interpretation** matters more than the code.
