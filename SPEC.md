# Training Fueling Calculator — Validated Spec

> A tool to calculate how much **carbohydrate** and **sodium** to prep for a training session, driven by session type, duration, and weather. This document is the fact-checked formulation and feature spec; the tool is **built** at `index.html`.

_Last updated: 2026-06-27_

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

**Weather:** date + location → **Open-Meteo** forecast API (free, no key, hourly temp + relative humidity, ~16-day horizon). The forecast **auto-fetches when a location is selected** (pick a suggestion, type + blur, or 📍 current location) and re-fetches when the date or start time changes — no button. If date is beyond the forecast window, fall back to manual entry / seasonal averages.

**Workout entry (optional):** instead of picking session type + duration by hand, paste a planned workout (time- or distance-based) and the tool derives both and auto-fills them — see §6.

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

## 4. Fluid & sodium model

> **Key correction (v2):** total sweat *losses* are NOT what you consume during the run.
> You only *partially* replace fluid and sodium while running, capped by what you can
> actually drink. The tool reports **recommended intake** as the headline number, with
> total losses shown only as context. (v1 wrongly presented total losses as intake.)

**Estimated sweat loss — context only, informational:**

- `Sweat loss (L) = Sweat rate (L/h) × Duration (h)`
- `Sodium lost (mg) = Sweat loss (L) × Sweat sodium concentration (mg/L)`
- **Sweat rate** anchored to the validated **0.5–2.0 L/h** range (Baker 2017; ACSM 2007), scaled by intensity and weather (temp ↑, humidity ↑ → more sweat); kept inside the envelope, directionally consistent with Gonzalez 2009.
- **Sweat sodium concentration:** ~**1,000 mg/L** default (42.9 mmol/L marathoner mean), salty-sweater **~1,500 mg/L**.

**Recommended fluid intake — actionable:**

- `Intake rate (L/h) = min(sweat rate, 1.0 L/h cap)` → `Fluid to carry (L) = intake rate × duration`
- Guidance is **drink to thirst**, up to **~1.0 L/h in the heat** — the gastric-emptying limit. Warm conditions (≤35 °C) don't impair emptying unless you're already dehydrated; only extreme heat (~49 °C) or hypohydration cut it ~20–25%. Never exceed your sweat rate — over-drinking causes hyponatremia.
- Finishing **2–3% of body mass down is normal** on long/hot efforts; rehydrate afterwards. The tool flags only large (>5%) projected deficits.

**Recommended sodium intake — the "to prep" number, actionable:**

Scales with losses so it responds to heat, then bounded by tolerance and palatability:
- `Loss rate (mg/h) = sweat rate × sweat-sodium concentration`
- `Target (mg/h) = clamp(loss rate × 60%, 300, 1500)` — partial replacement (top up after the run); floored 300, capped 1,500 mg/h for GI tolerance.
- `Sodium to add (mg) = min(target, intake rate × 1,500 mg/L) × duration` — delivered in the drink up to a **1,500 mg/L** palatability cap.
- **Salty-sweater** raises sweat-sodium (1,000 → 1,500 mg/L) → higher loss rate → higher target.
- Sodium matters most for efforts **> ~2 h**, **high sweat rates (>1.2 L/h)**, or **salty sweaters**; for short cool runs it's optional.

> **Why concentration scales (not just fluid):** in heat you can't drink more than ~1 L/h, so to replace more sodium the *drink* gets saltier — up to 1,500 mg/L (Johan's chosen cap). Palatability and voluntary intake drop above ~50 mmol/L (~1,150 mg/L), so the tool flags when the mix is strong. Beyond the drink's capacity, the excess should come from **salt capsules taken with water** — *parked; drink-only for now*. Replacement fraction (60%), the 1,500 mg/h ceiling, and the 1,500 mg/L cap are all tunable.

### Sodium source dropdown — conversions

| Format | Sodium (mg) → grams |
|---|---|
| Table salt (NaCl, 39.3% Na) | grams = mg ÷ 393 |
| Sodium citrate (23.4% Na) | grams = mg ÷ 234 |

**Output format:** `Total sodium = 1,000 mg → 2.54 g table salt` _(or)_ `→ 4.27 g sodium citrate`

