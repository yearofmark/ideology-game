# ideology-game — Claude context

## Architecture
Single file: `index.html` (~3000 lines). Everything — HTML, CSS, JS — lives here. No build step.

**Deployed** via GitHub Pages (auto-deploy from `main`). A PostToolUse hook in `.claude/settings.json` auto-pushes after every `git commit`.

**Preview server**: port 3456 (configured in `.claude/launch.json`). Use `mcp__Claude_Preview__*` tools to take screenshots and interact. Load preview tools at session start with one ToolSearch:
```
select:mcp__Claude_Preview__preview_start,mcp__Claude_Preview__preview_screenshot,mcp__Claude_Preview__preview_eval,mcp__Claude_Preview__preview_click,mcp__Claude_Preview__preview_resize,mcp__Claude_Preview__preview_inspect,mcp__Claude_Preview__preview_list
```

## File navigation (critical for this single-file project)
Before editing, always run: `grep -n "^// ═══\|^const \|^function \|^let \|^var " index.html | head -80`
This gives a line-number map so you can `Read(offset=X, limit=N)` precisely.

Key section line ranges (approximate — grep to confirm):
- CSS: lines 1–344
- HTML body / overlays: lines 345–620
- Setup screen HTML: lines ~620–740
- Game screen HTML: lines ~740–820
- Constants (SOCIETIES, SP_FIT, INIT_STATES, CARD_POOL): lines ~820–1050
- Setup JS (initSetup, renderSetupPlayers, startGame): lines ~1050–1200
- Render functions (renderAll, renderStates, renderHand): lines ~1200–1600
- Card / turn logic (doPlayCard, aiTurn, doEndTurn): lines ~1600–2000
- Commentary / narration (showCommentary, aiTurnNarrated): lines ~2000–2350
- Utility functions (FLAGS, showStatePopup, etc.): lines ~2350–end

## Core globals
- `G` — entire game state (players, states, turn, phase, etc.)
- `gMode` — `'hot' | 'solo' | 'sim'`
- `SOCIETIES` — 5 ideologies: TCH, THE, PLU, STR, ANA
- `INIT_STATES` — 25 countries on a 5×5 grid; superpowers have `sp:true`
- `CARD_POOL` — all card definitions; power cards have `power:true`, globals have `global:true`
- `narrQueue[]`, `advanceNarr()` — simulation turn narration steps
- `commSkipped` — if true, drain narration queue without showing overlay
- `simPaused`, `simSpeed`, `SIM_DELAYS` — simulation controls

## Key functions
- `cur()` → current player object
- `calcVP(player)` → live VP count
- `buildDeck(soc)` → shuffle deck for a society
- `applyEffect(effect, pIdx, sid)` → apply card effect
- `computeAIOptions(p)` → sorted `{c, i, targetSid, score}` array
- `snapshotStates()` / `showStateDeltas(before)` → delta chip animations
- `startSPPick(players, onDone)` → starting position picker flow
- `finishStartGame(players)` → initialize G and start game

## Edit discipline
- **Always `grep -n` for the exact string before editing.** Never edit from memory.
- CSS and JS are minified/dense — one wrong character makes the string not found.
- After editing JS, verify with: `node -e "new Function(require('fs').readFileSync('index.html','utf8').match(/<script>([\s\S]*)<\/script>/)[1]); console.log('OK')"`

## Commit and deploy
```bash
git add index.html && git commit -m "message"
# Auto-push hook fires automatically after commit
```
Do not `git push` manually — the hook handles it.

## What NOT to do
- Never read the entire file without offset/limit — it's 3000+ lines
- Never use `sed -n 'X,Yp'` — use `Read(file, offset=X, limit=N)` instead
- Never assume a CSS value is still what you last saw — grep to confirm
