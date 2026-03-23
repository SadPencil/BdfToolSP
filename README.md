# BdfToolSP — Special Pixel BDF Font Tool

A command-line tool for processing [BDF (Bitmap Distribution Format)](https://en.wikipedia.org/wiki/Glyph_Bitmap_Distribution_Format) font files.

**SP** stands for **Special Pixel** — fitting for a tool that works with pixel/bitmap fonts.

## Features

- **Downgrade** BDF version 2.2 files to version 2.1
- **Merge** two BDF 2.1 files into a single font file
- **Rename** the font name (`FAMILY_NAME`, `FONT_NAME`, `FACE_NAME`) in the output

## Usage

### `downgrade` — Convert BDF 2.2 to BDF 2.1

```
BdfToolSP downgrade [--font-name <name>] <input.bdf> <output.bdf>
```

| Argument | Required | Description |
|---|---|---|
| `<input.bdf>` | ✓ | Input BDF file (version 2.2 or 2.1) |
| `<output.bdf>` | ✓ | Output BDF 2.1 file |
| `--font-name <name>` | | Rewrite `FAMILY_NAME`, `FONT_NAME`, and `FACE_NAME` to `<name>` |

**What the downgrade does:**
1. Changes `STARTFONT 2.2` → `STARTFONT 2.1`
2. Removes `METRICSSET` (BDF 2.2-only keyword)
3. Removes global `SWIDTH`/`DWIDTH`/`SWIDTH1`/`DWIDTH1`/`VVECTOR` header lines
4. Removes per-glyph vertical metrics (`SWIDTH1`, `DWIDTH1`, `VVECTOR`)
5. Injects explicit `SWIDTH`/`DWIDTH` for glyphs that relied on global defaults
6. Renames each `STARTCHAR` to the 4-digit uppercase hex of its `ENCODING` value

**Examples:**

```bash
# Basic downgrade
BdfToolSP downgrade input.bdf output.bdf

# Downgrade and rename the font
BdfToolSP downgrade --font-name "My Font 14px" input.bdf output.bdf
```

---

### `merge` — Combine two BDF 2.1 files

```
BdfToolSP merge --font-name <name> <input1.bdf> <input2.bdf> <output.bdf>
```

| Argument | Required | Description |
|---|---|---|
| `--font-name <name>` | ✓ | Name to set for `FAMILY_NAME`, `FONT_NAME`, and `FACE_NAME` |
| `<input1.bdf>` | ✓ | First BDF 2.1 font file (takes precedence on conflicts) |
| `<input2.bdf>` | ✓ | Second BDF 2.1 font file |
| `<output.bdf>` | ✓ | Output merged BDF 2.1 file |

**Merge behaviour:**
- Both input files must be BDF 2.1 (the tool aborts if either is not)
- The header (font metadata) is taken from the first file
- When both files define a glyph for the same encoding, the first file's glyph wins
- Glyphs with `ENCODING -1` (unmapped) from both files are all retained
- The `CHARS` count in the output is updated to reflect the total number of glyphs
- Glyphs are written in ascending encoding order, with unmapped glyphs last

**Example:**

```bash
# Merge two fonts and name the result
BdfToolSP merge --font-name "SimSun 14px" base.bdf extra.bdf merged.bdf
```

---

## Sample BDF Files

Two example BDF 2.1 files are included in this repository:

| File | Glyphs | Description |
|---|---|---|
| `MicrosoftSansSerif-14.bdf` | 4,416 | Microsoft Sans Serif, 14 px |
| `simsun-14px-fontforge-21.bdf` | 28,431 | SimSun (Chinese), 14 px |

You can try merging them:

```bash
BdfToolSP merge --font-name "SimSun 14px" \
    MicrosoftSansSerif-14.bdf simsun-14px-fontforge-21.bdf merged.bdf
```

## Building

Requires [.NET 10 SDK](https://dotnet.microsoft.com/download) or later.

```bash
dotnet build BdfToolSP/BdfToolSP.csproj
dotnet run --project BdfToolSP/BdfToolSP.csproj -- <args>
```
