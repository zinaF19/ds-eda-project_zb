# King County Housing – EDA Capstone

## 1. Project Overview
Brief description of the task, the client, and the goal.



## 2. Data Source & Exploration
- Where the data came from (eda.king_county_house_details, eda.king_county_house_sales)
- Row counts of each raw table
- Null-value check per table/column
- How the tables relate (house_details.id = house_sales.house_id)
- Why some houses appear more than once in house_sales (multiple sale events)
- How you joined them (DISTINCT ON, keeping the most recent sale per house)
- Final row count after joining, and why it differs from the raw table counts

## Data Source & Exploration

The data comes from two tables in the King County housing database:

- `eda.king_county_house_details` — one row per unique house
- `eda.king_county_house_sales` — one row per sale event

### Row counts

| Table | Rows |
|---|---|
| `house_details` | 21,420 (unique houses) |
| `house_sales` | 21,597 (sale events) |

### Nulls in `house_details`

| Column | Nulls |
|---|---|
| `waterfront` | 2,360 |
| `view` | 63 |
| `sqft_basement` | 451 |
| `yr_renovated` | 3,811 |
| all other columns | 0 |

No nulls were found in `house_sales` (`house_id`, `date`, `price` all fully populated).

### Explicit 0 vs. NULL

Several columns contain both a confirmed "0" value and a missing (NULL) value —
these mean different things (a confirmed "no" vs. "unknown"), so they were
checked separately:

| Column | Explicit 0 | NULL | Real value |
|---|---|---|---|
| `waterfront` | 18,914 | 2,360 | 146 |
| `view` | 19,253 | 63 | 2,104 |
| `sqft_basement` | 12,717 | 451 | 8,252 |
| `yr_renovated` | 16,869 | 3,811 | 740 |

**Assumption:** missing values in these four columns are treated as equivalent
to 0 (e.g. missing `waterfront` → assumed not waterfront). This is a
simplifying assumption, not a confirmed fact.

### Referential integrity

| Check | Result |
|---|---|
| Houses in `house_details` with no matching sale | 0 |
| Sales in `house_sales` with no matching house | 0 |

Every house has at least one sale, and every sale belongs to a real house —
no rows are lost or orphaned when joining the two tables.

### Why the row counts differ: repeat sales

`house_sales` has more rows than `house_details` because some houses were
sold more than once:

- 175 houses sold exactly 2 times
- 1 house sold 3 times
- Total: 176 houses with repeat sales, producing 177 extra rows
  (175 × 1 + 1 × 2 = 177)

21,597 − 177 = 21,420 → matches `house_details` exactly.

### Join strategy

To get one row per house, the sales table was deduplicated using
`DISTINCT ON (house_id)`, keeping the most recent sale
(`ORDER BY house_id, date DESC`) before joining to `house_details`. This
produces a final dataset of 21,420 rows — matching `house_details` exactly.

To avoid losing sale history for the 176 repeat-sold houses, the final
dataset was enriched with three extra columns:

- `num_sales` — how many times the house sold
- `sale_dates` — list of all sale dates for that house
- `sale_prices` — list of all sale prices for that house

This keeps one row per house (so aggregate statistics aren't biased by
repeat-sold houses) while preserving full sale history for houses that sold
more than once.

## 3. Data Dictionary

| Column | Description |
|---|---|
| `id` | Unique identifier for each house |
| `bedrooms` | Number of bedrooms |
| `bathrooms` | Number of bathrooms (fractional values represent partial bathrooms, e.g. 0.5 = a half bath with no shower/tub) |
| `sqft_living` | Interior living space, in square feet |
| `sqft_lot` | Total lot size, in square feet |
| `floors` | Number of floors (levels) |
| `waterfront` | Whether the property has a waterfront view (1 = yes, 0 = no) |
| `view` | Quality rating of the property's view, scale 0–4 |
| `condition` | Overall maintenance/condition rating, scale 1–5 (how well-kept the house currently is) |
| `grade` | Construction and design quality rating, scale 1–13, per King County Assessor standards (quality of materials/build at time of construction, distinct from `condition`) |
| `sqft_above` | Square footage of the house above ground level (excludes basement) |
| `sqft_basement` | Square footage of the basement, if present |
| `yr_built` | Year the house was built |
| `yr_renovated` | Year of last renovation (0 or missing if never renovated) |
| `zipcode` | ZIP code of the property |
| `lat` | Latitude |
| `long` | Longitude |
| `sqft_living15` | Average living space of the 15 nearest neighboring houses |
| `sqft_lot15` | Average lot size of the 15 nearest neighboring houses |
| `date` | Date of sale |
| `price` | Sale price (target variable) |
| `is_renovated` | *Derived column* — 1 if the house has been renovated, 0 otherwise (built from `yr_renovated`, with nulls handled before flagging) |
| `sale_month` | *Derived column* — month extracted from `date`, used to check seasonal price effects |


## 4. Data Cleaning
- The is_renovated bug (NaN == 0 issue) and how you fixed it
- Any other cleaning steps

### Missing Data Handling

Four columns contain missing values: `waterfront` (2,360), `view` (63),
`sqft_basement` (451), and `yr_renovated` (3,811). Before deciding how to
handle them, a `missingno` heatmap and matrix were used to check whether the
gaps were random or followed a pattern (e.g. all missing from the same
zip code or time period). Both showed near-zero correlation between which
columns were missing together, and no repeating pattern across rows —
the missingness appears random, not systematic.

