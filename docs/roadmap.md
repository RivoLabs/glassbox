# Roadmap

In development. A milestone ships when its acceptance criteria pass. No dates.

## M0: core and report format

Cargo workspace. The report crate: finding, evidence and verdict model, integrity summary serializer, JSON and HTML output, signing and verification. No Windows dependencies.

Acceptance: tests pass on any platform. Fixed findings serialize to stable JSON and render to a self contained HTML file. A signed report verifies and a tampered one fails.

## M1: console shell and self check

The glassbox binary: admin manifest, startup self check (image hash against the published hash, signature when signed), consent flow (banner, plain disclosure, type Y to agree, decline exits, --yes skips), argument parsing, and the pause before exit.

Acceptance: consent, then an empty scan, then verdict, then pause, all work. A modified binary refuses to start. Declining exits with no scan.

## M2: integrity collectors

Layer 1 against real Win32, plus the testbed.

Acceptance: a clean VM shows zero detection counts and non-zero inventory. The testbed moves each counter on its own. Summary output matches the report crate for the same counts.

## M3: memory and forensics

Layers 2 and 3. Privacy sources in match mode.

Acceptance: a seeded injection and a seeded, then deleted, decoy both show up with correct evidence. Privacy sources report hits only. A wiped journal gets flagged.

## M4: rules and file scan

Layer 4 and the signature engine. First public behaviour and signature packs, tested against public samples. Embedded packs plus optional signed packs.

Acceptance: matches known sample hashes in a lab. Zero false positives on a clean baseline. An unsigned or tampered external pack is rejected.

## M5: polish and release

Console colour, live scan code, HTML report, verify command, speed tuning to the 60 second target and the --deep path, reproducible signed release builds with published hashes.

Acceptance: a full run on a reference machine stays under budget. The live code matches the verifiable report. The published hash reproduces from source.

## Ongoing

- CI runs the testbed on Windows on every change. A rule ships only if recall holds and clean baseline false positives stay at zero.
- Per rule precision tracking. Noisy rules get retired.
- Each release asks how the last one could be evaded and adds a check for it.
