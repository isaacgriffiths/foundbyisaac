# Handoff: The Mansion

Brief for whoever builds the mansion. Written at commit `d9b0d15`, `index.html` at 8657 lines.

---

## 1. What Isaac asked for

Verbatim, across two messages:

> "now we have prestige we have a million point buy that gives you access to a house
> where you go and its a nice house where you can play casino games and eat food ect
> ect with differnt benifits but no stalls"

So:

- A **big one-off purchase** unlocks it.
- It is **a place you go**, like the Warehouse — not a panel.
- **A nice house.** Aspirational, the opposite of a clearance sale.
- **Casino games** you can play.
- **Food** you can eat.
- Both give **benefits** ("different benefits" — so more than one kind).
- **No stalls.** Nothing to haggle over, nothing to buy as stock.

### The one open question — ASK BEFORE BUILDING

"A million point buy" is ambiguous and I did not resolve it. Isaac was asked twice and
has not answered.

- **£1,000,000 cash** — my reading. It matches the top of `PRO_GATES` (line 5155), and
  it is a real target at the late game.
- **Contacts** — the prestige currency (`contacts`, line ~5171). But you earn ~4 per
  prestige, so "a million" cannot mean contacts. A contacts price would be more like 15.

**Ask him.** If he does not answer, build it as £1,000,000 cash and say so plainly. If
you go cash, note it sits above the current top prestige gate, so it is the last thing
in the game — that is probably right for a mansion, but flag it.

---

## 2. The single most important thing to get right

Every visual bug in the estate build came from one thing, and it cost four rounds of
back-and-forth. **Learn the depth convention before you draw anything.**

The scene is 2D side-on. Vertical position in world units encodes **depth**, not height:

| world `bottom` | what lives there |
|---|---|
| `34` | the player's feet — the line you actually walk on |
| `60` | the pitches / stalls stand here |
| `0-92` | `#ground`, the walkable strip. Anything with its **base inside this band reads as standing on the floor** |
| `92` | the **back wall / boundary** line. Fences, walls, the warehouse back wall |
| `92+` | on the back wall, or hanging off it |

The trap: putting an object's base at `92` or above makes it a *boundary* object. If it
is a big free-standing thing it will look like it is floating, because there is no floor
drawn behind it. The pool was at `96` and read as hovering in the sky through three
attempted fixes; moving its base to `54` fixed it instantly (commit `d9b0d15`).

**Rule: free-standing objects get a base between ~40 and ~80. Only walls, fences and
things fixed to them go at 92+.** Add a soft shadow ellipse under anything large.

---

## 3. Which pattern to copy

There are two existing "places you go". Copy the **Warehouse**, not the Estate.

### The Warehouse (copy this)

An interior you enter from anywhere, with walk-up stations. This is exactly the mansion's
shape: bounded, no stalls, stations you interact with.

| thing | where |
|---|---|
| `enterWh()` / `exitWh()` | 6042 / 6053 |
| `WH_MIN = 140, WH_MAX = 3120, WH_SPAWN = 585` | 6041 |
| `buildWh()` — refreshes all the station signs | 6014 |
| station markup — `.whstation` divs inside `#view` | search `id="whVan"` |
| station wiring — array of `[id, openFn]` | search `whStations.push` |
| `dressWarehouse()` — builds the back wall and lights | search `dressWarehouse` |
| `#whWall` / `#whLight` parallax layers | markup near `id="whWall"` |
| body class `in-wh` gates all the CSS | search `body.in-wh` |

Existing stations, all following the same shape: `whVan`, `whLaptop`, `whPack`, `whShop`,
`whDex`, `whWard`. Add mansion ones the same way.

### The Estate (do NOT copy)

`buildEstate()` at 4243. It is a *venue* — a place you drive to, with pitches to haggle
at. The mansion has no stalls, so it is not a venue. Do not add it to `VENUES` (5810).
Its one useful lesson is the bounded-walk clamp, in the main loop:

