---
name: swan2swash
description: Converts SWAN (.swn) and SWASH (.sws) command files bidirectionally, with a prerequisite data checklist before conversion. Use when the user mentions swan2swash, SWAN/SWASH conversion, .swn/.sws, nesting SWAN into SWASH, or porting coastal wave models between spectral and nonhydrostatic codes.
---

# swan2swash

Bidirectional conversion between SWAN spectral wave models (`.swn`) and SWASH phase-resolving models (`.sws`). **Always deliver the prerequisite checklist first**, then convert only after the user confirms what they have.

## Repository assets

| Resource | Path |
|----------|------|
| SWAN manual | `swanuse.md` (Ch. 4 commands, §4.2 sequence) |
| SWASH manual | `swashuse.md` (Ch. 4 commands, §4.2 sequence) |
| Reference pair | `swan2swash_ex/2bb1v4.swn` ↔ `swan2swash_ex/2bb1v4.sws` |
| Command mapping | [mapping.md](mapping.md) |
| Worked diff notes | [examples.md](examples.md) |

Read the reference pair before writing a new conversion. Consult manuals for any command not covered in `mapping.md`.

## Workflow (mandatory order)

```
1. Ask direction: SWAN→SWASH | SWASH→SWAN | both (audit only)
2. Emit checklist (below) — mark [have] / [missing] / [N/A]
3. User confirms or supplies missing items
4. Read source .swn or .sws (+ linked grid/boundary files if present)
5. Draft target file; list assumptions and manual follow-ups
6. Verify block order vs manual §4.2 (SWAN: swanuse.md; SWASH: swashuse.md)
```

Do **not** invent bathymetry, spectra, or grid files. If data is missing, stop at checklist and specify exact file names/formats to create.

---

## Prerequisite checklist (give this to the user first)

Copy the section matching conversion direction. Adapt grid labels to the user’s coordinate system (Cartesian m vs spherical deg).

### A. Common (both directions)

| # | Item | Why needed | Typical source |
|---|------|------------|----------------|
| 1 | **Coordinate system** | Must match between models if nested | `COORD` / `COORDINATES` in command file |
| 2 | **Domain extent** | `CGRID` / `INPGRID` origin & length | Command file: x₀, y₀, α, Lx, Ly |
| 3 | **Horizontal resolution** | Grid dimensions & Δx, Δy | `CGRID` / `INPGRID` mx, my, dx, dy |
| 4 | **Bathymetry** | Bottom levels on input grid | `READINP BOTTOM` → `.bot` (or equivalent) |
| 5 | **Exception / land mask** | Dry points, harbours | `INPGRID … EXC` value; MPI may require EXC on bottom |
| 6 | **Project metadata** | Run ID in output | `PROJECT` / `PROJ` name & run number |
| 7 | **Output locations** | Points, frames, tables | `POINTS`, `FRAME`, `GROUP`, `.loc` files |

### B. SWAN → SWASH (extra)

| # | Item | Why needed | Notes |
|---|------|------------|-------|
| 8 | **SWASH grid resolution** | SWASH has no spectral grid | Often use **bathymetry grid** (mx, my from `INPGRID BOTTOM`), not SWAN `CGRID` spectral counts — see `2bb1v4` (540×424 vs 571×442) |
| 9 | **Vertical layers** | Dispersion / nonhydrostatic | `VERT n` (example: `VERT 2`) |
| 10 | **Time integration** | SWASH is time-marching | `TIMEI` dt, CFL; `COMPUTE t0 dt tend` — **not** in SWAN file |
| 11 | **Boundary wave parameters** | Replace `BOUNDSPEC` | Hs, Tp/Tm, dir, spread from `BOUNDSPEC … PAR` or spectrum file |
| 12 | **Boundary segment geometry** | `BOUND SEGMENT XY` | Endpoints from `BOUNDSPEC SEGMENT XY`; add sponge/velocity sides per site |
| 13 | **Physics choice** | No 1:1 for GEN3/QUAD | `NONHYDROSTATIC`, `DISCRET`, optional `BRE`, `FRIC`, `SPON` |
| 14 | **Structure / porosity grids** (if applicable) | Harbour, breakwaters | `INPGRID` + `READINP` for `POROSITY`, `PSIZE`, `HSTRUCTURE` — absent in minimal SWAN case |
| 15 | **Initial condition** | Start state | `INITIAL ZERO` or field from SWAN `INITIAL` |
| 16 | **Output variables** | Phase-resolving quantities | `WATL`, `VKSI`, `VETA`, `HS` vs spectral `HS TM01 DIR` |

### C. SWASH → SWAN (extra)