Each column's relevance to the client analysis was also considered:

| Column | Missing | Relevance to Jennifer's analysis |
|---|---|---|
| `waterfront` | 2,360 | **Critical** — her hard requirement, and the base filter for H1 and H2 |
| `yr_renovated` | 3,811 | **Critical** — drives `is_renovated`, her stated preference, tested directly in H2 |
| `view` | 63 | Supporting — contributes to the "show off" analysis in H1, not a hard filter |
| `sqft_basement` | 451 | Not relevant — not part of any client wish or hypothesis, cleaned only for dataset completeness |

Given the missingness is random and each of these columns has an established
"absence" value with real meaning (no waterfront, no special view, no
basement, never renovated), missing values were filled with a constant `0`
rather than a statistical measure like mean/median — which would not make
sense for these mostly binary/categorical fields.


### Cross-column consistency check

To catch errors that single-column checks would miss, values were also
checked against related columns for physical plausibility:

- `sqft_living / bedrooms` — flags bedrooms that would be unrealistically
  small. This confirmed the `bedrooms = 33` case: 1,620 sqft ÷ 33 = ~49
  sqft per bedroom (≈ 4.6 m², or a room roughly 213 cm × 213 cm) — smaller
  than a typical bathroom, and not a physically realistic bedroom size.
- `bedrooms` vs. `bathrooms` — checked for houses with many bedrooms but
  almost no bathrooms. No such cases found.
- `sqft_above + sqft_basement` vs. `sqft_living` — checked that the two
  parts of a house add up to the reported total. No mismatches found.

Result: `bedrooms = 33` was the only error caught by these checks —
confirming it as an isolated data-entry mistake rather than a wider
pattern in the dataset.





## Methodology

Before writing the hypotheses, Jennifer's stated wishes were broken down into
concrete, testable criteria — mapping each wish to specific columns in the
dataset:

| Wish | What it means for the data | Column(s) |
|---|---|---|
| High budget | No price ceiling — top end of market is in scope | `price` |
| Wants to show off | Prestigious location + high build quality | `zipcode`, `lat`/`long`, `grade`, `view`, `sqft_living`, `sqft_living15` |
| Timing within a month | Not directly testable (no "days on market" data) — addressed as a supply/inventory question instead | `waterfront`, `grade` (inventory count) |
| Waterfront | Hard requirement | `waterfront` |
| Renovated | Preference, tested as a price premium | `is_renovated`, `yr_renovated` |
| High grades | Hard requirement — threshold decided from the data, not guessed | `grade` |
| Resell within 1 year | Needs a realistic historical benchmark | `date`, `price`, `num_sales`, `sale_dates`, `sale_prices` |

**Deciding the "high grade" threshold:** rather than picking a cutoff
arbitrarily, the grade distribution was examined first. The overall dataset
has a mean grade of 7.66 and a median of 7, with the 75th percentile at only
8 — meaning a grade of 10 is already well above typical. This is also
consistent with King County's own grading scale, where grade 7 = average
construction and grade 10+ is the first tier described as "high quality
features, well-built with good materials." `GRADE_CUTOFF = 10` was chosen on
this basis.

**Limitation:** the dataset has no "days on market" field, so Jennifer's
1-month purchase timeline cannot be tested directly. It is instead addressed
indirectly, by reporting how many houses in the county currently match her
full criteria — low matching inventory implies a 1-month timeline is
optimistic, independent of how fast any single sale might close.




## 5. Chosen Client & Hypotheses
- Jennifer Montgomery, her goals/assumptions
- H1, H2, H3 with context, test, and verdict

## 6. Key Insights & Recommendations

## 7. Limitations




Here's the brief journey, start to finish:

We started with the raw King County housing data — 21,420 houses and their sale records — and first made sure it was trustworthy: verified row counts and referential integrity between the two source tables, preserved full resale history for the 176 houses that sold more than once, fixed the data types, and found and corrected two real data-entry errors (a `bedrooms = 33` typo and a systematic ×10 error in `yr_renovated`).

Then we translated Jennifer's 8 wishes into a concrete filter: waterfront houses, grade 10 or higher, with complete data on view and renovation status — narrowing 21,420 houses down to a verified shortlist of 46 candidates.

From there, we tested each of her remaining wishes against the real data rather than assuming them: we discovered renovation doesn't actually predict a better-condition house (condition does), quantified "showing off" through both size-relative-to-neighbors and neighborhood prestige (leading to your geographical insight and map), estimated how rarely matching houses appear on the market for the one-month timing wish, and honestly assessed the one-year resale question using the closest available comparable evidence, while flagging that no true waterfront precedent exists in the data.

To bring it all together, we built two scoring systems — a condition-based quality tier and a location-based prestige tier — and combined them into a single match score reflecting all her core criteria at once. Ranking all 46 candidates by that score revealed one house standing clearly above the rest.

Finally, we verified that house wasn't just "good because the whole list is good" — checking its percentile ranking against both the entire county and her own shortlist confirmed it's a genuine standout on price, size, and condition specifically, not just a lucky pick.

That's the full path from 21,420 raw houses to one confidently recommended home for Jennifer — every step documented, every assumption stated, and the final pick backed by both a visual and a number.

https://www.google.com/maps?q=47.6515,-122.277


