# Training Fueling Calculator — Validated Spec

> A tool to calculate how much **carbohydrate** and **sodium** to prep for a training session, driven by session type, duration, and weather. This document is the fact-checked formulation; the tool is **not yet built**.

_Last updated: 2026-06-21_

---

## 1. What it is

A **single self-contained `index.html`** page (HTML + CSS + JS, no dependencies) that you can open on your phone before a session. You enter the session and conditions; it outputs the total carbs and sodium to prep, broken down into actual ingredients to weigh out.

## 2. Inputs

| Input | Type | Notes |
|---|---|---|
| Session type | dropdown | Recovery/Easy, Long (steady), Tempo/Threshold, Intervals/VO₂, Race/Time trial |
| Duration | minutes | primary carb driver |
| Date | date picker | used to fetch forecast |
| Location | lat/lon or geolocation | required for forecast |
| Temperature | °C | auto-filled from forecast, editable |
| Humidity | % | auto-filled from forecast, editable |
| Body weight | kg | scales sweat rate; flags >2% body-mass loss |
| Gut-trained toggle | on/off | raises carb ceiling 90 → 120 g/h |
| Salty-sweater toggle | on/off | raises sweat sodium concentration |
| Carb format | dropdown | Glucose / 1:0.8 glucose:fructose / Sucrose |
| Sodium format | dropdown | Table salt / Sodium citrate |

**Weather:** date + location → **Open-Meteo** forecast API (free, no key, hourly temp + relative humidity, ~16-day horizon). Needs date, location, and time of day. If date is beyond the forecast window, fall back to manual entry / seasonal averages.

---

## 3. Carbohydrate model

**Two main drivers: duration × session type.** Duration sets the band (from the guidelines); session type positions within the band (higher intensity → faster glycogen turnover → fuel toward the top). Intake never exceeds the gut absorption ceiling.

**Duration bands (non-overlapping):** `< 45` · `45–75` · `75–150` · `> 150` min.
The tool **interpolates by exact duration within each band** to avoid hard jumps.

### Standard matrix — ceiling 90 g/h

| Session type | < 45 m | 45–75 m | 75–150 m | > 150 m |
|---|---|---|---|---|
| Recovery / Easy | 0 | 0 | 30 | 60 |
| Long (steady) | 0 | 15 | 40 | 70 |
| Tempo / Threshold | 0 | 25 | 50 | 90 |
| Intervals / VO₂ | 0 | 25 | 50 | 90 |
| Race / Time trial | 0–30 | 30 | 60 | 90 |

### Gut-trained matrix — ceiling 120 g/h

Rule: cells that are **absorption-limited (≥ 45 g/h)** scale ~1.33× toward the 120 ceiling; cells that are **demand-limited (≤ 40 g/h)** stay unchanged (no metabolic need to eat more just because the gut can absorb more).

| Session type | < 45 m | 45–75 m | 75–150 m | > 150 m |
|---|---|---|---|---|
| Recovery / Easy | 0 | 0 | 30 | **80** |
| Long (steady) | 0 | 15 | 40 | **95** |
| Tempo / Threshold | 0 | 25 | **65** | **120** |
| Intervals / VO₂ | 0 | 25 | **65** | **120** |
| Race / Time trial | 0–30 | 30 | **80** | **120** |

**Total carbs to prep (g) = chosen rate (g/h) × duration (h).**

### Carb format dropdown — affects ceiling AND ingredient breakdown

| Format | Absorption ceiling | Breakdown of total carb (g) |
|---|---|---|
| Glucose (single SGLT1 transporter) | **~60 g/h** | 100% glucose powder (dextrose monohydrate ≈ 91% carb) |
| 1:0.8 glucose:fructose (multi-transportable) | up to 90 (120 gut-trained) | glucose = 1/1.8 (55.6%), fructose = 0.8/1.8 (44.4%) → e.g. 90 g = 50 g glucose + 40 g fructose |
| Sucrose / table sugar (glucose+fructose disaccharide) | up to ~90 g/h | 100% carb → grams = total carb |

**Rule:** when the recommended rate **> 60 g/h**, the **Glucose** option is disabled/greyed out, with a note: _"Rate exceeds glucose-only absorption — use 1:0.8 glucose:fructose or sucrose."_

---

## 4. Sodium model

**Sodium loss (mg) = Sweat rate (L/h) × Duration (h) × Sweat sodium concentration (mg/L)**

