# CAL-culator

A single-page web tool that recommends how much **carbohydrate** and **sodium** to prep
for a training session, based on session type, duration, and weather (temperature +
humidity, auto-fetched by date and location).

## Status

- ✅ Formulation fact-checked and locked — see [SPEC.md](SPEC.md)
- ✅ Tool built (`index.html`) — carbs, fluid & sodium, Open-Meteo forecast + location autocomplete
- ✅ Workout-based session entry (paste a planned workout → auto-derive session type + duration)

## How it works (short version)

- **Carbs** are driven by session type × duration (zero for non-race sessions ≤ 65 min, tapered in to 75 min), capped at the gut's absorption
  ceiling (90 g/h standard, 120 g/h with the gut-trained toggle).
- **Sodium** = sweat rate × duration × sweat-sodium concentration, with sweat rate
  scaled by intensity and weather.
- Outputs are shown as total grams to prep, broken down into real ingredients
  (glucose / glucose:fructose / sucrose for carbs; table salt or sodium citrate for sodium).
- **Workout entry (optional):** paste a planned session — time-based (`15' easy, 20'
  tempo`) or distance-based (`14km easy, 3km HMP`, using your zone paces) — and the tool
  derives the session type + duration for you. See [SPEC.md](SPEC.md) §6.

See [SPEC.md](SPEC.md) for the full validated formulation, the carb matrices, the
sodium chain, and the peer-reviewed sources behind every number.

## Usage

Open `index.html` in a browser (works on mobile). No build step, no dependencies.
