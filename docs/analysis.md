# Analysis

Paid FiveM scanners sell themselves as hard to copy. This doc tests that by treating the category as an engineering problem.

## Short answer

The scanning is routine. It is documented Windows APIs plus standard forensics technique, public for years. Nothing needs secret knowledge or more than local admin.

What has real value:

1. Detection content: which indicators map to which current cheats, kept up to date as cheats change.
2. Low false positives. Staff only trust a tool that stays quiet on clean machines. That takes a clean baseline set and tuning.
3. Workflow. Turning findings into something a moderator can act on fast.

None of these is secret. All three get better when people can see and contribute.

## Where commercial tools spend effort

- Packing and anti-tamper. Heavy protection on a closed binary. It protects the business model, not the player. An open tool gets its integrity from being readable and reproducibly built.
- Hosted backend and accounts. Sessions, licensing, dashboards, result storage. Ordinary web work, there to support a subscription.
- Embedded browser UI. Polish. It adds nothing to detection and widens the attack surface of an elevated process.

Remove those and what is left is a few thousand lines of Win32 and forensics code plus rules. That remainder is Glassbox.

## Why self-hosting makes sense

- The expensive part is data, and data can be pooled. Indicators improve when many servers contribute. Nobody should have to rebuy them every month.
- No seat cost. A local tool has no backend to pay for. One signed build scans any number of machines.
- Auditability. A tool that reads someone's computer should be inspectable. Closed scanners ask you to trust them.
- Nothing leaves the machine except to the staff running the check. No third party holds the data.

## Limits

- An open detector is easier to evade in principle, since its logic is public. Behaviour-based rules, corroborating evidence that is hard to fake consistently, and community indicator updates reduce this. They do not remove it.
- A patched build can lie. The signed hash and live scan code catch a tampered official build. They cannot stop someone running a different program and calling it Glassbox. The real defence is staff watching a known good build run. That is partial.
- The tool is only as good as its rules. That is why the rules are open and need upkeep.

Bottom line: the mechanism does not justify a closed, paid, data-uploading product. A transparent, self-hosted alternative is feasible and better for most servers.
