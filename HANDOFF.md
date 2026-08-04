# Handoff - borismilner.github.io (personal site)

**Written:** 2026-08-05 (session of 2026-08-04) · **Assignment:** /home/boris-milner/me/projects/borismilner.github.io · **Type:** personal

This repo has no other docs. This file is the entry point: read it, then read
`index.html` (it is the whole site).

**Anything you add to this repo is published.** Pages serves the whole tree, so
a new doc appears on the public site unless you list it in `_config.yml`'s
`exclude:` - that is the only reason that file exists. This handoff is in git
(so it reaches the VM) and returns 404 on the web. Verified both.

## Do this next

Nothing is in flight. The next session starts from a request, not from a queue.
When one arrives:

```bash
cd /home/boris-milner/me/projects/borismilner.github.io
git status                      # expect clean at d7378bf or later
git log --oneline -6

# serve + drive a real browser (never review from code alone)
# use your own session scratchpad for D if you have one
D=/tmp/claude-verify; mkdir -p $D
setsid python3 -m http.server 8899 >$D/serve.log 2>&1 < /dev/null & disown
setsid google-chrome --remote-debugging-port=9222 --user-data-dir=$D/chrome \
  --no-first-run --no-default-browser-check --headless=new --hide-scrollbars \
  about:blank >$D/chrome.log 2>&1 < /dev/null & disown
sleep 5; curl -s http://127.0.0.1:9222/json/version | head -c 40
```

Then drive it with the `mcp__chrome-personal__*` tools (deferred - load their
schemas first with `ToolSearch("select:mcp__chrome-personal__new_page,...")`):
`new_page` `http://localhost:8899/?v=N` (bump `N` every reload - **python's
http.server serves a cached copy otherwise and you will review stale CSS**),
`emulate` for viewport + `colorScheme`, `take_screenshot`, `evaluate_script` to
measure boxes, `press_key` for the 1/2/3 nav.

Two browser tricks worth reusing: freeze a mid-animation frame by injecting a
`<style>` with `animation-play-state: paused` (or by pinning the animated
property directly) before screenshotting, and check reduced motion for real by
relaunching Chrome with `--force-prefers-reduced-motion`.

Check every change at **1680** (wide desktop), **1440** (the common laptop),
**1088** (exactly the hero-stack boundary) and **390** px (phone), in **both**
colour schemes. Those four catch every layout mode the page has.

Ship:

```bash
git add index.html && git commit -m "type(scope): ..." && git push origin master
# Pages publishes in 40-60s; poll for a string only the new version contains:
for i in $(seq 1 12); do curl -s https://borismilner.github.io/ | grep -q '<marker>' \
  && { echo live; break; }; sleep 10; done
```

Then **stop the server and Chrome** (`pkill -9 -f 'http\.server 8899'`,
`pkill -9 -f 'remote-debugging-port=9222'`) and re-verify against
`https://borismilner.github.io/` in a fresh page.

Two traps in that cleanup. A bare `pkill -f <pattern>` also matches the `zsh -c`
wrapper your own Bash call runs inside, so the command kills itself and returns
exit 144 - escape the pattern (`http\.server`), and confirm the kill in a
*separate* call. And Boris keeps his own Chrome open: only ever kill instances
whose `--user-data-dir` is your scratchpad, never a bare `chrome` PID.

## Where we are

A one-file static site (`index.html`, ~1480 lines, inline CSS + JS, no build
step, no frameworks) published by GitHub Pages from `master`. This session ran
three rounds of Boris's visual review: the project card became a full-width
three-panel picker, an 8200/military bubble joined the portrait ring with a cool
light that runs its rim, the AgentBox section stopped floating text around the
mascot, and the type scale went fluid so the reading column widens with the
screen while the hero fills it. All three rounds are committed, pushed and
verified live. He said "looks great" and called `/handoff`.

## Live state (volatile - verify on resume)

