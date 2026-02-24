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

## How the code is organized

### `app.py`

The Streamlit user interface.

- `init_state()` — Initializes session state (songs, profile, history)
- `default_songs()` — Returns the built-in list of 22 preset songs
- `profile_sidebar()` — Renders mood profile controls (name, energy thresholds, favorite genre, include mixed toggle)
- `add_song_sidebar()` — Renders the add-song form with duplicate detection and success/warning feedback
- `playlist_tabs(playlists)` — Renders Hype, Chill, and optionally Mixed playlists as tabs
- `format_song(song, detailed=False)` — Formats a song dict into a display string (shared helper used across views)
- `render_playlist(label, songs)` — Renders a single playlist tab with search-by-artist filtering
- `lucky_section(playlists)` — Renders the lucky pick controls and adds picks to history
- `stats_section(playlists)` — Displays playlist statistics (counts, hype ratio, average energy, top artist)
- `history_section()` — Shows pick history summary and optional full history
- `clear_controls()` — Renders reset and clear buttons in the sidebar
- `main()` — Entry point that wires everything together
- `GENRE_OPTIONS` — Shared list of genre choices used by both the profile and add-song dropdowns

### `playlist_logic.py`

The logic behind the app.

- `clean_string(value, lowercase=False)` — Strips whitespace and optionally lowercases a string
- `normalize_song(raw)` — Returns a normalized song dict with consistent keys and types
- `classify_song(song, profile)` — Returns a mood label ("Hype", "Chill", or "Mixed") based on energy, genre, and profile settings
- `build_playlists(songs, profile)` — Groups songs into Hype/Chill/Mixed playlists by classifying each song
- `merge_playlists(a, b)` — Merges two playlist maps into a new map (shallow copy, no mutation)
- `compute_playlist_stats(playlists)` — Computes total count, per-category counts, hype ratio, average energy, and top artist
- `most_common_artist(songs)` — Returns the most frequently occurring artist and their count
- `search_songs(songs, query, field)` — Filters songs by substring match on a given field
- `lucky_pick(playlists, mode)` — Picks a random song from the specified mode (any/hype/chill)
- `random_choice_or_none(songs)` — Returns a random song or None if the list is empty
- `history_summary(history)` — Returns a count of Hype/Chill/Mixed picks in the history
- `Song` — Dict type alias for song data
- `PlaylistMap` — Dict mapping playlist labels to lists of songs
- `DEFAULT_PROFILE` — Default mood profile settings