- **Sweat rate** anchored to validated range **0.5–2.0 L/h**, modulated by intensity (metabolic heat) and weather (temp ↑, humidity ↑ → less evaporative cooling → more sweat). Heuristic kept inside the 0.5–2.0 envelope; directionally consistent with Gonzalez 2009 prediction equations.
- **Sweat sodium concentration:** default **~1,000 mg/L** (= 42.9 mmol/L marathoner mean; 1 mmol/L Na = 23 mg/L). Salty-sweater setting **~1,500 mg/L** (≈65 mmol/L). Full population range 10–90 mmol/L (~230–2,070 mg/L).
- **Body weight** scales sweat rate and flags when projected loss exceeds ~2% body mass.

### Sodium format dropdown — conversions

| Format | Sodium (mg) → grams |
|---|---|
| Table salt (NaCl, 39.3% Na) | grams = mg ÷ 393 |
| Sodium citrate (23.4% Na) | grams = mg ÷ 234 |

**Output format:** `Total sodium = 1,000 mg → 2.54 g table salt` _(or)_ `→ 4.27 g sodium citrate`

> Note: dropped "dihydrate" label from the UI; ÷234 still corresponds to the common food-grade form. (Anhydrous form, 26.7% Na / ÷267, was removed as an option.)

---

## 5. Outputs

1. **Total carbs to prep (g)** + ingredient breakdown per chosen carb format (+ warning if rate exceeds chosen format's ceiling).
2. **Total sodium (mg)** → grams of table salt **or** sodium citrate per chosen sodium format.

---

## 6. Future pipeline (parked)

1. **Lab-test carb-oxidation** — if a metabolic test gives measured carb oxidation rate, use it for glycogen-drawdown estimates and pre-session loading (NOT to raise intake, which stays absorption-capped).
2. **Race-route fueling planner** — upload race route, detect aid/water stations, recommend a fueling strategy that times carb + fluid intake to where you can pick up or carry fuel.
3. **Personal sweat test calibration** — weigh-in before/after a run to replace population-average sweat rate and sodium concentration with your own numbers (gold standard per Baker 2017).

---

## 7. Key science notes

- **Exogenous vs endogenous oxidation.** Exogenous = carbs ingested during exercise (gels/drinks). Endogenous = your own stored glycogen (muscle + liver). Fueling aims to raise exogenous oxidation to **spare** limited endogenous glycogen and delay depletion ("bonking", ~90–120 min of hard effort). The 120-vs-90 g/h study showed 120 raised exogenous oxidation but did **not** spare endogenous glycogen better than 90 — which is why 90 stays the default and 120 is reserved for the longest efforts.
- **The gut is the ceiling, not intensity.** Carb oxidation during exercise is capped by intestinal transport (~60 g/h glucose-only; ~90 with glucose+fructose; up to ~120 trained). Higher intensity burns more glycogen but you can't absorb more, so intensity selects position within a band, not above the ceiling.

---

## 8. What's directly peer-reviewed vs. heuristic

- **Peer-reviewed (hard numbers):** carb duration bands & the 30–60 / 90 / 120 g/h ceilings; sweat rate range 0.5–2.0 L/h; sweat sodium concentration ranges; the salt/citrate chemistry conversions.
- **Practical heuristic (defensible, not separately RCT-validated):** session-type positioning within each band; the gut-trained ×1.33 scaling; the temp/humidity → sweat-rate scaling (simplified from validated prediction equations).

---

## 9. Sources

- 2016 Joint Position Stand — Nutrition and Athletic Performance (AND/DC/ACSM; Thomas, Erdman & Burke, *Med Sci Sports Exerc* 48(3):543–568)
- Jeukendrup A. (2014) Carbohydrate Intake During Exercise, *Sports Medicine* — https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4008807/
- 120 vs 90 g/h carbohydrate oxidation study — https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9560939/
- Baker LB. (2017) Sweating Rate & Sweat Sodium review, *Sports Medicine* — https://link.springer.com/article/10.1007/s40279-017-0691-5
- Gonzalez RR et al. (2009) Expanded prediction equations of human sweat loss, *J Appl Physiol* — https://pubmed.ncbi.nlm.nih.gov/19407259/
- Lara B et al. (2016) Sweat electrolyte variability in marathoners, *J Int Soc Sports Nutr* — https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4966593/
- ACSM (2007) Exercise and Fluid Replacement position stand — https://www.researchgate.net/publication/232208129_ACSM_Position_Stand_Exercise_and_Fluid_Replacement
- Open-Meteo forecast API — https://open-meteo.com/

---

## 10. Status

- ✅ Formulation fully specified and fact-checked
- ⬜ Build the `index.html` tool (next step — awaiting go-ahead)
