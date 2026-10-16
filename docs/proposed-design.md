# Proposed design

Design for a tool in development. Nothing here ships yet.

One native Windows executable. No installer, no service, no GUI, no network. The player runs it, it asks for consent, scans, prints a verdict, writes two reports, and waits for a key so the window stays open when double clicked.

```
 glassbox.exe  (x64, requireAdministrator, static CRT, rules embedded)
 +------------------------------------------------------------------+
 |  startup self check                                              |
 |    - verify own code signature (when the build is signed)        |
 |    - verify own image hash against the published release hash   |
 |    - refuse to continue if modified                              |
 +------------------------------------------------------------------+
 |  consent (console)                                               |
 |    banner, plain list of what will be read, "type Y to agree"    |
 |    decline exits with no scan. --yes skips for scripted use      |
 +------------------------------------------------------------------+
 |  scan orchestrator (atomic counters, jittered scheduling)        |
 |    integrity collectors                                          |
 |    live process memory collectors                                |
 |    execution history collectors                                  |
 |    file and posture collectors (signature engine, embedded rules)|
 +------------------------------------------------------------------+
 |  verdict and report                                              |
 |    console verdict: CLEAN, SUSPICIOUS or DETECTED, plus scan code|
 |    report.json (full evidence, signed)                           |
 |    report.html (self contained, offline, readable by staff)      |
 +------------------------------------------------------------------+
```

## Why a console app

Hosted scanners wrap the scan in an embedded browser and tie each run to an operator account on a backend. That exists to sell a subscription.

Without a backend there are no accounts, no session binding and no upload. A console app is smaller, has less attack surface, works the same over remote desktop or screen share (how these checks happen in practice), and is easy to read.

## Stack

- Rust, MSVC target x86_64-pc-windows-msvc, static CRT, one exe with rules and report template embedded
- The windows and windows-sys crates for Win32
- A pure Rust YARA compatible engine, so no C toolchain
- Windows 10 22H2 and Windows 11, x64 only. FiveM is x64 only.

## Crates

A Cargo workspace, so each part is testable and Windows code stays in its own crates.

| Crate | Job |
|---|---|
| report | Finding, evidence and verdict model. Integrity summary serializer. JSON and HTML output. Signing and verification. No Windows dependencies. |
| engine | Scan orchestrator. Collector trait, atomic counters, jittered scheduling, severity rollup. |
| collectors | Runtime integrity and live process memory, Win32. |
| forensics | Execution history parsers, the change journal and relevant event logs. |
| rules | Signature and behaviour packs embedded as data. Loader and signature check for external packs. |
| glassbox (bin) | The console app. Self check, consent, wiring, arguments, exit. |
| testbed (bin) | Plants each condition (inline hook, syscall patch, import hook, hardware breakpoint, decoy files) so each collector can be shown to fire in CI. |

## Self protection

1. Own image check. At startup the binary hashes its mapped image and checks its own signature when the build is signed. A modified binary refuses to run. Anyone can rebuild an open tool, so this catches a tampered official build against a published hash. It is not DRM.
2. Runtime checks. The integrity collectors also watch the scanner's own process, so hooks or breakpoints on it show up.
3. Tamper evident reports, below.

No kernel driver. User mode admin covers all of this. A driver adds BSOD risk, EV signing with attestation, and trust cost for an open tool.

## Reports

Staff need to trust a report sent after the fact.

- The console prints a short scan code at the end: scan ID plus a truncated report hash. Staff watching the share see it live.
- The report carries an Ed25519 signature from the release key.
- `glassbox verify <report>` recomputes the hash, checks the signature, and prints the code to match.

Nothing goes to a server. The signature shows an untampered official build made the file. The live code shows this is the report from the scan staff watched.

## Rules

Compiled in, so it works offline. Online, it may fetch a newer signed rule pack from the project releases and use it only if the signature checks out.

## Speed

Target: under 60 seconds on a normal gaming PC. Integrity, memory and forensics run in full. File scanning covers the likely places: Downloads, Desktop, AppData, Temp and the FiveM folders. --deep adds a full drive sweep.
