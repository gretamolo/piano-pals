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
| **⚙️ Grown-up corner** | Sound on/off, **choose your piano (touch / MIDI / listen)**, mic sensitivity, test panel, unlock all songs, reset progress, add/edit players. |

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

## Playing on a real piano 🎹

Piano Pals can take its notes from an instrument instead of the screen. Open
**⚙️ Grown-up corner → My piano** and choose one of three sources:

| Source | What it needs | Good for |
|---|---|---|
| **👆 Touch screen** | nothing | tablets, laptops, the default |
| **🎹 MIDI piano** | a USB-MIDI cable and a Chromium browser | digital pianos |
| **👂 Listen to my piano** | a microphone and a quiet room | any acoustic piano |

The choice is remembered on that device, and each kid can also have their own
(Free Play on the screen for one, the real piano for the other).

### MIDI piano (digital pianos)

Plug the piano into the computer with its USB cable, switch it on, then pick
**MIDI piano**. The status line turns green with the piano's name and a device
list appears if you have more than one. **Connect / Reconnect** re-scans at any
time, and unplugging or plugging back in is picked up automatically.

* Every channel is accepted, so it works whether the piano transmits on
  channel 1 or 16.
* A note-on with velocity 0 is treated as a note-**off**, which is what most
  keyboards send for a key release.
* Presses register the instant the message arrives — the app never waits for its
  own sound, so there is no added latency.
* The on-screen keyboard stays visible and tappable as a backup, but it dims
  while the real piano is driving.

> **Important:** Web MIDI is a Chromium feature. It works in **Chrome and Edge
> on a computer** (and Chrome on Android). **Safari — on a Mac, an iPad or an
> iPhone — does not support Web MIDI at all**, in any version. On an iPad, use
> **Listen to my piano** instead; that is exactly why it exists.

### Listen to my piano (acoustic pianos)

Pick **Listen to my piano**, press **Start listening**, and allow the
microphone. Then play one note at a time. The app:

* measures the room level and ignores anything below a noise gate
  (the **Mic level** slider moves that gate);
* finds the pitch by normalised autocorrelation over a 4096-sample window,
  with parabolic interpolation for cents accuracy;
* requires the pitch to hold steady for ~70 ms before counting it, so a
  scrape or a cough can't score a note;
* maps it to the nearest equal-tempered key within ±50 cents;
* locks that note until it decays or changes, so one struck key is one hit.

The **Test my piano** panel shows the live level meter, the note name, how many
cents off it is and the frequency — a parent can watch it work before handing
the iPad over. **Mic level** adjusts sensitivity if quiet playing isn't picked
up (or if the room is noisy).

> **Use headphones, or leave the app sound off.** Listening mode hears the
> room, so if the app plays its own tone the microphone will hear that too.
> Piano Pals mutes its own output automatically when listening mode starts, and
> the setting is right there if you want to change it. A quiet room and one
> note at a time give by far the best results.

### Latency

Both real-instrument paths register on the *key press*, not on the app's audio.
Visual feedback (the note disappearing, the star counter) happens immediately;
nothing is scheduled behind the synth.

### If something doesn't work

| What you see | What to do |
|---|---|
| "This browser can't talk to MIDI pianos" | You're in Safari/Firefox-on-Mac. Use Chrome/Edge on a computer, or switch to **Listen to my piano**. |
| "No piano found" | Check the USB cable, switch the piano on, press **Reconnect**. Some pianos need *USB MIDI* mode selected in their own settings. |
| "MIDI access was blocked" | Allow MIDI for the page in the browser's site settings, then reload. |
| "Microphone permission was refused" | Allow the mic for the page, then press **Start listening** again. |
| Listening misses quiet notes | Raise **Mic level**, play a little louder, or move closer. |
| Listening fires on the wrong note | Lower **Mic level** a touch, turn the app sound off, and play one note at a time. |

Everything still works exactly as before with no piano and no microphone:
if neither is available the app simply stays on the on-screen keyboard.

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
    ├── timed.py          "Keep going" scoring against the real note timeline
    ├── midi.py           fake Web MIDI device: notes -> songs, free play, ear game
    ├── listen.py         fake microphone: pitch -> notes, plus denial handling
    ├── pitch.py          the detector's maths, on synthetic tones
    └── midi_stub.py      the MIDI / microphone stubs those tests inject
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
