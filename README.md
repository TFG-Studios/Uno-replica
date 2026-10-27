# Uno Replica

A clean-room, browser-based UNO-inspired prototype built from scratch in HTML, CSS, and JavaScript.

What this project includes:
- local 2–4 player turn-based gameplay
- standard UNO deck generation and special cards
- playable rules engine for draw, skip, reverse, draw-two, wild, and wild-four
- minimal, non-AI-slop interface with classic card-table styling
- dedicated assets folder for custom card art overrides

## Design direction

- No chaotic, over-styled AI-generated UX.
- Clean, readable, classic gaming table layout.
- Card art can stay as elegant placeholders by default.
- If you want more faithful original art, drop replacement images into `assets/cards/` and the app will use them automatically when present.

## Run locally

Open `index.html` in a browser, or serve the folder with a simple local web server:

```bash
python3 -m http.server 8000
```

Then visit `http://localhost:8000`.

## Asset override

Place your own card images in `assets/cards/` using the same naming convention as the generated card IDs, or edit the card renderer in `app.js`.

See `assets/cards/README.md` for details.

## Notes

This is a clean-room implementation inspired by the UNO rule set and feel. It does not copy official proprietary assets or source code.
