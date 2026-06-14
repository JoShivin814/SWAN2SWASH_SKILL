# SWAN ↔ SWASH command mapping

Abbreviations: **≈** direct carry-over, **Δ** must change syntax, **✗** no counterpart (drop or redesign), **+** add in target model.

## Start-up (block a)

| SWAN | SWASH | Notes |
|------|-------|-------|
| `PROJECT` | `PROJ` | Same role; SWASH shorter keyword |
| `SET` | `SET` | ≈ shared params (`depmin`, `level`, …) |
| `MODE` | `MODE` | ≈ 1D/2D |
| `COORD` / `COORDINATES` | `COORD` | Must stay consistent for nesting |

## Grid (block b)

| SWAN | SWASH | Notes |
|------|-------|-------|
| `CGRID REG … mx my CIRCLE …` | `CGRID … mx my` | **Δ** drop spectral `CIRCLE` block |
| `READGRID` | `READGRID` | ≈ curvilinear/unstructured |
| — | `VERT n` | **+** SWASH vertical layers (required for multi-layer runs) |

**Resolution rule (from `2bb1v4`):** SWAN `CGRID` mx×my (540×424) ≠ SWASH `CGRID` (571×442); SWASH matches `INPGRID BOTTOM` dimensions.

## Input fields (block c)

| SWAN | SWASH | Notes |
|------|-------|-------|
| `INPGRID BOTTOM` | `INPGRID BOTTOM` | ≈ |
| `READINP BOTTOM` | `READINP BOTTOM` | ≈ same file often reusable |
| `INPGRID WLEV/CUR/VFRIC/…` | `INPGRID` + matching `READINP` | Δ quantity names; check manual |
| `WIND` (inline) | `WIND` | ≈ similar role |
| — | `INPGRID POROSITY` / `PSIZE` / `HSTRUCTURE` | **+** common in harbour SWASH cases |
| — | `INPTRAN` / `READTRA`, `INPAMB` / `READAMB` | **+** transport / ambient current |

## Boundaries (block d)

| SWAN | SWASH | Notes |
|------|-------|-------|
| `BOUND SHAPE JONSWAP PEAK … POWER` | `BOUND SHAPE JON PEAK … POWER` | **Δ** keyword spelling |
| `BOUNDSPEC SEGMENT XY … CONSTANT PAR hs per dir dd` | `BOUND SEGMENT XY … BTYPE WEAK … UNIFORM SPECT hs per …` | **Δ** see `examples.md` |
| `BOUNDSPEC` + spectrum file | `BOUND` + `SPECfile` / `SPECSWAN` | Nesting path |
| `BOUNDNEST*` | `SPECSWAN` | SWASH reads SWAN `SPECOUT` format |
| — | `BOUND SIDE … BTYPE VEL UNIFORM FOURIER` | **+** open boundaries |
| — | `SPON` / `SPONGE LAYER` | **+** absorption |
| `INITIAL` | `INITIAL` | ≈ `ZERO` / `PAR` differ by manual |

## Physics (block e)

| SWAN | SWASH | Notes |
|------|-------|-------|
| `GEN1`/`GEN2`/`GEN3`, `OFF QUAD` | — | **✗** spectral source terms |
| `WCAP`, `QUAD`, `TRIAD` | — | **✗** |
| `BREAKING` | `BRE` / breaking-related | **Δ** different formulation |
| `FRIC JONSWAP cf` | `FRIC` | **Δ** verify law & coefficient |
| `FRICTION` (SWAN) | `FRIC` | |
| — | `NONHYDROSTATIC` | **+** typical SWASH default |
| — | `DISCRET UPW …` | **+** advection scheme |
| — | `TIMEI` / `TIME INT` | **+** time stepping |
| `VEGETATION` | `VEGET` | ≈ |
| `MUD` etc. | check manual | case-by-case |

## Output (blocks g–h)

| SWAN | SWASH | Notes |
|------|-------|-------|
| `POINTS 'name' FILE 'x.loc'` | `POINTS 'name' x y` or FILE | **Δ** SWASH inline coords common |
| `FRAME` / `GROUP` / `CURVE` | same family | ≈ |
| `QUANTITY` | `QUANTITY` | **Δ** quantity names differ |
| `TABLE … HS TM01 DIR` | `TABLE … WATL VKSI VETA` / `HS` | **Δ** match user goal |
| `BLOCK … HS` | `BLOCK` / `FRAME` tables | ≈ structure similar |
| `SPECOUT` | drives `SPECSWAN` | one-way coupling |

## Execute (block j)

| SWAN | SWASH | Notes |
|------|-------|-------|
| `COMPUTE` (stationary) | `COMPUTE t0 dt tend` | **Δ** SWASH needs time window |
| `HOTFILE` | — | **✗** in SWASH |
| `STOP` | `STOP` | ≈ |

## Command block order (both manuals §4.2)

Sequence: **(a)→(b)→(c)→(d)→(e)→(g)→(h)→(j)** with **(f)** and **(i)** flexible before **(j)**.

SWAN-specific inside blocks: `BOUND SHAPE` before `BOUNDSPEC`; one `GEN` active.

SWASH-specific: `READGRID` after `CGRID`; `VERT` after `CGRID`; `TEST POINTS` after `READINP BOTTOM`.
