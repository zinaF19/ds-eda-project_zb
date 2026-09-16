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

## 3. Data Dictionary
(the column table you already built)

## 4. Data Cleaning
- The is_renovated bug (NaN == 0 issue) and how you fixed it
- Any other cleaning steps

## 5. Chosen Client & Hypotheses
- Jennifer Montgomery, her goals/assumptions
- H1, H2, H3 with context, test, and verdict

## 6. Key Insights & Recommendations

## 7. Limitations





## Data Dictionary

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


