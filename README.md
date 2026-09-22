# TOF Coverage Calculator

A single-page web calculator for a **ceiling-mounted Time-of-Flight (TOF) sensor**: it works out the
floor area the sensor covers, and — more usefully — the smaller area over which a *standing person of
a given height* is actually detected.

No build step, no dependencies. Open `index.html` in a browser, or serve it from GitHub Pages.

## The problem

A TOF lens has two field-of-view angles and casts a rectangular pyramid of IR down onto the floor,
producing a rectangular detection area. But a person standing at the edge of that rectangle is not
detected: their **head** is the topmost point, and at head height the cone cross-section is far
narrower than it is at floor level. The rectangle you can rely on is therefore the cross-section at
head height, not the floor footprint.

This calculator quantifies that gap and draws it.

## Inputs

| Input | Symbol | Unit |
|---|---|---|
| Sensor FOV, width direction | α | degrees |
| Sensor FOV, length direction | β | degrees |
| Ceiling / sensor lens height above floor | H | metres |
| Target human height | h | metres |

## Outputs

* Full floor coverage — width × length, and area
* Usable coverage for height `h` — width × length, area, and percentage of full
* Blind margin lost on every side, in both directions
* Head clearance `H − h`, usable diagonal, and slant range to the floor corner
* Three scaled, fully dimensioned diagrams:
  * **Side elevation, width direction** — cone, FOV angle α, `H`, `h`, `W_floor`, `W_usable`, the
    blind margins, a person detected with their head exactly on the FOV edge, and a person just
    outside it who is missed
  * **Side elevation, length direction** — the same in the β plane
  * **Plan view** — the full floor rectangle, the usable rectangle, a 1 m grid, and people on both
    the width and length edges

Every diagram can be downloaded as a standalone SVG (theme colours baked in), and the whole page
prints cleanly. Results copy to the clipboard as plain text, and the current configuration is
encoded in the URL so a set of parameters can be shared as a link:

```
index.html#a=60&b=45&H=2.8&h=1.75
```

## Formulas

Lens treated as a point source at height `H`, aimed straight down, with a rectangular pyramid FOV.

Floor coverage at the ground plane:

```
W_floor = 2 · H · tan(α / 2)
L_floor = 2 · H · tan(β / 2)
```

A standing person's highest point is the top of the head at `z = h`. The cone cross-section shrinks
linearly with height, so the head is inside the FOV only while its horizontal offset from the sensor
axis stays within the cross-section at that height:

```
W_usable = 2 · (H − h) · tan(α / 2)
L_usable = 2 · (H − h) · tan(β / 2)
```

Since the feet sit directly below the head, that rectangle is also the usable floor footprint:

```
W_usable / W_floor = L_usable / L_floor = (H − h) / H
A_usable / A_floor = ((H − h) / H)²
```

Blind margin lost on each side:

```
margin_width  = (W_floor − W_usable) / 2 = h · tan(α / 2)
margin_length = (L_floor − L_usable) / 2 = h · tan(β / 2)
```

Note that the blind margin depends only on the person's height and the FOV — **not** on the ceiling
height. Raising the sensor grows the floor rectangle but the lost ring around it keeps the same
width.

### Worked example

α = 60°, β = 45°, H = 2.80 m, h = 1.75 m

| Quantity | Value |
|---|---|
| `W_floor` | 3.23 m |
| `L_floor` | 2.32 m |
| `A_floor` | 7.50 m² |
| `W_usable` | 1.21 m |
| `L_usable` | 0.87 m |
| `A_usable` | 1.05 m² (14.1 % of full) |
| Blind margin | 1.01 m (width) / 0.72 m (length) |

## Assumptions and caveats

* Sensor points straight down; tilt skews the footprint and is not modelled.
* FOV is a rectangular pyramid, as quoted on most multi-zone TOF datasheets. A circular-cone lens
  would inscribe a smaller rectangle.
* Detection is treated as geometric line of sight to the top of the head. Real detection also needs
  sufficient return signal, and the range needed at a corner is the slant distance, which is longer
  than `H`.
* Zone resolution is ignored: with a coarse zone grid the outermost zone may straddle the boundary,
  so keep design margin.
* A shorter person is covered over a larger area, a taller person over a smaller one. Size for the
  tallest occupant that must be caught.
* Lens distortion and any mounting recess are not modelled — enter the true lens height as `H`.

## Publishing

The page is fully self-contained, so GitHub Pages needs nothing more than being pointed at the
default branch root: **Settings → Pages → Source: deploy from branch → `main` / `/ (root)`**.
