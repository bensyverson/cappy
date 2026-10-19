# Cappy — first cut plan

*2026-10-19 · Ben Syverson & Claude · status: planned; dispatch after Penumbra's knVkPr lands*

The first cut ruled in [2026-10-19-vision.md](2026-10-19-vision.md): `load`, `shot`, `ax`, `query`, `find`, `press` and `doctor`, plus the bare primer and `recipes`. Every leaf mirrors SleepyHollow (`../sleepyhollow/`): read the sibling's verb, its `--help` text and its output writer before writing Cappy's, and match names, flags, formats and exit codes unless the native world forces a difference. Record each difference in the verb's `--help` and in the vision doc.

The order follows the dependencies. The skeleton and the helper come first, because every verb needs the exit-status contract and the permissions. `ax`, `query` and `find` share the tree walker. `shot` is independent of them, and `press` needs `query`.

```yaml
tasks:
  - title: "Cappy first cut: load, shot, ax, query, find, press, doctor"
    ref: root
    desc: |
      Cappy is SleepyHollow for native Mac apps; see project/2026-10-19-vision.md (rulings at its end). Mirror sleepy's repo layout, verbs, output formats and exit codes so an agent can switch between them without re-reading help. First user: Penumbra's probe (../Penumbra/scripts/probe), whose in-process captures miss layer-drawn controls, material colors and layer opacity.
    children:
      - title: "Package skeleton mirroring SleepyHollow"
        ref: skeleton
        desc: |
          Package.swift with library Cappy (folders Core, Host, Operations, Output, Session), CappyCLIKit (shared ArgumentParser options), executable cappy, TestSupport and CappyTests, as sleepy has (../sleepyhollow/Package.swift). Swift 6 strict concurrency, macOS 15+. ExitStatus with sleepy's six codes (0 success, 1 negative, 2 usage, 3 timeout, 4 target not found, 5 environment). The bare `cappy` primer and `cappy --help` in sleepy's format; `cappy recipes` (goal-to-verb routing). Target addressing shared by every verb: <app> as bundle ID, name or pid:<n>, plus --window <title|index|id>. A pre-commit hook running swiftformat --lint and swift test, like sleepy's scripts/githooks.
        criteria:
          - "`cappy` with no arguments prints a primer in sleepy's shape, and every verb's --help lists flags, examples and exit codes"
          - "Exit codes match sleepy's table, with 4 meaning target not found and 5 a missing permission"
      - title: "Permission helper app and doctor"
        ref: helper
        blockedBy: [skeleton]
        desc: |
          A small signed LSUIElement helper app holds Screen Recording and Accessibility; the CLI talks to it over XPC (vision ruling 1). Its signing identity must be stable across rebuilds so grants persist; document how to build, sign, install and grant it once. `cappy doctor` reports each grant, the helper's path and signature, and the macOS version, and says exactly what to do when something is missing. Never trigger a permission prompt without saying so first; tests needing a grant are tagged and skip with a clear message.
        criteria:
          - "With grants missing, every verb exits 5 with the grant it needs and how to give it"
          - "`cappy doctor` reports both grants, the helper and the OS, and exits 5 when anything is missing"
      - title: "load: is the target there, and its windows"
        blockedBy: [helper]
        desc: |
          Mirror `sleepy load`: report the app (name, bundle ID, pid, active), its windows (title, id, frame, key, main, on-screen, minimized) and its appearance (light or dark), as JSON. Exit 4 when no such app or window.
        criteria:
          - "load lists a fixture app's windows with frames that match the app's own report"
      - title: "ax, query and find: the accessibility tree"
        ref: ax
        blockedBy: [helper]
        desc: |
          One tree walker over the Accessibility API, with a budget and depth limit that always terminates. `ax` prints sleepy's outline format (one node per line, `role "name" (states)`, indented by depth) or --format json with frames, ids and actions; roles are mapped to readable names the way sleepy's are. `query` matches by role, name, value or identifier and reports frame, enabled, focused, hidden, on-screen and actions. `find` searches titles, values and labels; exit 0 if found, 1 if not. Being an assistive client makes SwiftUI build its tree: test against a fixture app with AppKit and SwiftUI windows, including a SwiftUI checkbox, a pop-up, a disclosure header with an Expanded value, and a control hidden by opacity 0.
        criteria:
          - "ax reads a SwiftUI fixture's heading, checkbox and pop-up with their names and states"
          - "A control hidden by opacity 0 or isHidden is reported hidden, not visible"
      - title: "shot: the composited window"
        blockedBy: [helper]
        desc: |
          Mirror `sleepy shot` (--out, --scale, refuse to upsample): capture one window through ScreenCaptureKit, including while it is behind other windows, without raising or activating it; --element <query> crops to an element's frame, --rect to a window rect. Prove what in-process capture misses: a fixture window with a checkbox, a pop-up, secondary text on a sidebar material and a layer at opacity 0 must look as it does on screen.
        criteria:
          - "shot captures a fixture window behind another window without activating it"
          - "The checkbox, pop-up bezel and material-colored text appear, and the opacity-0 layer does not"
      - title: "press: accessibility actions"
        blockedBy: [ax]
        desc: |
          Mirror `sleepy click`, named press because it performs an accessibility action: AXPress by default, --action <name> for another (AXShowMenu, AXIncrement), on the element a query names; refuse when the query matches none or several, listing candidates. --point x,y posts a real event at a window point instead. Prove it on SwiftUI controls that synthetic in-process clicks never reach: a Button, a checkbox Toggle, and a tap-gesture disclosure header.
        criteria:
          - "press toggles a SwiftUI fixture checkbox and fires a SwiftUI button, verified by reading ax afterwards"
      - title: "README, recipes and docs in sleepy's voice"
        blockedBy: [skeleton]
        desc: |
          README like sleepy's: one-line description, what it is, install (experimental-install --product cappy, plus the helper), quick start with load, ax | grep, shot and press, the exit-code table, links. recipes covers every first-cut verb. DocC for the library at 100%.
        criteria:
          - "An agent that knows sleepy can find each first-cut verb from `cappy` and `cappy recipes` alone"
```

## After the first cut

`contrast`, `fill`, `pick`, `clip`, sessions (`open`/`close`/`sessions`) and `log`, then the Penumbra probe step (vision ruling 2), which is filed in Penumbra's own job tree.
