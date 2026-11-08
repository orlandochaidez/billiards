# Rack & Roll: 8-ball pool for the browser

A mobile-first 8-ball pool game. Portrait table, drag-to-aim, pull-back cue power,
spin, real ball physics, a CPU opponent with three levels, pass-and-play for two,
and a practice table. Everything is drawn and synthesised in code: no images, no
audio files, no libraries.

## Run
No build step and no dependencies. Open `index.html`, or serve the folder statically:

```
python3 -m http.server 8000   # then open http://localhost:8000/
```

All paths are relative, so the folder can be pushed as-is to a GitHub repo and served
with GitHub Pages (root or any subfolder).

## How to play
- **Aim:** drag anywhere on the table and the aim line swings around the cue ball.
  Tap a ball to aim straight at it. On desktop the mouse wheel and arrow keys fine-tune too.
- **Fine aim:** slide the ridged wheel on the right for tiny adjustments.
- **Shoot:** pull the cue on the left down and let go. Push it back to the top to cancel.
- **Spin:** tap the cue-ball button and touch where the tip should strike
  (follow, draw, left/right english).
- **Ball in hand:** after a foul, drag the cue ball anywhere (behind the head string after a
  scratch on the break).

## Rules (standard 8-ball, kept simple)
- 15 balls racked with the 8 in the middle and one solid and one stripe on the back corners.
- Table is open after the break; the first legally potted ball sets your group.
- Fouls (scratch, no ball hit, wrong ball first, 8 first on an open table, no rail after
  contact) give the opponent ball in hand.
- Potting the 8 before clearing your group, or scratching/fouling on the 8, loses.
  Clearing your group and then sinking the 8 in any pocket wins (no call pocket).
- An 8 potted on the break is spotted back on the foot spot.

## Modes
- **vs CPU** (Easy / Medium / Hard): the CPU builds candidate shots (each legal ball into
  each pocket at several speeds and spins, plus safe hits and kicks), plays every candidate
  out with the same physics on a cloned table, scores the result with the same rules plus
  next-shot position (Medium/Hard) and safety (Hard), then shoots the best with aim and
  power error that grows as the level drops. Thinking runs a few milliseconds per frame so
  the game never freezes. With ball in hand it also chooses where to place the cue ball.
- **2 Players**: pass-and-play on one device.
- **Practice**: no opponent, clear the table; fouls just give you ball in hand.

Wins and losses vs the CPU per level, the last level, and mute are kept in `localStorage`.

## Files
- `index.html`: markup, HUD, menus
- `style.css`: UI styling, safe areas, small-screen tweaks
- `js/physics.js`: table geometry (cushions, pocket jaws, mouths), fixed-step sub-stepped
  physics, ray casts. No DOM, also runs in Node.
- `js/rules.js`: 8-ball rules (fouls, groups, win/loss), pure functions
- `js/ai.js`: CPU opponent (generator, time-sliced)
- `js/audio.js`: WebAudio sound effects, all synthesised (strike, clack, cushion, pocket, foul, win)
- `js/render.js`: cached table art; per-pixel rolling ball sprites with foreshortened numbers
- `js/game.js`: loop, layout, input, turns, HUD, test hook

## Physics notes
- Fixed 120 Hz step with adaptive sub-steps (up to 16) so a ball never moves more than a
  third of its radius per sub-step, so no tunnelling at full power.
- Each ball tracks velocity and a rolling "surface velocity". Sliding friction converts slide
  into natural roll (the 5/7 rule falls out), which gives real stun, follow and draw.
- Equal-mass ball collisions with restitution; cushions with restitution, roll loss and side
  spin throw; pockets have angled jaws, so balls rattle out when they hit a jaw.

## URL options
- `?mode=cpu|pvp|practice`: skip the title screen
- `?mode=cpu&level=easy|medium|hard`: choose the CPU level

## Test hook
`window.RackRoll` exposes `start(mode, level, breaker)`, `setup(spec, opts)`, `aimAt(id, pocket)`,
`shoot(angle, power, spinX, spinY)`, `settle()`, `placeCue(x, y)`, `forceTurn(n)`,
`setSpeed(x)` and `info()` for automated tests. It goes through the same code paths as play.
