# Glassbox

Research and design for a consent-first anti-cheat scanner for FiveM servers.

**Status: in development.** This repo has the research and the plan. There is no working program yet.

Glassbox is meant to be one Windows program. A player runs it while staff watch. It checks the machine for traces of cheats, prints a verdict, and writes a report. No accounts, no backend, no screen recording. The code will be public so anyone can read what it does.

Closed scanners cost a monthly fee, upload player machine data to a third party, and hide their detection logic. Most of what they do is standard Windows API work. The part worth paying for is the detection content and tuning, and that works better when it is shared. The argument is in [docs/analysis.md](docs/analysis.md).

Four detection layers are planned: runtime integrity, live process memory, execution history, and file and posture checks. See [docs/methodology.md](docs/methodology.md).

Rules the tool is designed around:
- The player is told what will be read before anything runs. Saying no means no scan.
- No screenshots, no recording, no live monitoring.
- Private history sources are only checked for known cheat indicators. Raw history is never printed or saved.
- The player gets the same report staff get.
- Staff make the call. The tool does not ban or kick.

## Docs

| File | What it covers |
|---|---|
| [docs/analysis.md](docs/analysis.md) | Why the scanning mechanism is routine and why self-hosting makes sense |
| [docs/methodology.md](docs/methodology.md) | Each detection layer and the Windows APIs behind it |
| [docs/cheat-landscape.md](docs/cheat-landscape.md) | Cheat, loader and bypass categories, described by behaviour |
| [docs/proposed-design.md](docs/proposed-design.md) | Architecture, self protection, report signing |
| [docs/roadmap.md](docs/roadmap.md) | Milestones M0 to M5 with acceptance criteria |

## License

GPL-3.0. See [LICENSE](LICENSE). Forks stay open. Detection rules will live in a separate openly licensed pack.
