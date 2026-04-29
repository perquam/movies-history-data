# movies-history-data

Community data repository for the **movie-history** app. Use this repo to contribute new data and to correct historical inaccuracies in the data the app ships with.

All pull requests are moderated. Accepted changes are merged into the app **once per month**.

---

## What you can contribute

The repo is split into four areas. Each one has different contribution rules.

### 1. `achievements.json`

A flat list of viewing achievements. You can **add** new achievements and **correct** existing ones.

Rules for a new achievement:

- **Minimum 3 movies** in `required_movies_to_solve` (more is fine)
- `achievement_name`, `description`, and `icon` are required
- `title_hint` should mirror `required_movies_to_solve` one-for-one (same order, same length)

#### Achievements ↔ movies ↔ quizzes

Every achievement is a set of movies. The quiz that gates each movie is **not** stored on the achievement — it lives on the movie entry itself, in the `quiz` field inside `per_epoch_movie_data/<epoch>/movies_<continent>.json`.

**Therefore: every movie listed in an achievement's `required_movies_to_solve` MUST have a `quiz` on its movie entry.** No exceptions. If a movie you want to reference does not yet have a quiz, you must add one to that movie's entry **in the same PR** as the achievement.

Quiz shape (always **MAX 2 questions**, and it depends on the achievement type):

- **Historical achievement** (movies depict real history) — each referenced movie's `quiz` has exactly **2 questions**:
  1. A question about the movie itself (`"type": "plot"`)
  2. A question about the historical background — a real historical fact the movie depicts (`"type": "historical"`)
- **Non-historical achievement** (pure fiction, no real-world historical content) — each referenced movie's `quiz` has exactly **1 question**, about the **plot** of the movie (`"type": "plot"`)

Each question has three options `A`, `B`, `C` and a `correct` letter.

#### How to add a new achievement — worked example

Say you are adding a "Stalingrad" achievement that includes **Enemy at the Gates** (`movie_id: 853`), which lives in `per_epoch_movie_data/age-of-total-war/movies_europe.json`. It is a historical achievement.

**Step 1 — add the achievement to `achievements.json`:**

```json
{
  "achievement_name": "Stalingrad or Bust",
  "description": "Certified tourist of the Eastern Front. Winter coat optional, survival not guaranteed.",
  "icon": "❄️",
  "required_movies_to_solve": [25237, 853, 31442, 613],
  "title_hint": [
    "Come and See",
    "Enemy at the Gates",
    "Ivan's Childhood",
    "Downfall"
  ]
}
```

**Step 2 — open each referenced movie's entry and make sure it has a `quiz`.** For Enemy at the Gates (`movie_id: 853`) in `per_epoch_movie_data/age-of-total-war/movies_europe.json`, the quiz field must look like this (2 questions because the achievement is historical):

```json
"quiz": [
  {
    "question": "What happens when Vassili finally confronts the German sniper König?",
    "type": "plot",
    "options": {
      "A": "Danilov shoots König from across the river",
      "B": "König escapes after Vassili is captured",
      "C": "Vassili tricks König into revealing himself and then shoots him"
    },
    "correct": "C"
  },
  {
    "question": "Which real World War II battle is the film set during?",
    "type": "historical",
    "options": {
      "A": "The Battle of Moscow",
      "B": "The Battle of Stalingrad",
      "C": "The Battle of Kursk"
    },
    "correct": "B"
  }
]
```

Repeat Step 2 for **every** movie id in `required_movies_to_solve`. If the movie already has a `quiz` of the correct shape, leave it alone. If it has no `quiz`, or the shape doesn't match the achievement type (1 plot question for non-historical, 1 plot + 1 historical for historical), add or fix it in the same PR.

### 2. `epochs_core_data/`

One JSON file per epoch (e.g. `ancient-world.json`, `industrial-age.json`).

You can contribute changes to:

- `aliases` — alternative names for the epoch
- `headline` — the one-line tagline
- `epoch_summary` — the longer paragraph describing the epoch
- `regions[].description_of_an_epoch_in_the_region` — per-region descriptions of what the epoch looks like there

You **cannot** change: `id`, `label`, `time_range`, region names.

### 3. `per_epoch_category_data/`

One folder per epoch, with files split by continent (e.g. `ancient-world/categories_europe.json`).

The `categories` array at the bottom of each file is fully open:

- **Edit** any existing category's `label`, `description`, or `icon`
- **Add** new categories
- **Remove** categories that don't fit the epoch/region

A category describes the **background state of the world** that a movie set in this epoch and region depicts — the political order, the technology level, the social conditions a viewer sees on screen. It is *not* a genre and *not* a plot tag.

Categories must be **factual** — grounded in real, documented historical states of the world for that epoch and region. Do not invent fictional regimes, made-up technologies, or imaginary social orders. Examples from the Ancient World / Europe set: *"rome holding provinces with soldiers"*, *"war bands seeking fame and plunder"*, *"gods, omens, and sacred fate"*.

The `sample` array of movies in these files is reference material — do not edit it as part of category contributions.

### 4. `per_epoch_movie_data/`

One folder per epoch, with files split by continent (e.g. `ancient-world/movies_europe.json`). Each entry is a movie.

**Correcting an existing movie.** You can propose corrections to any field **except** `movie_id`, `movie_idx`, and `imdb_id`. Common corrections: wrong `plot_year_start` / `plot_year_end`, wrong `action_countries` / `action_continents`, miscategorized `categories`, wrong `epoch`, etc.

**Adding a new movie.** Every field except the IDs must be filled in by hand. ID fields are assigned by the maintainers after the PR is accepted — leave them out, or leave them as `null`.

Field rules when adding or correcting a movie:

- `movie_id`, `movie_idx`, `imdb_id` — **do not fill in.** Maintainers assign these.
- `wikipedia_cache_file` — **leave empty / do not fill in.** This is generated by the pipeline.
- `poster_file` — **leave empty.** This is generated by the pipeline.
- `poster` — must be a **direct link to a `.jpg` image** (the URL ends in `.jpg`). No HTML pages, no thumbnails embedded in other formats.
- `action_countries` — use **ISO 3166-1 alpha-2** country codes (`"GR"`, `"IT"`, `"GB"`, `"US"`, …), one entry per country where the action takes place.
- `action_universum` — for any movie set in the real world, use the exact string `"Real World"` (always, no variations). For fictional universes, use the canonical universe name, e.g. `"Marvel Cinematic Universe"`, `"Star Wars"`, `"Middle-earth"`, `"Harry Potter"`, `"Star Trek"`, `"DC Extended Universe"`.
- `categories` — must be picked from the **existing** category list in the matching `per_epoch_category_data/<epoch>/categories_<continent>.json` file. **Copy the chosen category object exactly** — `label`, `description`, and `icon` must match the source verbatim. Do not invent categories inline. If the category you need does not exist yet, add it to `per_epoch_category_data/` in the same PR.

Reminder: a category is the **background state of the world in the movie**, not the plot. Pick the categories that describe the world the characters live in.

---

## How to contribute

1. Fork the repo.
2. Make your edits. Keep one logical change per PR (e.g. don't bundle a movie correction with an unrelated achievement addition).
3. Open a pull request with a short description of what you changed and why. For factual corrections, link a source where possible.
4. Wait for moderation. Accepted changes ship in the next monthly batch to the app.

## Repo layout

```
achievements.json             viewing achievements
epochs_core_data/             one JSON per epoch — aliases, headlines, summaries, regions
per_epoch_category_data/      one folder per epoch — category definitions per continent
per_epoch_movie_data/         one folder per epoch — movie entries per continent
```
