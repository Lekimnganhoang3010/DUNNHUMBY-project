# Grocery Campaign Effectiveness & Retention (Dunnhumby "The Complete Journey")

Do grocery marketing campaigns change how customers shop, and does the change last?
This project analyzes 2 years of loyalty-card transactions from 2,500 households and 30 direct-marketing campaigns.

**Status:** Data cleaning and exploratory analysis complete. Core analysis (BQ1, BQ2) in progress.

---

## Business questions

**BQ1. Which campaign types drive the biggest short-term lift in spending?**
- 1a Campaign & audience profile: Type A/B/C by duration, reach, coupons, categories, and who each type targets
- 1b Spending lift and basket size vs. each household's own baseline and a matched comparison group
- 1b+ Linear regression on lift drivers (campaign and household features), checked against 1b
- 1c *(optional)* Who responds

**BQ2. Do campaigns build lasting customer value, or just a temporary bump?**
- 2a Persistence of spending in the 8 weeks after a campaign
- 2b Full-price repurchase of coupon products (new vs. existing buyers)
- 2c Drop-off risk after a campaign (logistic regression vs. random forest)

---

## Data

[The Complete Journey](https://www.dunnhumby.com/source-files/) by dunnhumby: transactions, products, household demographics, campaigns, coupons, and coupon redemptions.

The data is **not included** in this repository. To reproduce:
1. Download the dataset from dunnhumby's Source Files page.
2. Place the CSV files in `data/raw/`.
3. Run `notebooks/01_data_overview.ipynb` to create the cleaned files in `data/clean/`.

---

## Repository structure

```
├── data/
│   ├── raw/       # original CSVs (not tracked)
│   └── clean/     # cleaned outputs (not tracked)
├── notebooks/
│   ├── 01_data_overview.ipynb   # data model, key checks, health check, cleaning
│   └── 02_eda.ipynb             # targeted exploratory analysis
├── requirements.txt
└── README.md
```

## Setup

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

---

## Progress

- [x] Step 1: Data model overview (table inventory, primary/foreign key checks, coverage)
- [x] Step 2: Data health check and cleaning
- [x] Step 3: Columns and metric definitions by business question
- [x] Step 4: Targeted EDA (households, campaigns, audiences, redemptions, comparison groups)
- [ ] Step 5: One-page data summary
- [ ] BQ1: Campaign profile, spending lift, regression
- [ ] BQ2: Persistence, repurchase, drop-off model
- [ ] Final charts / dashboard, report, slides

---

## Data cleaning summary

| Issue | Decision | Impact |
|---|---|---|
| Coupon table exact duplicates | Removed | 5,164 rows |
| Redemptions credited to two overlapping campaigns | Kept for campaign-level analysis; deduplicated for totals | 14 rows |
| Transactions with quantity 0 and sales 0 | Removed (no purchase) | 14,399 rows |
| Non-grocery items: fuel (COUPON/MISC ITEMS is ~94% gasoline, KIOSK-GAS), tickets, misc transactions | Removed | ~8.5% of spend |
| Panel ramp-up (weeks 1-16) and tail-off (weeks 101-102) | Analysis window = weeks 17-100 | - |
| `WEEK_NO` differs from `DAY // 7 + 1` for 14% of rows | Campaign windows defined in days | - |
| Anonymized demographics (`classification_1`-`5`) | Mapped by value ranges (age confirmed; others inferred); income grouped into 4 bands | 801 households |

**Result:** 2,548,827 clean transactions (1.8% of rows removed), all 2,500 households retained.

---

## Key methodology decisions

- **Windows:** 8-week baseline before each campaign and 8-week post period after it (56 days each).
- **Eligible campaigns:** 29 for BQ1 (campaign 15 excluded: 161 days long, 100% overlap, 17 households) and 23 for BQ2 (campaigns ending too close to the data's end excluded).
- **Campaign overlap:** 59% of household-campaign exposures overlap another campaign. Kept and flagged (`has_overlap`, `n_concurrent`); lift is validated on non-overlapping exposures.
- **Minimum activity:** exposures need at least 2 baseline trips (185 of 7,191 removed).
- **Comparison group:** households targeted at other times but not in any campaign during the window, matched on baseline spend. Never-targeted households are used only as a secondary check.
- **Metrics:** medians and % change from each household's own baseline, because spend is right-skewed.

---

## Early findings (EDA)

**Households**
- The median household spends **$21.80 per week** (mean $31.80) and makes **0.77 trips per week**.
- Households differ more in **how often** they shop than in **how much per trip**.

**Campaign types**
- **TypeA:** large and personalized: 332-1,133 households per campaign, each receiving 16 coupons tailored to past purchases; covers the widest range of departments, including fresh food.
- **TypeB:** small and short: mostly 33-day campaigns, 13-276 households, grocery-focused.
- **TypeC:** small and long: median 64 days, focused on Drug/GM and grocery.

**Targeting and response**
- Demographic profiles are **similar across types**, but TypeB/C audiences spend **~50% more per week** before the campaign: targeting appears to be based on purchase behavior.
- **TypeA has double the redemption rate** (16.0% vs. ~8% for TypeB/C), despite reaching lighter shoppers.
- Redeemers were already heavier shoppers (median spend $5,022 vs. $2,632), so redemption alone does not show that coupons increase spend.
- Never-targeted households are light shoppers (median $696 vs. $3,192 over two years), so they are not a fair comparison group on their own.

---

## Limitations

- Campaigns were not randomly assigned, so results are **estimated lift**, not proven causal effects.
- Heavy campaign overlap limits attribution to a single campaign, especially for TypeC (92-100% overlap).
- Demographics cover 32% of households, mostly engaged and targeted customers.
- TypeA coupons are personalized, and the dataset does not show which 16 each household received.

---

## Tools

Python (pandas, NumPy, matplotlib, seaborn, scikit-learn), Jupyter, Git/GitHub.