```js
if (inWh) px = Math.max(WH_MIN, Math.min(WH_MAX, px));
else if (estateNow) px = Math.max(EST_MIN, Math.min(EST_MAX, px));
```

Add an `inManor` branch there. Also skip the stall-wrapping block below it (it is already
guarded by `!inWh && !estateNow`).

---

## 4. Save format — read this before touching `save()`

`SAVE_KEY = "fbi-flip-v1"` (5252). `save()` at 5266, loader immediately after.

**Keys already taken.** Do not reuse any of these:

```
w p i pk rp vn tp t e v b d l g ct bk lk q li or dx ft of ok av pf nl vs pl pa ll la iw x wx
```

Free two-letter keys that read sensibly for this: `mn` (manor owned), `mf` (food buff),
`mc` (casino state), `mb` (mansion upgrades bag).

**Hard rules, from `memory/foundbyisaac-workflow.md`:**

- New fields must be added to **both** `save()` and the loader. Miss the loader and it
  silently never persists.
- New entries in `var items` go at the **END** of the array. Saved inventories store
  array indexes, so the order is load-bearing. Same for `PACK_POOL`.
- Prefer one nested object over several new top-level keys (see how `g:` bags the four
  newer gear levels).

---

## 5. Existing systems to hook the benefits into

Do not invent a parallel buff system. There is already one.

**`perks()` at 5116** is the single place the haggle asks "what has the player got going
for them". It returns:

```js
{ floorMul, pat, cashMul, flawMul, hint }
```

- `floorMul` — multiplies the seller's floor price. Lower is better.
- `pat` — extra offers before a seller walks off.
- `cashMul` — what "cash in hand" knocks off.
- `flawMul` — what pointing out a flaw knocks off.
- `hint` — seller lets slip what it is really worth.

A meal buff should fold into this and nothing else. It is read fresh on every restock,
so a buff takes effect at the next pitch with no extra wiring.

**Other levers already in place:**

- `proMult()` (5154) — multiplies every sale. `BOOK.margin` already feeds it.
- `BOOK` (5172) — permanent, survives prestige. If a mansion benefit should be forever,
  it belongs here.
- `STAFF` (5194) — the Barry/Shazza pattern for anything that works on a timer.
- `walkSpeed()`, `gateFee()`, `bagCap()`, `maxListings()` — all single-source functions,
  all cheap to hook.

**Suggested split**, since Isaac said "different benefits":

- **Food** — temporary, expires. Fold into `perks()`. A big meal = more seller patience;
  a coffee = sharper haggling; etc.
- **Casino** — pure cash swing, no buff. Stakes real money.
- Consider one **permanent** mansion perk so the million feels bought, not rented.

---

## 6. Mini-games: reuse the harness, do not build a new one

`playMini(game, done, stakes)` at 6757. Already used by the packing bench and the
listing flow, and it handles the timer, the skip button, mobile deferral and the panel.

A game object (see `gFold` at 6786 for the simplest real example):

```js
{
  title: "🃏 Blackjack",       // panel heading
  hint:  "Tap to twist",        // one-line instruction
  cls:   "blackjack",           // CSS class on the stage host
  defer: true,                  // optional: hold the clock until first input
  start: function (host, done, timeLeft) {
    // build your UI into host, call done(score) where score is 0..1
  }
}
```

`done(score)` with `score` 0-1. `MINI_MS` is the clock. `stakes` is the strip of text
explaining what is being played for.

Casino games fit this shape well. **But note:** the existing games all pay out via a
score multiplier on a fixed reward. A casino needs to be able to *lose* the stake, which
the harness does not do for you — take the stake up front, then pay back on the result.

---

## 7. House rules — non-negotiable

From `memory/foundbyisaac-workflow.md`. These are Isaac's, not suggestions.

1. **No em dashes anywhere in the file.** Comments use `-`, user-facing text uses `·`.
   Check with a grep for `\u2014` before every commit.
