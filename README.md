# Detego

Chromium-based browser credential recovery tool. Extracts saved passwords from all installed Chromium browsers (Chrome, Edge, Opera, Brave, Vivaldi, etc.)

## Features

- **Auto-discovery** — Scans registry and filesystem for all installed Chromium browsers and profiles (*NO hardcoded paths*)
- **v10 + v20 decryption** — Recovers both DPAPI keys (v10) and app-bound encrypted keys (v20)
- **Raw NTFS reading** — Reads files directly from disk by parsing NTFS structures (boot sector, MFT, index B-trees), ***bypassing the entire filesystem stack and all minifilters***. (*Falls back to ntdll section mapping + file ID open, then path-based open*)
- **No CRT dependency** — Zero C runtime. All standard library functions (memcpy, strlen, sprintf, etc.) resolved from ntdll at runtime via PEB walk + PE export hash
- **String encryption** — All strings encrypted at compile time with a position-dependent spiral cipher. Decrypted on the stack at call sites
- **Minimal imports** — Injection APIs (NtAllocateVirtualMemory, NtWriteVirtualMemory, NtCreateThreadEx, etc.) resolved from ntdll at runtime.
- **Inline SQLite parser** — Custom read-only SQLite page/cell parser. ***No sqlite3 dependency.***

## Architecture

```
chromium.c              Main entry point
common.h                Buf (dynamic byte array), print(), ntdll CRT resolver, string encryption macros
ntdll_reader.h          Three-tier fallback file reading: raw NTFS -> section mapping + file ID -> path-based
sqlite_reader.h         Read-only SQLite parser (B-tree pages, cell/record decoding)
chromium_finder.h       Browser discovery (registry scan, filesystem enum, PE version info)
chromium_injector.h     v20 key extraction via DLL injection into suspended browser process
chromium_decryptor.h    AES-GCM decryption (v10 DPAPI + v20 app-bound), credential output
chromium_util.dll       Injected DLL — calls browser's internal decryption for app-bound keys
```

## Build

Requires Visual Studio Build Tools (MSVC). Build from **x64 Native Tools Command Prompt**:

```
rc chromium.rc

cl /O1 /GS- /Gs999999 /Zl /TP /EHs-c- /GR- /Zc:threadSafeInit- chromium.c /link /NODEFAULTLIB /ENTRY:mainCRTStartup /SUBSYSTEM:CONSOLE /OPT:REF /OPT:ICF /MERGE:.rdata=.text chromium.res kernel32.lib advapi32.lib bcrypt.lib crypt32.lib version.lib shell32.lib ole32.lib user32.lib
```

## Requirements

- Windows 10/11 x64
- Administrator privileges (required for raw NTFS volume access; UAC prompt via embedded manifest)


*0xROOTPLS — Icalia Systems*
