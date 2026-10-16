# The FiveM cheat landscape

Research background for a tool in development. This maps what Glassbox should catch: the categories of cheats, loaders and bypasses, and the behaviour that gives each one away. It names no products and links to nothing. Brand names change fast, so rules key on behaviour.

## What gives them away

A cheat shows up as one or more of:
- A file on disk, or a trace of one that was deleted
- A process that ran. Execution history keeps this even after deletion.
- A known bypass or artifact
- Code injected into another process

## Categories

Most FiveM cheats are paid closed menus sold on subscription, plus cracked copies.

- Lua or resource executors: load unauthorised client code to spawn items, teleport, or fire server events
- ESP and aim menus: read and draw game state. The aim part writes to game memory.
- External memory cheats: a separate process reads and writes game memory through VM handles, often with an overlay
- Asset tampering: modified game archives swapped in for real ones, such as spawn weapon mods

Most of these show up through behaviour alone: injection, foreign handles, unbacked memory, overlays and execution history. See [methodology.md](methodology.md).

## Loaders and bypasses

- Mapper loaders: load a signed but vulnerable driver to get the access a cheat needs. Caught by the vulnerable driver check and by service install records, not by the cheat's own signature.
- Loaders: small stubs that fetch and start the real cheat
- Bypasses: tools that claim to beat anti-cheat, usually by wiping the artifacts layer 3 reads. A known bypass present, or evidence of wiping such as a cleared journal or event log, counts as a finding.

## Technique indicators

- PE injection into a running process
- Java agent injection
- DLL injection into the game or other processes
- PowerShell profile persistence
- Asset archive tampering
- Direct3D proxy DLLs on disk
- Unsigned processes starting during the scan

## Rules

Two packs, both open:
- Behaviour rules as data: artifact source, match, severity, label. Anyone can read and fork them.
- Signature rules in YARA format, written against public samples.

Rules live outside the engine on purpose. A server can add a rule for a new cheat without a rebuild, and everyone can see what is matched and why.