> Note: dropped "dihydrate" label from the UI; ÷234 still corresponds to the common food-grade form. (Anhydrous form, 26.7% Na / ÷267, was removed as an option.)

---

## 5. Outputs

1. **Total carbs to prep (g)** + ingredient breakdown per chosen carb format (+ warning if rate exceeds chosen format's ceiling).
2. **Sodium to add to drinks (g)** of table salt **or** sodium citrate — recommended *intake* (partial replacement), mixed across your planned fluid.
3. **Recommended fluid intake** (mL/h + total L, drink-to-thirst), with estimated sweat loss and projected deficit shown as context.

---

## 6. Workout-based session entry (optional)

Instead of manually choosing session type + duration, paste a planned workout and the tool **derives both** and auto-fills them. Fully client-side — no LLM, no backend.

### Input mode toggle: time-based or distance-based

The toggle only changes **how each segment's duration is derived**; classification and all downstream carb/sodium logic are identical in both modes.

- **Time-based** — segments carry a duration: `15' easy, 5x3' hard w/2' jog, 10' cd`. Duration is read directly; no paces needed.
- **Distance-based** — segments carry a distance: `14km easy, 3km HMP`. Duration = distance × the pace for that segment's intensity, drawn from the **My paces** panel (or an inline `@m:ss`).

### Paste syntax (grammar)

- Segments separated by `,`, a new line, `+`, or the word `into`.
- **Time:** `15'`, `15min`, `15m`, `90s`, `1:30` (mm:ss).
- **Distance:** `14km`, `400m`, `5k`, `1.5km`, `1mi`.
- **Reps:** `5x1km`, `8x90s`, `6x3'` — optional recovery `w/<time|distance> <label>`.
- **Inline pace (distance):** `@3:50` (per km).
- **Intensity words → matrix row · pace zone:**
  - `easy / recovery / wu / cd / jog / z1 / z2` → Easy · Easy pace
  - `long / steady` → Long · Easy pace
  - `mp / marathon` → Long · MP pace
  - `hmp / half` → Tempo · HMP pace
  - `tempo / threshold / t / z4` → Tempo · Threshold pace
  - `intervals / vo2 / reps / hard / 5k / 3k / z5` → Intervals · Interval pace
  - `race / tt` → Race · HMP pace (default)
- A rep block with recovery but no label defaults to **Intervals**.

### My paces panel (distance mode only)

Five editable zone paces (min:ss /km); defaults: Easy 5:30 · MP 4:30 · HMP 4:00 · Threshold 3:55 · Interval 3:40. Hidden in time mode. The same paces feed the parked carb-expenditure model later — entered once, reused.

### Parser → segments → effort

1. Parse each segment into `{ durationMin, row, pace?, distance? }`. Distance × pace → duration; recoveries count as Easy and add to total time.
2. **Total duration** = sum of all segments (warm-up + work + recoveries + cool-down).
3. **Session row (classification):** sum duration by row; the session row = the **highest-intensity** row whose total time is **≥ max(10 min, 15% of session)**; if none qualifies, the row with the most total time. This filters tiny additions (e.g. strides) while letting a real quality block define the session.
4. Auto-fill the session-type dropdown + duration → the normal carb + sodium readout updates. The derived row also drives the sweat-rate base, so sodium stays consistent.

### Worked example — `14km easy into 3km HMP` (distance mode, Easy 5:30 / HMP 4:00)

- 14 km easy → 77 min; 3 km HMP → 12 min; **total 89 min**.
- Quality gate = max(10, 15% × 89) = 13.4 min; the 12-min HMP finish just misses → classified **Long** (aerobic run with a fast finish, not a race session).
- → carb rate ≈ 46 g/h → **~68 g**; sodium from 89 min + weather.

### Output

Just **auto-fills** session type + duration (no extra output panel). A small parsed-summary line confirms what was read (e.g. `14 km easy (77 min) + 3 km HMP (12 min) = 89 min · Long`) and flags any segment it couldn't time (a distance with no pace).

---

## 7. Future pipeline (parked)

1. **Free-text / screenshot workout parsing** — paste un-structured coach notes or a screenshot; needs an LLM (Claude Haiku via a small serverless proxy, or bring-your-own-key) + hosting, so deferred from the offline-file build. The paste-syntax planner (§6) is the no-backend version.
2. **Carb-expenditure model** — use segment pace × intensity (the My-paces data already captured) to estimate g/min carb burn and glycogen drawdown, refining where in the band the rate sits. Intake stays absorption-capped.
3. **Lab-test carb-oxidation** — measured oxidation rate for glycogen-drawdown estimates and pre-session loading.
4. **Race-route fueling planner** — upload race route, detect aid/water stations, time carb + fluid intake to them.
5. **Personal sweat-test calibration** — weigh-in before/after a run to replace population-average sweat rate and sodium concentration with your own numbers (gold standard per Baker 2017).
6. **Salt-capsule overflow** — when the sodium target exceeds the 1,500 mg/L drink cap, route the remainder to salt capsules taken with water.

---

## 8. Key science notes

- **Exogenous vs endogenous oxidation.** Exogenous = carbs ingested during exercise (gels/drinks). Endogenous = your own stored glycogen (muscle + liver). Fueling aims to raise exogenous oxidation to **spare** limited endogenous glycogen and delay depletion ("bonking", ~90–120 min of hard effort). The 120-vs-90 g/h study showed 120 raised exogenous oxidation but did **not** spare endogenous glycogen better than 90 — which is why 90 stays the default and 120 is reserved for the longest efforts.
- **The gut is the ceiling, not intensity.** Carb oxidation during exercise is capped by intestinal transport (~60 g/h glucose-only; ~90 with glucose+fructose; up to ~120 trained). Higher intensity burns more glycogen but you can't absorb more, so intensity selects position within a band, not above the ceiling.

---

## 9. What's directly peer-reviewed vs. heuristic

- **Peer-reviewed (hard numbers):** carb duration bands & the 30–60 / 90 / 120 g/h ceilings; sweat rate range 0.5–2.0 L/h; sweat sodium concentration ranges; the salt/citrate chemistry conversions.
- **Practical heuristic (defensible, not separately RCT-validated):** session-type positioning within each band; the gut-trained ×1.33 scaling; the temp/humidity → sweat-rate scaling (simplified from validated prediction equations).

---

## 10. Sources

- 2016 Joint Position Stand — Nutrition and Athletic Performance (AND/DC/ACSM; Thomas, Erdman & Burke, *Med Sci Sports Exerc* 48(3):543–568)
- Jeukendrup A. (2014) Carbohydrate Intake During Exercise, *Sports Medicine* — https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4008807/
- 120 vs 90 g/h carbohydrate oxidation study — https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9560939/
- Baker LB. (2017) Sweating Rate & Sweat Sodium review, *Sports Medicine* — https://link.springer.com/article/10.1007/s40279-017-0691-5
- Gonzalez RR et al. (2009) Expanded prediction equations of human sweat loss, *J Appl Physiol* — https://pubmed.ncbi.nlm.nih.gov/19407259/
- Lara B et al. (2016) Sweat electrolyte variability in marathoners, *J Int Soc Sports Nutr* — https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4966593/
- ACSM (2007) Exercise and Fluid Replacement position stand (fluid ~400–800 mL/h; sodium 0.5–0.7 g/L; hyponatremia) — https://www.researchgate.net/publication/232208129_ACSM_Position_Stand_Exercise_and_Fluid_Replacement
- GSSI SSE-122 — Sodium Ingestion, Thirst and Drinking During Endurance Exercise — https://www.gssiweb.org/sports-science-exchange/article/sse-122-sodium-ingestion-thirst-and-drinking-during-endurance-exercise
- Gastric emptying during exercise: effects of heat stress and hypohydration (PubMed 2920721) — https://pubmed.ncbi.nlm.nih.gov/2920721/
- Compositional aspects of hydration beverages / sodium palatability ≥50 mmol/L (MDPI 2024) — https://pmc.ncbi.nlm.nih.gov/articles/PMC10781183/
- Beverage sodium concentration, plasma sodium & volume in the heat (RCT) — https://pmc.ncbi.nlm.nih.gov/articles/PMC9813217/
- Open-Meteo forecast API — https://open-meteo.com/

---

## 11. Status

- ✅ Formulation fully specified and fact-checked
- ✅ Tool built at `index.html` (carbs, fluid & sodium, Open-Meteo forecast + location autocomplete, two-column design)
- 🔨 Workout-based session entry (§6) — time/distance toggle + paces panel + paste parser