- Background jobs: none. My `python3 -m http.server 8899` and headless Chrome
  (`--user-data-dir=/tmp/claude-1000/.../chrome-profile*`) are killed. Chrome
  PID 12117 is **Boris's own** browser - leave it alone.
- PRs: none. This repo does not use PRs; `master` is the deploy branch.
- Git: `master` clean and fully pushed. **`d7378bf` is the last commit that
  touched the site itself**; everything after it is docs (`f2119c8` this file,
  `3114c76` the Pages exclude, then this doc's own corrections). Run
  `git log --oneline -6` for the real tip rather than trusting this line.
- In-flight edits: none.

## Blocked on you (Boris)

Nothing - proceed autonomously. One open call he may want to make:
`card.jpg` (the OG/social preview, 1200x630) still shows the **pre-redesign**
site. Regenerating it needs a new render; nobody has asked for it yet.

## I can do solo (no input needed)

1. Regenerate `card.jpg` from the current design if asked (or propose it).
2. Cross-browser pass: Firefox and Safari are **untested**. The at-risk features
   are listed under Facts; each already has a fallback, but none was exercised.
3. Any further visual iteration - the workflow above is the whole loop.

## Facts - verified vs assumed

Verified this session, in a real headless Chrome 151 at the widths above:

- [verified] Hero, card, all three sections and the footer render correctly at
  1680 / 1500 / 1440 / 1280 / 1190 / 1101 / 1088 / 390 px, light and dark.
- [verified] No horizontal scroll at any of those widths, including with the
  root font forced to 24px (the `--bleed` gates are in `em` for that reason).
- [verified] Keyboard 1/2/3 sets the hash, the browser scrolls, the landed
  heading gets its colour wash, and the `h2` takes focus.
- [verified] `Tab` shows a focus ring on the card options.
- [verified] `prefers-reduced-motion: reduce` (Chrome `--force-prefers-reduced-motion`)
  kills the bob, the rim light, the card arrival, the marching ants and the rail,
  and the light-run scheduler never fires (waited 12s past its first delay).
- [verified] The light run fires every ~5-11s, one bubble at a time, a different
  bubble each time, with a random entry angle, direction and duration per run
  (observed: 8200 180deg/+375, Rafael 123deg/+416, Intel 233deg/-382).
- [verified] The reading rail is `scale: 0 1` at the top and `1` at the bottom.
- [verified] No console errors or warnings.
- [verified] `https://borismilner.github.io/` serves `d7378bf`, and after the
  `_config.yml` exclude the site is still 200 while `/HANDOFF.md` and
  `/HANDOFF.html` are 404.

Re-check before relying on these:

- [assumed] Firefox and Safari. Untested. At risk: `mask-composite: exclude`
  (the light ring), scroll-driven `view()` / `scroll(root)` timelines (the
  self-drawing arrow, the queue fills, the archive rows, the rail),
  `text-box: trim-both` (Chrome 133+ only, `@supports`-guarded), `:has()`,
  container queries, `color-mix()`. Every one of them degrades to a static but
  correct layout **by construction**, not by test.
- [assumed] The archive rows' `view()` reveal cannot get stuck invisible. The
  reasoning: they sit far down a long page, so they always cross their `entry`
  range. A full-page screenshot shows them faded - that is a screenshot
  artefact of the giant viewport, confirmed by reading their opacity as `1`
  after a normal scroll.
- [assumed] Print styles. There are none.

## How the CSS is put together (read before editing)

- **One fluid scale.** `html{ font-size: clamp(1rem, 0.95rem + 0.4vw, 1.2rem) }`
  and every measure is in `em`, so the reading column widens with the screen
  while a line keeps the same character count. Do not "fix" a narrow column by
  changing `em` counts - change the root clamp.
- **Two shells.** `.col` = 41em (prose), `.wide` = 62em (hero + card). They
  share edges on purpose: the masthead and the card are one composition.
- **`--bleed`** is how far a mock reaches past the prose column, per side. It is
  declared in `rem` (read by elements at different font sizes) and gated in `em`
  media queries (74em / 90em) so a large default font never pushes a figure off
  screen. Figures bleed symmetrically; `figcaption` walks back to the text edge.
- **Mocks size to their content**, then centre: `pre{ width: fit-content }`,
  `.capture{ max-width: 46rem }`, `.queue{ max-width: 42rem }`. A block padded
  out with empty space is the bug this prevents.
- **The Snapper mock is one composition**: `.shot` holds the window *and* the
  annotation, with `padding-left: 9rem` as the canvas room the annotation hangs
  into. Anchor annotations to `.shot`, never to `.capture`, or they drift away
  from the checkbox they point at as the figure widens.
- **Every breakpoint, and why its unit is what it is.** `em` where the reader's
  own font size must move the breakpoint with the type: `68em` (hero stops being
  two columns and the bubbles become pills), `74em` / `90em` (the two `--bleed`
  steps). `px` where it is a physical small-screen concession: `640px` +
  `(pointer: coarse)` (drop the keyboard hint in the card strip), `700px` (the
  mascot stops being a column), `560px` (archive rows stack). Container queries,
  not media queries, wherever a component must react to its own box: `46em` on
  the card seat (three panels become a list), `36em` / `30em` on the Snapper
  figure (annotation gives up its canvas room, then moves), `26em` on the queue.
  Media-query `em` resolves against the browser's *initial* font size, not the
  fluid root - that is exactly why the bleed gates are safe.
- **The light run** is CSS (`.bubble::before` conic ring masked to 2px,
  `::after` halo) plus a JS scheduler that sets `--arc-from`, `--arc-turn` and
  `--arc-time` per run. Keep the randomness in JS and the drawing in CSS.

## House rules that shaped this site

- Product names are capitalised in prose and UI (**AgentBox, Snapper, Grabbit**);
  the CLI stays lowercase (`agentbox confirm ...`), because that is the real
  command.
- Redundant means delete it, not condense it. The footer's duplicate link row is
  gone for that reason - do not reintroduce a second copy of the contact links.
- Nothing on this page may read as machine-written: no em-dashes, no filler
  vocabulary, no emoji garnish. Comments in `index.html` explain *why* a rule
  exists, in that voice. Match it.
- Never report a visual change as working from reading the diff. Exercise it.

## Declutter ledger

| Removed / condensed | Where its knowledge now lives |
|---|---|
| Nothing removed in this repo - it had no docs before this file | n/a |
| The AgentBox FR83 block in `~/.claude/last-handoff.md`, condensed from ~20 lines to a dense pointer when this handoff took the top slot | `/home/boris-milner/me/projects/agentbox/HANDOFF.md` (unchanged); the pointer keeps the deployed SHA, both defect fixes and the full next-step order |

## Map

1. `HANDOFF.md` (this file) - the entry point.
2. `index.html` - the entire site, in three parts: one inline `<style>`, then the
   markup, then one inline `<script>`. The page reads:
   `.rail` (scroll progress) → `header.masthead` (`.words` + `.art-wrap` with the
   `.bubbles` ring) → `.card-seat` (the AgentBox card, which is also the nav) →
   `main` with `#agentbox` (`.with-mascot` band + `pre` snippet), `#snapper`
   (`.capture` > `.shot` mock), `#grabbit` (`.queue` mock), `#archive`, `#hello`
   (`.ways` contact row) → `footer` (one colophon line). The script holds three
   things: the 1/2/3 keyboard nav, the Grabbit countdown theatre, and the
   light-run scheduler.
3. `boris.webp` - hero portrait. `agentbox-mascot.webp` - AgentBox section aside.
   `card.jpg` - OG preview, **stale**.
4. `_config.yml` - Pages build config, and the only thing in it is the list of
   files to keep off the web. Add any future doc there.
5. Live: https://borismilner.github.io/ · Source:
   https://github.com/borismilner/borismilner.github.io
