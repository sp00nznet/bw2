# Black & White 2 — Static Recompilation

Static recompilation of **Black & White 2** (Lionhead Studios / Electronic
Arts, 2005) from its shipping Win32 binary to native C.

Built on the [pcrecomp](https://github.com/sp00nznet/pcrecomp) toolchain, and
directly on [bw](https://github.com/sp00nznet/bw) — the same studio's engine,
four years on.

## Project Status: **P0 partial. The target binary is on a later disc and has not been extracted yet.**

---

## What P0 found

Four discs, `data1.hdr` + `data1.cab` + `data2.cab` per disc — **InstallShield
version 9**.

The installer header is authoritative about what it holds, and what it says is
the headline:

```
 893      C      21739061      19605290   white.exe
```

**`white.exe` is 21,739,061 bytes.** That is the largest binary this toolchain
has ever been pointed at, by a distance:

| Binary | Size | Status |
|--------|-----:|--------|
| **`white.exe`** (B&W2, 2005) | **21.74 MB** | not yet extracted |
| Rise of Legends (2006) | 13.25 MB | "the stress test", private |
| Encarta 97 `ENC97.EXE` + 5 DLLs | ~7 MB total | runs |
| Black & White (2001) | ~5 MB | active, 569 types done |

`white.exe` lives at file index 893, and disc 1's cabinets only carry indices
0–103. It is on disc 2, 3 or 4 and pulling it means walking a four-disc
multi-volume InstallShield set. That is the next thing to do here and it has
not been done.

What *did* come off disc 1, byte-exact:

| Binary | Size | `.text` | Built | Role |
|--------|-----:|--------:|-------|------|
| `Black & White 2_code.exe` | 323,584 | 134,068 | 2005-08-19 | launcher / config |
| `binkw32.dll` | 339,456 | — | 2005-03-03 | Bink video (88 exports) |
| `DrvMgt.dll` | 41,472 | — | 2005-02-16 | **SafeDisc** driver manager |

MSVC 7.1 (Visual Studio .NET 2003). The launcher is clean — entropy 1.17–6.64,
no wrapper sections.

`DrvMgt.dll` plus `00000001.TMP` and `00000002.TMP` in the disc root is the
SafeDisc signature. Whether it wraps `white.exe` itself is unknown until
`white.exe` is in hand; `Black & White 2_code.exe` is not wrapped, and on
OMF: Battlegrounds the same disc-level markers turned out to cover only the
53 KB launcher and none of the 4.5 MB of engine. Do not assume either way.

## Why bother, given `bw` exists

The first Black & White is the most C++-heavy project in the collection and it
is what forced the whole of `tools/cpp/` into existence — MSVC and Metrowerks
mangling in both directions, vtable parsing, a seven-level class hierarchy
(`Base` → `GameThing` → `Object` → `Mobile` → `Living`…) and a 135 KB
`CreatureMental` struct. All 569 types are done.

Black & White 2 is that engine four years and one compiler generation later.
Every type name recovered by hand for `bw` is a hypothesis to test against this
binary, and where they still match, that is 569 types of head start on a 21 MB
image. Where they do not, the diff is the four years.

It is also the honest answer to "can this toolchain handle a 2005 AAA title".
Rise of Legends says *not yet* — 25,513 functions reachable only through
vtables, no RTTI, and the standing conclusion is that a real vtable scanner has
to be ported from `xboxrecomp` first. B&W2 is bigger than Rise of Legends and
will need the same thing. **Treat the vtable scanner as a prerequisite, not a
discovery.**

## Where it goes next

1. Unpack discs 2–4 and extract `white.exe`. Nothing else can start.
2. Port the vtable scanner from `xboxrecomp` (see
   [CONSOLIDATION.md](https://github.com/sp00nznet/pcrecomp/blob/main/docs/CONSOLIDATION.md)).
   This target and Rise of Legends both need it.
3. Run `bw`'s recovered type names against the new binary and measure what
   survived.
4. Only then disassemble.

## Layout

```
bw2/
  original/   four discs (.bin/.cue, and bw2d1.iso converted from disc 1)
  disc/       disc 1's installer cabinets
  game/       what has been extracted so far
  analysis/
  docs/
```

## Credits

Black & White 2 © 2005 Lionhead Studios / Electronic Arts. This project neither
contains nor distributes any part of it.
