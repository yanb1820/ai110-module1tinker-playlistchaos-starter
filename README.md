# Playlist Chaos

A smart playlist generator built with Streamlit. Songs are automatically classified into Hype, Chill, or Mixed playlists based on energy levels, genre keywords, and a configurable mood profile.

## Getting Started

### Prerequisites

- Python 3.10+
- Streamlit

### Installation

```bash
pip install streamlit
```

### Running the app

```bash
streamlit run app.py
```

---

## Features

- **Mood-based playlists** — Songs are automatically sorted into Hype, Chill, or Mixed tabs based on energy and genre
- **Configurable mood profile** — Adjust hype/chill energy thresholds and set a favorite genre
- **Add songs** — Add new songs with title, artist, genre, energy, and tags (with duplicate detection)
- **Search** — Filter any playlist by artist name
- **Lucky pick** — Get a random song from any, hype, or chill pools
- **Stats** — View total counts, hype ratio, average energy, and most common artist
- **History** — Track and review past lucky picks by mood

---

## Project structure

The project follows a clean separation between **presentation** and **business logic**, so the
UI and the rules that drive it can change independently.

```
playlistchaos/
├── app.py              # Streamlit UI layer — rendering and user interaction
├── playlist_logic.py   # Pure logic layer — classification, stats, search (no UI imports)
├── requirements.txt    # Python dependencies
└── README.md
```

### `playlist_logic.py` — the logic layer

Pure functions with no Streamlit dependency. Everything here is deterministic given its inputs
(aside from the intentional randomness in `lucky_pick`), which makes the module easy to read,
reason about, and unit-test in isolation.

| Function | Responsibility |
| --- | --- |
| `clean_string(value, lowercase=False)` | Strip whitespace and optionally lowercase a string |
| `normalize_song(raw)` | Coerce a raw song dict into a consistent set of keys and types |
| `classify_song(song, profile)` | Assign a mood label (`Hype` / `Chill` / `Mixed`) from energy, genre, and profile thresholds |
| `build_playlists(songs, profile)` | Classify every song and group the results into a playlist map |
| `merge_playlists(a, b)` | Combine two playlist maps into a new one without mutating either input |
| `compute_playlist_stats(playlists)` | Derive totals, per-mood counts, hype ratio, average energy, and top artist |
| `most_common_artist(songs)` | Return the most frequent artist and their song count |
| `search_songs(songs, query, field)` | Filter songs by case-insensitive substring match on a field |
| `lucky_pick(playlists, mode)` | Return a random song from the `any` / `hype` / `chill` pool |
| `random_choice_or_none(songs)` | Return a random element, or `None` for an empty list |
| `history_summary(history)` | Tally past picks by mood |

**Types & constants:** `Song` (dict alias for a song), `PlaylistMap` (label → list of songs),
`DEFAULT_PROFILE` (starting mood profile).

### `app.py` — the UI layer

Streamlit rendering and session-state wiring. Each function renders one cohesive piece of the
interface, and `main()` composes them top to bottom.

| Function | Responsibility |
| --- | --- |
| `init_state()` | Seed session state (songs, profile, history) on first load |
| `default_songs()` | Provide the built-in catalog of 22 preset songs |
| `profile_sidebar()` | Render the mood-profile controls (name, thresholds, favorite genre, mixed toggle) |
| `add_song_sidebar()` | Render the add-song form with validation, duplicate detection, and feedback |
| `playlist_tabs(playlists)` | Render Hype / Chill / (optional) Mixed playlists as tabs |
| `render_playlist(label, songs)` | Render one playlist tab with search-by-artist filtering |
| `format_song(song, detailed=False)` | Shared helper that turns a song dict into a display string |
| `lucky_section(playlists)` | Render the lucky-pick control and append the result to history |
| `stats_section(playlists)` | Display playlist statistics as metrics |
| `history_section()` | Show the pick-history summary and an optional full list |
| `clear_controls()` | Render the reset/clear buttons |
| `main()` | Application entry point that wires the layers together |

**Constant:** `GENRE_OPTIONS` — the single source of truth for genre choices, shared by both
dropdowns.

---

## What was fixed

The original starter code "ran" but behaved unpredictably. The work below restored correct
behavior and then tidied the structure, in two passes.

### Behavioral bug fixes

| Area | Before | After |
| --- | --- | --- |
| Search | `value in q` matched the field *inside* the query, so "Qu" never found "Queen" | `q in value` — the query is matched inside the field |
| Hype ratio | `total = len(hype)`, so the ratio was always `1.00` | `total` counts all songs, giving a real ratio |
| Average energy | Summed energy over Hype songs only, divided by all songs | Sums over all songs |
| Empty lucky pick | `random.choice([])` raised `IndexError` | Guarded — returns `None` on an empty pool |
| Chill keywords | Matched against the song *title* instead of genre | Matched against genre, consistent with Hype |
| "Any" lucky pick | Pooled only Hype + Chill | Includes Mixed as well |
| `merge_playlists` | Reused the input list, so `.extend()` mutated the original | Copies first — inputs are never mutated |
| Add-song UX | No duplicate check and no confirmation | Detects duplicates and shows success/warning messages |

### Readability & structure refactors

- **Unified normalization** — three near-identical `normalize_title` / `normalize_artist` /
  `normalize_genre` helpers collapsed into one `clean_string(value, lowercase=False)`.
- **Single source of truth for genres** — the duplicated genre list became the `GENRE_OPTIONS`
  constant used by both dropdowns.
- **Removed dead code** — dropped the no-op `merge_playlists(base_playlists, {})` call.
- **Shared formatting** — extracted `format_song()` to replace song-display formatting that was
  duplicated across three views.
- **Clearer logic** — `compute_playlist_stats` now derives its total from a single song list,
  `lucky_pick` dispatches through a small mode→label map, `history_summary` normalizes unknown
  moods up front, and the misleading no-op sidebar `columns()` scaffold was removed.
- **Import hygiene** — moved `import random` to module scope and removed two stray, unused imports
  (`from turtle import mode`, `import profile`) that had slipped in via autocomplete.

---

## Reflection

The central concept in this exercise was separating correct behavior from clean structure:
code that runs without crashing is not the same as code that is correct, and code that is correct
is not automatically readable. Students were most likely to get stuck on the silent logic bugs, 
the ones that produced a plausible-looking screen rather than an error. The hype ratio that always
read `1.00` and the backwards substring search are easy to scroll past, because nothing throws and
the numbers look real until you check them against the data by hand. AI assistance was genuinely
helpful for the mechanical, well-scoped cleanups, unifying the three normalize functions, pulling
out the shared `format_song` helper, and explaining why `merge_playlists` was mutating its input.
It was potentially misleading, it can "fix" a symptom while preserving the
underlying flawed assumption, so a green run can feel like proof of correctness when it isn't. The
way I would guide a student is not to point at the broken line but to ask them to predict a value
first,  "with these 22 songs, what *should* the hype ratio be, and what does the app actually
show?", so the gap between expectation and output leads them to the bug themselves.
