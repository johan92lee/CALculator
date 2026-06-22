# CAL-culator

A single-page web tool that recommends how much **carbohydrate** and **sodium** to prep
for a training session, based on session type, duration, and weather (temperature +
humidity, auto-fetched by date and location).

## Status

- ✅ Formulation fact-checked and locked — see [SPEC.md](SPEC.md)
- 🔨 Tool (`index.html`) — in progress

## How it works (short version)

- **Carbs** are driven by session type × duration, capped at the gut's absorption
  ceiling (90 g/h standard, 120 g/h with the gut-trained toggle).
- **Sodium** = sweat rate × duration × sweat-sodium concentration, with sweat rate
  scaled by intensity and weather.
- Outputs are shown as total grams to prep, broken down into real ingredients
  (glucose / glucose:fructose / sucrose for carbs; table salt or sodium citrate for sodium).

See [SPEC.md](SPEC.md) for the full validated formulation, the carb matrices, the
sodium chain, and the peer-reviewed sources behind every number.

## Usage

Open `index.html` in a browser (works on mobile). No build step, no dependencies.
