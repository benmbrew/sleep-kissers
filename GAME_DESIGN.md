# Sleep Kissers — Game Design Doc

## Overview
A 2D browser game. One player controls a sleepwalking "kisser" and tries to catch (kiss) three awake runners. Open `index.html` directly in any browser — no build step required.

---

## Files
| File | Purpose |
|---|---|
| `index.html` | Entire game (HTML + Canvas + JS, single file) |
| `headshot1.png` | Player-selectable character 1 (Bri) |
| `headshot2.png` | Player-selectable character 2 (Benzi) |
| `headshot3.png` | Player-selectable character 3 (Gabey) |
| `headshot4.png` | Player-selectable character 4 (Jonny Boy) |

---

## State Machine
```
loading → login → difficulty → select → dozing → countdown → playing → win | gameover
```

| State | What happens |
|---|---|
| `loading` | Images load; shows "Loading…" |
| `login` | HTML overlay — player enters name; shows current top scores per difficulty from Firestore |
| `difficulty` | 3 cards (Easy / Medium / Hard) with top score shown on each card |
| `select` | All 4 headshots shown; click/tap one to be the kisser |
| `dozing` | Chosen face sways, purple glow builds, Zzz appear — "falling asleep" animation (~100 frames); obstacles generated here |
| `countdown` | Full board visible; kisser sleeping in center, 3 runners at corners; big 3 → 2 → 1 → GO! |
| `playing` | Player sleepwalks after runners; kiss all 3 to win; 45-second time limit |
| `win` | Dark overlay — shows name, score, time, full top-10 leaderboard; Play Again + Watch Replay buttons |
| `gameover` | Big "Game Over 😟 / Daddy woke up!" overlay when 45s expires; 0 pts; Play Again + Watch Replay buttons |
| `replay` | Plays back the recorded game session frame-by-frame with progress bar |

---

## Characters

### Kisser (player)
- Controlled with **WASD** or **Arrow Keys** (desktop) or **virtual joystick** (mobile — dedicated zone below canvas)
- Speed varies by difficulty: Easy 3.2 · Medium 2.8 · Hard 2.4
- Visual: blue/purple glow ring, sleep-walk sway animation, floating Zzz particles, 👄 red lips emoji on face

### Runners (3 awake characters)
- Speed varies by difficulty: Easy 2.6 · Medium 3.6 · Hard 4.6
- Visual: yellow ring, no sway, no Zzz
- **Flee AI**: steer away from kisser within flee radius (Easy 130px · Medium 250px · Hard 360px)
- Wall repulsion + runner separation forces keep them from bunching or wall-hugging
- After being kissed: drift to a stop, pink overlay + 💋 emoji
- Speech bubbles appear with commentary as the game progresses

---

## Scoring
- Formula: `max(50, floor((5000 − seconds × 20) × diffMult))`
- diffMult: Easy = 1 · Medium = 1.5 · Hard = 2.5
- Fast + hard difficulty = highest scores
- Game over (45s timeout) = 0 pts

---

## Difficulty Settings
| Difficulty | Player Speed | Runner Speed | Flee Radius | Wall Margin |
|---|---|---|---|---|
| Easy | 3.2 | 2.6 | 130 | 90 |
| Medium | 2.8 | 3.6 | 250 | 140 |
| Hard | 2.4 | 4.6 | 360 | 170 |

---

## Obstacles
- 4 obstacles generated randomly at the start of each game (during `dozing` → `countdown` transition)
- One obstacle per quadrant (top-left, top-right, bottom-left, bottom-right) for even spread
- Each obstacle is randomly horizontal (110–180px × 22–34px) or vertical (22–34px × 90–160px)
- Positioned randomly within its quadrant zone, with a retry loop to avoid character starting positions
- Drawn as dark purple rounded rectangles with a purple glow border
- Both player and runners collide with obstacles (circle-vs-AABB); runners also get velocity deflected

---

## Replay System
- Every playing frame, a snapshot is pushed to `replayFrames[]`: player pos, runner pos/kiss state, hearts, zzzs, score, elapsed time
- Capped at 2,800 frames (~47s) to bound memory (~1MB max)
- "Watch Replay 📽️" button appears after win or game over (next to Play Again)
- Replay loops through snapshots at native framerate using the same draw helpers
- Pink progress bar + timestamp + kiss count shown at bottom during replay
- "✕ Close Replay" returns to the win/gameover screen

---

## Visual Details

### Headshot rendering
Images are drawn with **object-fit: cover** math — the image's natural aspect ratio is preserved, scaled so it fills the circle without stretching, centered.

### Particles
- **Zzz**: float up from the kisser's head during `playing` (every 55 frames)
- **Hearts**: burst from runners when kissed (7 per kiss)

---

## Firebase / Firestore
- Project: `sleep-kissers` (console.firebase.google.com)
- Firestore structure: `leaderboard/{Easy|Medium|Hard}/entries/{auto-id}`
- Each entry: `{ name, score, time, ts }`
- Rules: test mode (check expiry periodically)
- Config is hardcoded in `index.html`

---

## Mobile
- `<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">`
- Canvas scales via CSS `max-width: 100%; height: auto`
- Virtual joystick lives in a dedicated `#joyZone` div below the canvas (shown during countdown + playing)
- Joystick base is fixed at center of its own `#joyCanvas` so the thumb never covers the play area
- All touch events use `passive: false` + `touch-action: none` to prevent scroll

---

## Live URL
benmbrew.github.io/sleep-kissers (GitHub Pages)
