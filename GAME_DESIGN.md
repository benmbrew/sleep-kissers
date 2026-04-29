# Sleep Kissers — Game Design Doc

## Overview
A 2D browser game. One player controls a sleepwalking "kisser" and tries to catch (kiss) three awake runners. Open `index.html` directly in any browser — no build step required.

---

## Files
| File | Purpose |
|---|---|
| `index.html` | Entire game (HTML + Canvas + JS, single file) |
| `headshot1.png` | Player-selectable character 1 |
| `headshot2.png` | Player-selectable character 2 |
| `headshot3.png` | Player-selectable character 3 |
| `headshot4.png` | Player-selectable character 4 |

---

## State Machine
```
loading → select → dozing → countdown → playing → win
```

| State | What happens |
|---|---|
| `loading` | Images load; shows "Loading…" |
| `select` | All 4 headshots shown; click one to be the kisser |
| `dozing` | Chosen face sways more and more, purple glow builds, Zzz appear — "falling asleep" animation (~100 frames) |
| `countdown` | Full board visible; kisser sleeping in center, 3 runners at corners; big 3 → 2 → 1 → GO! |
| `playing` | Player sleepwalks after runners; kiss all 3 to win |
| `win` | Overlay: "All kissed! 💋💋💋" |

---

## Characters

### Kisser (player)
- Controlled with **WASD** or **Arrow Keys**
- Speed: `2.6`
- Visual: blue/purple glow ring, sleep-walk sway animation, floating Zzz particles, **sleep mask** over eyes (dark purple mask with star decorations `✦`)

### Runners (3 awake characters)
- Speed: `3.4` (faster than the kisser — intentional challenge)
- Visual: yellow ring, no sway, no Zzz
- **Flee AI**: within 260px of the kisser they steer away; flee strength `0.85` (strong)
- After being kissed: drift to a stop, pink overlay + 💋 emoji

---

## Key Constants (top of `<script>`)
| Constant | Value | Effect |
|---|---|---|
| `PLAYER_SPEED` | 2.6 | Kisser movement speed |
| `RUNNER_SPEED` | 3.4 | Runner base speed |
| `FLEE_RADIUS` | 260 | Pixels at which runners start fleeing |
| `DOZING_TICKS` | 100 | Length of falling-asleep animation |
| `CD_PER_NUM` | 65 | Frames per countdown number (3, 2, 1) |

---

## Visual Details

### Headshot rendering
Images are drawn with **object-fit: cover** math — the image's natural aspect ratio is preserved, scaled so it fills the circle without stretching, centered.

### Sleep mask
Drawn on top of the kisser's headshot at all times (dozing, countdown, playing).  
- Dark purple rounded rectangle spanning both eye areas
- Strap lines extending to circle edge
- Two subtle eye-cup ovals
- `✦` star icons in each cup
- Glossy sheen highlight

### Particles
- **Zzz**: float up from the kisser's head during `playing` (every 55 frames)
- **Hearts**: burst from runners when kissed (7 per kiss)

---

## Possible Next Steps
- Active flee AI (runners pathfind away, not just steer)
- Timer / speed-run scoring
- Sound effects (snoring, kiss smack)
- Mobile touch controls
- More headshot slots