2. **Push every finished chunk to `main`.** He reviews on the live site, not locally.
3. **Then poll until it is actually live**, and only then say it is:
   ```
   gh api "repos/isaacgriffiths/foundbyisaac/actions/runs?per_page=1" \
     --jq '.workflow_runs[0] | .head_sha + " " + .status + " " + (.conclusion // "-")'
   ```
   Wait for `<your-sha> completed success`. Note the `pages/builds/latest` endpoint went
   stale during this session and reported an old commit as current — use the Actions run,
   not that endpoint.
4. **Syntax-check after every edit.** Extract the inline `<script>` and `new Function()`
   it. There is a ready-made checker at
   `scratchpad/check.js` — it does the syntax check and the em-dash grep in one go.
5. **`#world` scales up to 2.5x.** UI inside it must counter-scale with
   `transform: scale(var(--iz))`, and be positioned with `bottom:`, never `top:`.

### Verifying it

Local harness, not committed:

```bash
python -m http.server 3001 --bind 127.0.0.1
NODE_PATH="C:/Users/ISAAC/AppData/Local/npm-cache/_npx/9833c18b2d85bc59/node_modules" node yourtest.js
```

Playwright with `chromium.launch({ channel: "chrome" })`. **The sandbox has no external
DNS — test only against 127.0.0.1.**

Seed a save with `context.addInitScript`, and guard it so it does not re-seed on reload:

```js
await ctx.addInitScript(() => {
  if (localStorage.getItem("seeded")) return;
  localStorage.setItem("seeded", "1");
  localStorage.setItem("fbi-flip-v1", JSON.stringify({ /* ... */ }));
});
```

Without that guard the init script re-runs on every navigation and silently undoes
anything you are trying to test across a reload. It cost me a false "reset is broken"
result this session.

There is a venue regression script at `scratchpad/venues.js` that loads all five venues
and reports scenery counts plus console errors. Run it after any scene change — the
mansion should not disturb any of them.

---

## 8. Watch out for

- **`hardReset()` and the `wiping` flag.** `save()` early-returns when `wiping` is true,
  because a reload fires `pagehide` which fires `save()`, which used to write the whole
  game back over the key that had just been deleted. If you add save-on-event handlers,
  respect that flag.
- **Position persistence.** `notePlace()` writes `px` on stop, plus a 1200ms tick, plus
  `pagehide`/`beforeunload`/`blur`. If the mansion is a new scene, it needs its own
  saved x (like `wx` for the warehouse) or you will come back to the wrong place.
- **`buildWh()` is called from `updBadges()`**, which runs on every wallet change. If you
  add a mansion sign that reads the wallet, it will refresh for free. Do not add a
  separate polling loop for it.
- **`applyVenue()` toggles body classes and rebuilds scenery.** If the mansion is
  entered from the warehouse it does not touch venues at all, which is simpler. Prefer
  that.
- **Escape handling.** `openPanels` array near the top of the keydown handler is ordered
  — `dexPop` is first so it closes before the index behind it. Add mansion panels in the
  right order.

---

## 9. Suggested shape (not prescriptive)

If it helps, this is what I would have built:

- Unlock: a **Big Stuff** row in the Upgrades shop (`buildShop()` at 7647, `shopTab`
  `"big"`), £1,000,000, two-tap confirm like the existing `nolinks` and `pro` rows.
- Entry: a sixth warehouse station, `whManor` — "the car comes for you". Or a HUD button
  once owned.
- Scene: bounded interior, `MANOR_MIN`/`MANOR_MAX`, own back-wall layer. Parquet,
  chandeliers, panelling. It should feel expensive against the warehouse's strip lights.
- Stations, all `.whstation`-style walk-ups:
  - 🃏 **The card room** — casino, stakes cash, uses `playMini`.
  - 🍽️ **The dining room** — buy a meal, get a `perks()` buff for N boot runs.
  - 🥃 **The study** — the permanent perk, so the million buys something forever.
- No `.stall` elements anywhere in it. That is the one hard requirement he gave.

Nothing above is agreed with Isaac beyond the first paragraph of section 1. Check the
shape with him before building it out.
