# 🎹 Piano Pals

A friendly first-piano app for little kids. It runs entirely from **one HTML file** —
no server, no internet, no accounts, no ads, no purchases.

Made for two beginner pianists:

| | |
|---|---|
| **Maya** (6) | reads a little — whole tunes, steady beat |
| **Kai** (4) | can't read yet — picture-first screens, short tunes, notes that wait |

---

## How to run it

**Easiest:** open `index.html` in any browser (double-click it).

**On an iPad:** put `index.html` in the Files app (or on a web host) and open it in
Safari. Then *Share → Add to Home Screen* for a full-screen, app-like experience.
Everything works offline once the page has loaded.

**From a tiny local server** (handy for testing on a phone over Wi-Fi):

```bash
cd kids-piano
python3 -m http.server 8000
# then open http://<your-computer-ip>:8000/
```

There is nothing to install and nothing to build. `index.html` is the whole app.

---

## What's inside

| Screen | What it does |
|---|---|
| **Who's playing?** | Big avatar buttons, one per kid. Add or edit players. |
| **Home** | Four giant tiles: Songs · Lessons · Free Play · My Stars |
| **Songs** | 12 tunes, each in a 🌱 *First notes* (6–10 notes) and a 🌳 *Whole song* version. Falling-note game with finger numbers, hand colours and a 5-star score. |
| **Lessons** | 12 short interactive lessons, each ending in a star. |
| **Free Play** | Two labelled octaves, a "guess the note" ear game, and record/playback. |
| **My Stars** | Stars, songs finished and a garden that fills up as you earn. |
| **⚙️ Grown-up corner** | Sound on/off, unlock all songs, reset one kid's progress, add/edit players. |

### Two play modes

* **🐢 Wait for me** — the next note falls and *stops* on the hit line until the
  right key is pressed. Wrong keys wobble and give a soft thud, never a penalty.
  This is the default for Kai and for every beginner.
* **🐇 Keep going** — notes scroll at a steady tempo. A press counts if it lands
  within about half a second of the note reaching the line.

### Stars

Accuracy → stars, and finishing always earns at least one:

| Accuracy | Stars |
|---|---|
| 95 %+ | ★★★★★ |
| 85 %+ | ★★★★☆ |
| 70 %+ | ★★★☆☆ |
| 50 %+ | ★★☆☆☆ |
| finished | ★☆☆☆☆ |

A song unlocks the next one at ★☆☆☆☆ or better. Lessons unlock in order.
A grown-up can unlock everything at once from the ⚙️ panel.

---

## How songs are defined

Songs live in one array near the top of the `<script>` block, written as compact
text so they are easy to edit or extend:

```js
{
  id: "twinkle",
  title: "Twinkle, Twinkle, Little Star",
  em: "⭐",                 // sticker shown on the song card
  col: "#FFC83D",           // card colour
  bpm: 92,                  // tempo for 🐇 "Keep going" mode
  full:  parseMelody("C4:1 C4:1 G4:1 G4:1 A4:1 A4:1 G4:2 ..."),
  first: parseMelody("C4:1 C4:1 G4:1 G4:1 A4:1 A4:1 G4:2")
}
```

### The `parseMelody` notation

Each token is `NOTE:BEATS`, separated by spaces. A space is just for readability —
timing comes from the beat values, not from the spaces.

| Token | Meaning |
|---|---|
| `C4:1` | middle C, one beat |
| `G4:0.5` | G above middle C, half a beat |
| `C5:2` | the C an octave above middle C, two beats |
| `E4:1.5` | dotted quarter |
| `L-C3:0` | **left-hand bass note** — played *by the app*, together with the next melody note |

Octave numbers follow scientific pitch: `C4` is middle C, `A4` is 440 Hz.
Only the natural notes are used (`C D E F G A B`), so everything sits on white
keys — no sharps to hunt for.

Left-hand notes are **accompaniment**. They are drawn in the cool blue colour,
sound automatically, and never block the child's progress. If adding the bass
would make the on-screen keyboard too wide for the device, the app quietly drops
them and plays the melody alone.

### Adding a song

1. Copy any entry in the `SONGS` array.
2. Give it a new `id`, a title, an emoji and a colour.
3. Write `full` (the whole tune) and `first` (6–10 notes a beginner can manage).
4. Nothing else to change — the song list, locking order and star tracking all
   pick it up automatically.

### Fingering

`assignFingers()` works out the 1–5 numbers shown on each note block. If a tune
fits inside one five-finger position it numbers from the lowest note (so
`E D C` reads **3 2 1**); otherwise each note is numbered from a small local
window of its neighbours so the numbers stay steady and singable. When a note
falls outside the hand position the number clamps to 1 or 5 — the hand simply
shifts, which is what a primer book does.

---

## Sound

Everything is synthesised live with the **Web Audio API**; there are no audio
files. Each note is three detuned oscillators (triangle + triangle + sine, with a
quiet octave partial) through a low-pass filter, with a 7 ms attack and a long
exponential decay, plus a short shared feedback delay for a little room. Several
keys can sound at once. Browsers require a tap before audio starts, so the first
tap anywhere switches the engine on.

---

## Saved progress

Progress is kept in `localStorage` under the key `pianoPals.v1` — one record per
kid, so stars, completed lessons and settings never mix. Nothing ever leaves the
device.

Clearing browser data erases progress; the ⚙️ panel also has a per-kid reset.

---

## Layout & devices

Built for an iPad in Safari, and fine on a laptop or a phone:

* Pointer Events for touch and mouse; no hover-only controls.
* `viewport-fit=cover` plus safe-area insets, and double-tap zoom disabled.
* Tap targets are large and outlined; text is sized in `vh` so it scales.
* Checked with no overflow at **1024×768**, **1280×800**, **390×844** and **820×1180**.
* On narrow phones the free-play keyboard keeps a minimum key width and scrolls
  sideways rather than shrinking into slivers.
* `prefers-reduced-motion` is respected.

---

## Project layout

```
kids-piano/
├── index.html        ← the entire app (HTML + CSS + JS, no dependencies)
├── README.md         ← this file
└── tests/            ← Playwright smoke tests used while building
    ├── full.py           all screens, both modes, persistence, kid editing
    ├── play_all.py       plays all 12 songs in both versions to the end
    ├── lesson_all.py     completes every step of all 12 lessons
    ├── sizes.py          layout checks + screenshots at four viewports
    └── timed.py          "Keep going" scoring against the real note timeline
```

Run a test with:

```bash
/workspace/.pwvenv/bin/python tests/full.py
```

(They use the system Chrome at `/opt/google/chrome/chrome`.)

---

## Notes

* All tunes are traditional / public domain.
* Branding, palette and layout are original to Piano Pals.