| # | Item | Why needed | Notes |
|---|------|------------|-------|
| 8 | **Spectral discretization** | SWAN requires frequency/direction grid | `CGRID … CIRCLE ndir fmin fmax nf` — **must be designed**; not recoverable from `.sws` alone |
| 9 | **Generation / dissipation** | Spectral physics | `GEN3`, `WCAP`, `QUAD`, `BREAKING`, `FRIC` — choose explicitly |
| 10 | **Boundary spectra** | SWAN uses integral spectra | Map `BOUND … SPECT` → `BOUND SHAPE` + `BOUNDSPEC`; or use `SPECOUT`/`SPECFILE` workflow |
| 11 | **Propagation** | Numerics block | `PROP`, `NUMERIC` |
| 12 | **Stationarity** | SWAN often stationary | `COMPUTE` vs `COMPUTE sta` / time windows |

### D. Nesting SWAN → SWASH (alternative to full rewrite)

If the goal is **nested** SWASH inside SWAN (not full command translation):

- Same coordinate system (manual `swashuse.md` §2.6).
- SWAN: `POINTS`/`CURVE` on boundary + `SPECOUT` spectra file.
- SWASH: `BOUND` with `SPECSWAN` (see SWASH manual §4.5.3).
- Skip inventing parametric `BOUNDSPEC`-equivalent boundaries when spectra drive SWASH.

---

## Conversion rules (high level)

### SWAN → SWASH

1. **Header**: `PROJECT 'a' 'b' 'title'` → `PROJ 'a' 'b'` (titles optional in SWASH).
2. **SET**: Copy compatible keys (`depmin`, `level`, …); drop SWAN-only (`grav` defaults usually OK).
3. **CGRID**: Remove `CIRCLE`/spectral tail. Use spatial extent + **count matching bathymetry input grid** when that grid is finer than SWAN computational grid.
4. **Input**: Keep `INPGRID BOTTOM` + `READINP BOTTOM` paths; add SWASH-only grids if files exist.
5. **Boundaries**: `BOUND SHAPE JONSWAP` → `BOUND SHAPE JON`; `BOUNDSPEC SEGMENT … CONSTANT PAR hs per dir spread` → `BOUND SEGMENT … BTYPE WEAK … UNIFORM SPECT hs per …` plus short sponge segments and `BOUND SIDE … BTYPE VEL` as in reference.
6. **Remove** SWAN-only: `GEN*`, `OFF QUAD`, `WCAP`, `PROP`, `NUMERIC` (spectral), `BOUNDSPEC`, nested spectral commands.
7. **Add** SWASH numerics: `VERT`, `NONHYDROSTATIC`, `DISCRET`, `TIMEI`, `INITIAL`, `SPON` as required.
8. **Output**: Map `POINTS`/`TABLE` spectral outputs to phase outputs; add `QUANTITY` definitions; set `COMPUTE` time range.
9. **Order**: SWASH blocks (a)→(e) then flexible (f),(g),(h),(i) before (j) — see `swashuse.md` §4.2.

### SWASH → SWAN

1. **Reverse** header/grid/input where formats align.
2. **Insert** spectral `CGRID … CIRCLE …` — **ask user** for ndir, fmin, fmax, nf if not documented.
3. **Replace** `BOUND SEGMENT` / `SPECSWAN` with `BOUND SHAPE` + `BOUNDSPEC` (or nest via spectrum files).
4. **Add** `GEN3` (or chosen generation), `FRIC`, `BREAKING`, `PROP`, `NUMERIC`.
5. **Simplify** output to spectral quantities (`HS`, `TM01`, `DIR`, `RTP`) unless user needs otherwise.
6. **Warn**: time-dependent SWASH results cannot be fully represented in a single stationary SWAN setup without extra commands.

---

## Output deliverables

After conversion, always provide:

1. **Completed checklist** with user items marked.
2. **New `.swn` or `.sws`** (full file or clearly marked patch sections).
3. **Assumptions table** (grid resolution choice, dt, VERT layers, boundary type).
4. **File manifest** — external files that must sit beside the command file (unchanged paths vs renamed).
5. **Manual verification steps** — run pre-processing, check `.PRT` for boundary/grid warnings.

---

## Quality checks

- [ ] `READINP` follows each `INPGRID` for the same quantity (both manuals).
- [ ] SWASH: `TEST POINTS` only after `READINP BOTTOM` if used.
- [ ] SWASH: `STOP` after `COMPUTE`.
- [ ] SWAN: `BOUND SHAPE` before `BOUNDSPEC`.
- [ ] Boundary segment coordinates consistent with domain rotation (`alpx` in `CGRID`).
- [ ] No SWAN spectral keywords left in `.sws`; no `VERT`/`TIMEI` left in minimal `.swn` unless intentional.

---

## Additional resources

- Detailed command mapping: [mapping.md](mapping.md)
- Reference case walkthrough: [examples.md](examples.md)
