# Cappy

A command-line tool that looks at Mac apps and tells computer programs what it sees.

Cappy is built for AI agents and automated tests, not people. It looks at a running app's real window, as the screen shows it, and lets you ask for what you need — a screenshot, the accessibility tree, one control's state — as one command that prints structured text and exits with a code that means something. It can also press buttons and toggles the way assistive technology does. It is the native-app sibling of [SleepyHollow](https://github.com/bensyverson/sleepyhollow) (`sleepy`), with the same verbs, formats and exit codes, so an agent that knows one knows the other.

**Status: planned, not yet built.** The design is in [project/2026-10-19-vision.md](project/2026-10-19-vision.md) and the first cut in [project/2026-10-19-first-cut-plan.md](project/2026-10-19-first-cut-plan.md).

## Quick start (planned)

Is the app there, and what windows does it have?

```sh
cappy load com.example.MyApp
```

Read a window the way VoiceOver does:

```sh
$ cappy ax com.example.MyApp | grep heading
    heading "Position" (expanded)
```

Take a screenshot of the real, composited window, even while it's behind others:

```sh
cappy shot com.example.MyApp --out window.png
```

Press a control:

```sh
cappy press com.example.MyApp --role checkbox --name "Visible"
```

Exit codes are a public contract shared with `sleepy`: 0 success, 1 a clean negative, 2 a usage error, 3 a timeout, 4 target not found, 5 environment (such as a missing permission). `cappy doctor` says what is missing and how to fix it.

## Author and license

Ben Syverson. MIT License; see [LICENSE](LICENSE).
