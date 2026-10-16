# Detection methodology

Design research for a tool in development. Every technique here is publicly documented Windows or forensics work. It runs user mode and elevated, with no kernel driver.

## Layer 1: runtime integrity

Checks for a cheat, debugger or injector interfering with code and threads.

### Module and driver trust
Enumerate loaded user modules (EnumProcessModules, or toolhelp Module32First and Module32Next) and kernel drivers (EnumDeviceDrivers, or NtQuerySystemInformation with SystemModuleInformation). Verify each signature with WinVerifyTrust, falling back to catalog hashes (CryptCATAdminCalcHashFromFileHandle) for catalog-signed system files. Count the untrusted ones. The totals double as inventory, so a clean machine still shows a non-zero count.

### Inline hooks in system DLLs
Cheats often overwrite the start of an exported ntdll function with a jump. Map a clean copy of the DLL from disk as data and compare the first bytes of each export against memory. A mismatch is a likely hook.

### Syscall stubs
A real ntdll syscall stub has a fixed shape: mov r10, rcx; mov eax, syscall number; syscall. Walk the stubs and count any that do not match.

### Hardware breakpoints
Debug registers Dr0 to Dr3, armed through Dr7, are a common tracing and anti-tamper signal. Snapshot threads (CreateToolhelp32Snapshot, Thread32First, Thread32Next), read each thread's context with CONTEXT_DEBUG_REGISTERS, and count a thread if any register is set and enabled:

```
(Dr0 && (Dr7 & 0x03)) || (Dr1 && (Dr7 & 0x0C)) ||
(Dr2 && (Dr7 & 0x30)) || (Dr3 && (Dr7 & 0xC0))
```

### Import table
For each module, every resolved import should point into the module that exports it. Anything pointing into unrelated or unbacked memory is a hook.

## Layer 2: live process memory

Runs only while the game is up. Catches in-memory cheats that never touch disk.

- Unbacked executable memory: committed PAGE_EXECUTE regions with no mapped image behind them (VirtualQueryEx). Injected code lives here.
- Threads outside modules: a thread whose start address sits in no known module.
- Foreign handles: other processes holding VM_READ, VM_WRITE or VM_OPERATION handles to the game (NtQuerySystemInformation with SystemHandleInformation).
- Signature scan of the game's private executable memory.
- Overlay windows: topmost, layered or click-through windows sitting over the game (EnumWindows). Window metadata only, no pixels.

## Layer 3: execution history

Answers "was it here before you cleaned up?" Each source is matched against indicators. Raw contents are never dumped.

| Source | What it shows |
|---|---|
| Prefetch | Program ran, when, and how often |
| Application compatibility and install caches | Executables seen, often with hash and path, surviving deletion |
| Background activity records | Executables a user ran, with last run time |
| Explorer launch records | GUI programs started, with counts |
| NTFS change journal | Create, delete and rename events. Flags the journal itself being wiped |
| Recycle Bin metadata | Deleted files with original path and time |
| Event logs | Service installs (driver mappers use these), log clears, code integrity blocks, PowerShell script blocks |
| PowerShell profile and history | Persistence and loader scripts |

Findings say what ran, what was created and what was deleted, with timestamps, even when the file is gone.

## Layer 4: files and posture

### File signature scan
Signature rules over candidate executables, DLLs and drivers. Default scope is the likely spots. The --deep flag widens it to every drive.

### FiveM specific
- FiveM plugin, mod and runtime folders
- Game logs and crash dumps: loaded modules and fault signatures
- Proxy DLLs placed beside the game (d3d10, d3d11, dxgi, dinput8)
- Modified game asset archives, such as spawn weapon mods

### Posture
Context only, never a verdict on its own:
- Test signing on, HVCI off, Secure Boot off, vulnerable driver blocklist off
- Antivirus exclusions and past detections. Cheat install guides routinely say to add exclusions.
- Running inside a VM
- Known vulnerable drivers loaded, matched against a public list. This is how mapper loaders get caught, since they ship a signed but vulnerable driver.

## Privacy sensitive sources

Included in match mode only: read, compare to indicators, report hits.

- Resource usage history: application execution rows only, no network byte counts
- DNS cache: known cheat auth and loader domains only
- Browser caches and history: known cheat shop and loader URLs only, never browsing history
- Hardware IDs (TPM, disk serial): salted hash, so staff can link alt accounts without a raw ID

## Verdict

Each finding has a severity (info, suspicious, detected) and evidence: source, path, timestamp, matched rule, raw value.

- DETECTED: at least one detected finding, such as a matched cheat, an active injection, or a wiped journal alongside other traces.
- SUSPICIOUS: suspicious findings only, such as posture oddities or unexplained unbacked memory.
- CLEAN: nothing above info.

The tool shows evidence and a verdict. It does not act. A person reads the report and decides.
