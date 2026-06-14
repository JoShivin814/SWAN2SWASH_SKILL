# Reference case: `swan2swash_ex/2bb1v4`

Minimal SWAN harbour spectral run → full SWASH phase-resolving setup. Use as the default pattern when the user’s SWAN file resembles this structure.

## Source SWAN (`2bb1v4.swn`) — key lines

```
CGRID REG 0.675 0 0 1798.65 1392.3 540 424 CIRCLE 36 0.04 1.0 34
INPGRID BOTTOM REG 0.675 0 0 571 442 3.15 3.15 EXC -99
READINP BOTTOM 45. '2BB1v4_bot_gridmodify4i.bot' 4 0 FREE
BOUNDSPEC SEGMENT XY 45.0 0.0 1755.0 0.0 CONSTANT PAR 4.7556 9.9952 90.0 2.0
GEN3 / OFF QUAD / FRIC JONSWAP 0.01 / BREAKING
POINTS 'buoy' FILE '2bb1v4.loc'
TABLE … HS TM01 DIR RTP
COMPUTE / STOP
```

## Target SWASH (`2bb1v4.sws`) — what changed

### Grid

| Item | SWAN | SWASH |
|------|------|-------|
| Horizontal count | 540 × 424 (`CGRID`) | **571 × 442** (matches `INPGRID BOTTOM`) |
| Spectral | `CIRCLE 36 0.04 1.0 34` | removed |
| Vertical | — | `VERT 2` |

### Extra input grids (not in SWAN)

```
INPGRID POROSITY … + READINP POROSITY … '2bb1v4_por_gridmodify4d.por'
INPGRID PSIZE …     + READINP PSIZE …     '2bb1v4_psiz_gridmodify4d.psiz'
INPGRID HSTRUCTURE …+ READINP HSTRUCTURE …'2bb1v4_hstr_gridmodify4i.hstr'
```

User must supply these files for equivalent physics; omit blocks only if structures are absent.

### Boundaries

SWAN single spectral segment:

```
BOUNDSPEC SEGMENT XY 45.0 0.0 1755.0 0.0 CONSTANT PAR 4.7556 9.9952 90.0 2.0
```

SWASH decomposition:

```
BOUND SEGMENT XY 0.675 0 45 0 BTYPE VEL UNIFORM FOURIER 0 0 0 0
BOUND SEGMENT XY 45 0 1755 0 BTYPE WEAK SMOO 1 SEC UNIFORM SPECT 4.7556 9.9952
BOUND SEGMENT XY 1755 0 1799.325 0 BTYPE VEL UNIFORM FOURIER 0 0 0 0
BOUND SIDE E CCW … / BOUND SIDE W CCW …
SPON N 37.8
```

Pattern: short velocity sponge segments at corners + main **WEAK** absorbing segment carrying **SPECT** parameters from SWAN `PAR`.

### Physics / numerics

Removed: `GEN3`, `OFF QUAD`, `BREAKING` (spectral).

Added:

```
INITIAL ZERO
NONHYDROSTATIC
DISCRET UPW MOM
DISCRET UPW WMOM
TIMEI 0.2 0.5
COMPUTE 000000.000 0.01 SEC 002953.000
```

`TIMEI` and `COMPUTE` end time are **user/site specific** — not derivable from SWAN file.

### Output

SWAN: one buoy from `.loc` file, spectral tables.

SWASH: 27 inline `POINTS`, dual `FRAME`s, `QUANTITY` defs, time series `WATL VKSI VETA` + separate `HS` tables — scale down for new projects unless user needs dense monitoring.

## Checklist items satisfied by this case

- [x] Bathymetry `.bot`
- [x] Domain & Δx from `INPGRID`
- [x] Boundary Hs/Tp/dir from `BOUNDSPEC PAR`
- [x] Porosity / grain / structure grids (SWASH-only files)
- [x] `VERT`, `TIMEI`, `COMPUTE` window defined
- [ ] Spectral grid — N/A for SWAN→SWASH

## SWASH → SWAN (reverse) for this case

Not uniquely invertible: would need new `CIRCLE` specification, `GEN3`, `BOUNDSPEC` replacing `BOUND SEGMENT`/`SPON`, and reduction of output points. Document assumed `CIRCLE 36 0.04 1.0 34` from original SWAN when reversing.
