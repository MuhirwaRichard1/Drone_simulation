# VTOL Aircraft — RealFlight Model Creation Guide

Complete step-by-step workflow for converting STL CAD files into a flyable VTOL aircraft
in the RealFlight RC flight simulator, using Blender as the 3D preparation tool.

---

## Table of Contents

1. [Overview & Tools Required](#overview--tools-required)
2. [Source Files](#source-files)
3. [Step 1 — Install the Blender Add-on](#step-1--install-the-blender-add-on)
4. [Step 2 — Import STL Files into Blender](#step-2--import-stl-files-into-blender)
5. [Step 3 — Decimate Heavy Meshes](#step-3--decimate-heavy-meshes)
6. [Step 4 — Rename and Split Parts](#step-4--rename-and-split-parts)
7. [Step 5 — Set Pivot Points (Origins)](#step-5--set-pivot-points-origins)
8. [Step 6 — Build the Object Hierarchy](#step-6--build-the-object-hierarchy)
9. [Step 7 — Create the Texture](#step-7--create-the-texture)
10. [Step 8 — Export the FBX](#step-8--export-the-fbx)
11. [Step 9 — Import into RealFlight](#step-9--import-into-realflight)
12. [Step 10 — Physics Editor Setup](#step-10--physics-editor-setup)
13. [Naming Convention Reference](#naming-convention-reference)
14. [Troubleshooting](#troubleshooting)

---

## Overview & Tools Required

### What This Guide Produces
A flyable VTOL (Vertical Take-Off and Landing) fixed-wing aircraft in RealFlight simulator,
built from raw STL mesh files exported from CAD software.

### Aircraft Configuration
- Fixed-wing fuselage with left and right main wings
- Ailerons for roll control
- V-tail ruddervators for pitch and yaw control
- Lift boom arms with vertical VTOL motors for hover
- Forward propeller for cruise flight

### Tools Required

| Tool | Purpose | Download |
|------|---------|----------|
| **Blender** (3.x or 4.x) | 3D mesh preparation and export | https://www.blender.org |
| **io_scene_realflight** add-on | RealFlight FBX exporter for Blender | https://github.com/robertlong13/io_scene_realflight |
| **RealFlight Evolution** (Windows) | RC flight simulator for final import and physics | https://www.realflight.com |

### File Formats
- **Input:** `.STL` (from CAD software, units in millimetres)
- **Intermediate:** `.blend` (Blender project)
- **Output:** `.FBX` + `.TGA` (imported into RealFlight)
- **RealFlight native:** `.KEX` / `.RFX` (created automatically by RealFlight on import)

---

## Source Files

Located in: `Downloads/Real Flight STL files/`

| Filename | Part | Original Face Count | Notes |
|----------|------|-------------------|-------|
| `Real Flight Model - Fuselage Alone-1.STL` | Main body | 714,304 | Needs heavy decimation |
| `Real Flight Model - WingL-1.STL` | Left wing | 3,702 | Ready to use |
| `Real Flight Model - WingR-1.STL` | Right wing | 4,300 | Ready to use |
| `Real Flight Model - Aileron-1.STL` | Both ailerons (combined) | 270 | Must be split into L/R |
| `Real Flight Model - Tail wing-1.STL` | Tail assembly | 944 | Ready to use |
| `Real Flight Model - Ruddervators-1.STL` | Both ruddervators (combined) | 496 | Must be split into L/R |
| `Real Flight Model - Lift Boom Holders-1.STL` | Central boom mount | 476 | Ready to use |
| `Real Flight Model - Lift Booms with Motors-1.STL` | VTOL arms + motors | 451,442 | Needs heavy decimation |
| `Real Flight Model - Props-1.STL` | Propellers | 88,162 | Needs decimation |

### Raw Dimensions (after correct scale applied)

| Part | Width (X) | Depth (Y) | Height (Z) |
|------|-----------|-----------|------------|
| Fuselage | 0.45 m | 0.642 m | 1.976 m |
| Each Wing | 1.518 m span | 0.127 m | 0.427 m |
| Lift Boom Assembly | 1.307 m span | 0.055 m | 1.182 m |

---

## Step 1 — Install the Blender Add-on

The `io_scene_realflight` add-on adds a **RealFlight FBX** export option to Blender that
correctly formats the FBX file with RealFlight's required properties and scaling.

### Download
The add-on ZIP file is already downloaded at:
```
Downloads/io_scene_realflight-master.zip
```

### Installation
1. Open **Blender**
2. Go to **Edit → Preferences**
3. Click the **Add-ons** tab
4. Click **Install…** (top right)
5. Navigate to `Downloads/` and select `io_scene_realflight-master.zip`
6. Click **Install Add-on**
7. In the search box type `realflight`
8. **Tick the checkbox** next to **"Import-Export: RealFlight FBX format"**
9. Click the hamburger menu (☰) → **Save Preferences**

### Verify
Go to **File → Export** — you should see **"RealFlight FBX (.fbx)"** in the list.

---

## Step 2 — Import STL Files into Blender

### Clear the Default Scene
1. Press **A** to select everything in the default scene
2. Press **Delete** to remove the default cube, camera, and light

### Import All STL Files
1. Go to **File → Import → Stl (.stl)**
2. Navigate to `Downloads/Real Flight STL files/`
3. Press **A** to select all 9 STL files
4. Click **Import STL**

All 9 parts will appear as separate objects in the Outliner — this is correct.

### Fix the Scale (Critical)

STL files have no embedded units. Blender defaults to 1 unit = 1 metre, but the model
was designed in millimetres, making every part 1000× too large.

**Symptom:** Objects are invisible; fuselage shows as 450 m × 642 m × 1976 m.

**Fix:**
1. Press **A** to select all objects (even if invisible)
2. Press **Numpad .** (period) to zoom to fit — the objects will appear
3. With everything selected, press **S**, type **0.001**, press **Enter**
   (this divides all dimensions by 1000, converting mm → m)
4. Apply the scale: **Ctrl+A → Scale**
5. Press **Numpad .** again to re-fit the view

**Expected result after fix:**
- Fuselage: X = 0.45 m, Y = 0.642 m, Z = 1.976 m
- Each wing: ~1.5 m span

---

## Step 3 — Decimate Heavy Meshes

Three meshes have far too many polygons for RealFlight to handle efficiently.
The Decimate modifier reduces the face count while preserving overall shape.

### Target Face Counts

| Object | Original Faces | Target Faces | Decimate Ratio |
|--------|---------------|--------------|----------------|
| Fuselage Alone | 714,304 | ~5,000 | 0.007 |
| Lift Booms with Motors | 451,442 | ~4,000 | 0.009 |
| Props | 88,162 | ~2,000 | 0.023 |

### Procedure (repeat for each of the 3 meshes)

1. **Click** the mesh to select it
2. Go to **Properties panel → Modifier Properties** (wrench icon)
3. Click **Add Modifier → Decimate**
4. Leave mode as **Collapse**
5. Set the **Ratio** to the value from the table above
6. Check the **Face Count** shown in the modifier header
7. Adjust ratio up or down slightly if needed
8. Click **Apply** when satisfied

### Quality Check
After decimating, press **Numpad 1** (front view) and visually inspect each part.
The mesh should look smooth, not jagged or blocky. If quality is too low, undo
(**Ctrl+Z**), re-add Decimate with a higher ratio (e.g. 0.015 for fuselage ≈ 10,000
faces — still acceptable for RealFlight).

---

## Step 4 — Rename and Split Parts

### Part A — Split the Aileron into Left and Right

The Aileron STL contains both ailerons as one mesh. RealFlight needs them as
separate objects so each can be assigned its own servo.

1. Click the **Aileron** object
2. Press **Tab** to enter Edit Mode
3. Press **A** to deselect all vertices
4. Press **P → By Loose Parts**
5. Press **Tab** to exit Edit Mode

You now have 2 separate aileron objects. Rename them (see Part B below):
one becomes `~CS_LMA`, the other `~CS_RMA`.

**Repeat this exact procedure for Ruddervators** → becomes `~CS_LRUD` and `~CS_RRUD`.

> **If By Loose Parts produces more than 2 pieces:** Select all left-side pieces,
> press **Ctrl+J** to join them into one object, do the same for the right side.

### Part B — Rename All Objects

Double-click each object name in the **Outliner** panel to rename it.
RealFlight identifies parts by these exact names.

| Original STL Name | Blender Object Name | Purpose |
|---|---|---|
| `Fuselage Alone` | `FUSELAGE` | Root object / main body |
| `WingL` | `~CS_LMW` | Left main wing |
| `WingR` | `~CS_RMW` | Right main wing |
| Left aileron half | `~CS_LMA` | Left aileron (control surface) |
| Right aileron half | `~CS_RMA` | Right aileron (control surface) |
| `Tail wing` | `~CS_TAIL` | Tail assembly |
| Left ruddervator half | `~CS_LRUD` | Left ruddervator (V-tail control surface) |
| Right ruddervator half | `~CS_RRUD` | Right ruddervator (V-tail control surface) |
| `Lift Boom Holders` | `~CS_BOOMMOUNT` | Central VTOL boom mount |
| `Lift Booms with Motors` | `~CS_ENGINE_2` | VTOL lift motor assembly |
| `Props` | `~CS_ENGINE` | Forward propeller |

### Part C — Add Spinner Placeholders

RealFlight requires a `~CS_SPINNER` object paired with every `~CS_ENGINE` object,
or it will throw warnings during import.

Create a simple placeholder for each engine:

1. Press **Shift+A → Mesh → Cylinder**
2. Scale it small: **S → 0.05 → Enter**
3. Move it to the prop/motor location using **G**
4. Rename it `~CS_SPINNER` (paired with `~CS_ENGINE`)
5. Repeat, naming the second one `~CS_SPINNER_2` (paired with `~CS_ENGINE_2`)

---

## Step 5 — Set Pivot Points (Origins)

The pivot point (origin) of each object defines its rotation axis in RealFlight.
Getting this wrong means control surfaces rotate around the wrong point.

### Fuselage — Approximate Center of Mass

1. Click `FUSELAGE`
2. **Object menu → Set Origin → Origin to Geometry**
   (RealFlight allows fine CG adjustment in the Physics Editor later)

### Wings — Root Rib Edge

The wing must rotate/attach from where it meets the fuselage.

For `~CS_LMW` (repeat for `~CS_RMW`):
1. Click the wing, press **Tab** (Edit Mode)
2. Press **Alt+Z** (X-ray mode)
3. **Box select** only the vertices at the **root edge** (inner edge, closest to fuselage)
4. **Shift+S → Cursor to Selected**
5. Press **Tab** (exit Edit Mode)
6. **Object → Set Origin → Origin to 3D Cursor**

### Control Surfaces — Hinge Line

The hinge line is the **leading edge** of each control surface (the edge that
connects to the wing or tail structure).

For each of `~CS_LMA`, `~CS_RMA`, `~CS_LRUD`, `~CS_RRUD`:
1. Click the part, press **Tab** (Edit Mode)
2. Press **2** (Edge select mode)
3. Click the **hinge edge** (front/leading edge of the surface)
4. **Shift+S → Cursor to Selected**
5. Press **Tab** (exit Edit Mode)
6. **Object → Set Origin → Origin to 3D Cursor**

### Forward Propeller — `~CS_ENGINE`

1. Click `~CS_ENGINE`
2. **Object → Set Origin → Origin to Geometry**
3. Press **N** to open the side panel and verify the origin is centered on the prop disk
4. **The local Y-axis must point forward** (toward the nose).
   If it points the wrong way: **R → Y → 90 → Enter**

### VTOL Motors — `~CS_ENGINE_2`

1. Click `~CS_ENGINE_2`
2. **Object → Set Origin → Origin to Geometry**
3. **The local Y-axis must point upward** (toward sky / thrust direction).
   If it points the wrong way: **R → X → 90 → Enter**

### Spinners

For each spinner (`~CS_SPINNER`, `~CS_SPINNER_2`):
1. Click the spinner
2. **Object → Set Origin → Origin to Geometry**
3. Verify local Y-axis matches its paired engine

### Final Scale Check

With any object selected, press **N** and check that **Scale = (1.0, 1.0, 1.0)**
on all axes. If not: **Ctrl+A → Scale** to apply.

---

## Step 6 — Build the Object Hierarchy

All objects must be parented into a tree with `FUSELAGE` as the root.
When the fuselage moves, everything moves with it. Control surfaces rotate
relative to their parent.

**Rule:** Select **child first**, then **Shift+click parent last**, then
**Ctrl+P → Object → Keep Transform**.

### Hierarchy to Build

```
FUSELAGE
├── ~CS_LMW                  (left wing → child of fuselage)
│   └── ~CS_LMA              (left aileron → child of left wing)
├── ~CS_RMW                  (right wing → child of fuselage)
│   └── ~CS_RMA              (right aileron → child of right wing)
├── ~CS_TAIL                 (tail → child of fuselage)
│   ├── ~CS_LRUD             (left ruddervator → child of tail)
│   └── ~CS_RRUD             (right ruddervator → child of tail)
├── ~CS_BOOMMOUNT            (boom mount → child of fuselage)
│   ├── ~CS_ENGINE_2         (VTOL motors → child of boom mount)
│   │   └── ~CS_SPINNER_2   (VTOL spinner → child of VTOL engine)
├── ~CS_ENGINE               (forward prop → child of fuselage)
│   └── ~CS_SPINNER          (forward spinner → child of forward engine)
```

### Parenting Commands in Order

```
~CS_LMW      → parent to → FUSELAGE
~CS_RMW      → parent to → FUSELAGE
~CS_LMA      → parent to → ~CS_LMW
~CS_RMA      → parent to → ~CS_RMW
~CS_TAIL     → parent to → FUSELAGE
~CS_LRUD     → parent to → ~CS_TAIL
~CS_RRUD     → parent to → ~CS_TAIL
~CS_BOOMMOUNT → parent to → FUSELAGE
~CS_ENGINE_2  → parent to → ~CS_BOOMMOUNT
~CS_SPINNER_2 → parent to → ~CS_ENGINE_2
~CS_ENGINE    → parent to → FUSELAGE
~CS_SPINNER   → parent to → ~CS_ENGINE
```

### Verify in Outliner
Click the arrow next to `FUSELAGE` to expand — all objects should be nested
underneath it in the correct tree structure shown above.

To fix a wrong parent: right-click the object in Outliner → **Parent → Clear Parent
(Keep Transform)** then redo the correct parenting.

---

## Step 7 — Create the Texture

RealFlight requires a `.TGA` texture file with the **exact same base name** as the
exported FBX file (e.g. `MyVTOL.fbx` + `MyVTOL.tga`).

### Create the Image

1. Switch to the **UV Editing** workspace (top tab bar)
2. In the left UV Editor panel, click **New**
3. Set:
   - Name: `MyVTOL`
   - Width: `2048`, Height: `2048`
   - Color: your desired base aircraft color
4. Click **OK**

### UV Unwrap All Parts

1. In the 3D Viewport (right panel), press **A** to select all objects
2. Press **Tab** to enter Edit Mode
3. Press **A** to select all faces
4. **UV menu → Smart UV Project** → click **OK**
5. Press **Tab** to exit Edit Mode

### Assign Texture to All Parts

1. Switch back to the **Layout** workspace
2. Select each object and go to **Properties → Material (sphere icon)**
3. Click **New**, then set **Base Color → Image Texture**
4. Select `MyVTOL` from the image selector

### Save as TGA

1. In the **UV Editing** workspace, go to **Image → Save As**
2. Set filename to `MyVTOL.tga`
3. Save to `Downloads/RealFlight_Export/`

> The texture does not need to be detailed at this stage. RealFlight has a
> built-in paint editor for detailed livery work after the model is imported.

---

## Step 8 — Export the FBX

### Pre-Export Check

1. Press **A** to select all objects
2. Press **Ctrl+A → All Transforms** to apply all transforms

### Export Settings

1. **File → Export → RealFlight FBX (.fbx)**
2. Navigate to `Downloads/RealFlight_Export/`
3. Filename: **`MyVTOL.fbx`** (must match the TGA name)
4. Set these options in the right panel:

| Option | Value |
|--------|-------|
| Selected Objects | OFF |
| Scale | 1.0 |
| Apply Scalings | FBX All |
| Forward | Y Forward |
| Up | Z Up |

5. Click **Export RealFlight FBX**

### Verify Output

Your export folder should contain exactly:
```
Downloads/RealFlight_Export/
├── MyVTOL.fbx
├── MyVTOL.tga
└── README.md  (this file)
```

---

## Step 9 — Import into RealFlight

1. Open **RealFlight Evolution**
2. Go to **Simulation → Import → 3D Model (FBX, KEX)**
3. Select `MyVTOL.fbx`
4. When prompted for aircraft type: select **Drone** (VTOL)
5. Choose a simple existing VTOL as the base physics template
   (e.g. the built-in **Convergence VTOL** or similar quadplane)
6. When asked about color schemes: **Skip**
7. Click **OK** to overwrite and complete the import

RealFlight converts the FBX to its internal KEX format automatically.

---

## Step 10 — Physics Editor Setup

Access the Physics Editor via **Aircraft → Edit Current Aircraft**.

There are 3 tabs. Work through them in order: **Radio → Electronics → Physics**.

> **Save frequently** — RealFlight's Aircraft Editor can crash during complex edits.
> Use **Aircraft → Save** after every successful change.

---

### Tab 1 — Radio

This defines your transmitter channels and their ranges.

1. Click the **Radio** tab
2. Set **Number of Channels** to **10** (or more)
3. Configure each channel:

| Channel | Function | Notes |
|---------|----------|-------|
| 1 | Aileron | Roll control |
| 2 | Elevator | Pitch via ruddervators |
| 3 | Throttle | Forward motor |
| 4 | Rudder | Yaw via ruddervators |
| 5 | VTOL Motor 1 | Front-left lift |
| 6 | VTOL Motor 2 | Front-right lift |
| 7 | VTOL Motor 3 | Rear-left lift |
| 8 | VTOL Motor 4 | Rear-right lift |
| 9 | VTOL/FW Transition | Switch between hover and cruise |
| 10 | Flaps (optional) | — |

4. For **every channel**: set **Exponential = 0%**, **Low Rates = 100%**, **High Rates = 100%**

---

### Tab 2 — Electronics

#### Add Control Surface Servos

Click **Add Component → Servo** for each control surface:

| Servo | Input Channel | Assigned Object | Notes |
|-------|-------------|----------------|-------|
| Aileron Left | CH 1 | `~CS_LMA` | |
| Aileron Right | CH 1 | `~CS_RMA` | Tick **Reverse** |
| Ruddervator Left | CH 2 + CH 4 mixed | `~CS_LRUD` | Enable V-Tail mixing |
| Ruddervator Right | CH 2 + CH 4 mixed | `~CS_RRUD` | Tick **Reverse** |

For each servo:
- Set **Travel** to **±30°** as a starting point (fine-tune during test flights)
- Tick **Reverse** where noted so both sides deflect symmetrically

#### Add ESCs and Motors

Click **Add Component → Electric Motor + ESC** for each motor:

| ESC | Input Channel | Assigned Object | Thrust Direction |
|-----|-------------|----------------|-----------------|
| Forward Motor | CH 3 | `~CS_ENGINE` | Forward (Y+) |
| VTOL Motor 1 | CH 5 | `~CS_ENGINE_2` | Upward (Z+) |
| VTOL Motor 2 | CH 6 | `~CS_ENGINE_2` | Upward (Z+) |
| VTOL Motor 3 | CH 7 | `~CS_ENGINE_2` | Upward (Z+) |
| VTOL Motor 4 | CH 8 | `~CS_ENGINE_2` | Upward (Z+) |

For each motor:
- **Motor Type:** Electric Brushless
- **KV Rating:** 900 KV (adjust to match your real motor specs)
- **Battery Cells:** match your intended battery (e.g. 4S = 4 cells)

> All 4 VTOL motors are assigned to `~CS_ENGINE_2` because they are currently
> one combined mesh. To assign them individually, go back to Blender, separate
> the mesh into 4 objects named `~CS_ENGINE_2` through `~CS_ENGINE_5`, and re-export.

---

### Tab 3 — Physics

#### 3A — Fuselage

- Click **FUSELAGE** in the parts list
- Set **Mass** to your aircraft's total weight (e.g. 2.5 kg)
- Set **Dimensions:** X = 0.45 m, Y = 1.976 m, Z = 0.642 m
- Set **CG Position:** move the marker to approximately **1/3 from the nose**

#### 3B — Wings

For each wing (`~CS_LMW`, `~CS_RMW`):

| Property | Value |
|----------|-------|
| Span | 1.518 m |
| Chord | ~0.35 m (measure from your model) |
| Airfoil | Flat-bottom or semi-symmetric |
| Incidence Angle | 0° (starting point) |
| Position offset | Left wing: negative X / Right wing: positive X |

#### 3C — Control Surfaces

For each control surface (`~CS_LMA`, `~CS_RMA`, `~CS_LRUD`, `~CS_RRUD`):
- Set **Deflection Range:** ±25° for ailerons, ±30° for ruddervators
- Check the 3D preview — if the surface deflects the wrong way, tick **Reverse**
- For ruddervators: enable **V-Tail Mixing** — this automatically combines
  elevator (CH 2) and rudder (CH 4) inputs into the V-tail surfaces

#### 3D — VTOL Motors

For each VTOL motor, set the correct position relative to the CG:

| Motor | X offset | Y offset |
|-------|----------|----------|
| Front-left | −boom span | +forward offset |
| Front-right | +boom span | +forward offset |
| Rear-left | −boom span | −rear offset |
| Rear-right | +boom span | −rear offset |

- **Thrust Direction:** Upward (Z+) for all VTOL motors
- **Max Thrust per motor:** at least 1.5× (total weight ÷ number of motors)
  e.g. for 2.5 kg aircraft: 2500 g ÷ 4 motors × 1.5 = ~940 g per motor minimum

#### 3E — Forward Motor

- **Thrust Direction:** Forward (Y+)
- **Position:** at the nose (tractor) or tail (pusher) depending on your design
- **Max Thrust:** sufficient for cruise flight (typically 30–50% of aircraft weight)

#### 3F — Transition Setup

- Find the **Flight Mode** or **Transition** setting in the Physics tab
- Assign **CH 9** as the transition switch input
- Set **Transition Speed:** 15–20 m/s (airspeed at which fixed-wing lift becomes effective)
- Below transition speed: VTOL motors active; above: fixed-wing surfaces take over

---

### First Flight Checklist

Work through this in order — do not attempt the next step until the current one is stable:

```
1. [ ] Hover test    — throttle up gently in VTOL mode, check for stability
2. [ ] Flip check    — if aircraft flips, reverse that VTOL motor's ESC direction
3. [ ] Drift check   — if it drifts, adjust motor position offsets to correct CG
4. [ ] Surface check — verify ailerons and ruddervators deflect correctly
5. [ ] FW test       — gain speed in VTOL mode, then flip CH 9 to transition
6. [ ] Transition    — confirm smooth handover from VTOL to fixed-wing at ~15 m/s
7. [ ] Fine-tune     — adjust CG, deflections, and motor power to taste
```

### Suggested Starting Parameters

| Parameter | Starting Value |
|-----------|---------------|
| Aileron deflection | ±25° |
| Ruddervator deflection | ±30° |
| VTOL motor KV | 900 KV |
| Forward motor KV | 700 KV |
| Battery | 4S LiPo |
| CG position | 33% from nose |
| Transition speed | 15 m/s |

---

## Naming Convention Reference

| Prefix / Name | Meaning |
|---|---|
| `FUSELAGE` | Root object — no prefix needed |
| `~CS_` | Any moving, breakable, or engine part |
| `~CS_LMW` / `~CS_RMW` | Left / Right Main Wing |
| `~CS_LMA` / `~CS_RMA` | Left / Right Main Aileron |
| `~CS_TAIL` | Tail assembly |
| `~CS_LRUD` / `~CS_RRUD` | Left / Right Ruddervator |
| `~CS_ENGINE` | Primary engine/prop (forward motor) |
| `~CS_ENGINE_2…N` | Additional engines (VTOL lift motors) |
| `~CS_SPINNER` | Spinner paired with `~CS_ENGINE` |
| `~CS_SPINNER_2…N` | Spinners paired with additional engines |
| `~CS_COLL_` | Collision mesh prefix |
| `~CS_LG` / `~CS_RG` | Left / Right landing gear |
| `~CS_LW` / `~CS_RW` | Left / Right wheel |
| `~ALPHA` | Material uses alpha transparency |
| `~CANOPY` | Transparent canopy material |

---

## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| Objects invisible after STL import | Scale is 1000× too large | Select all → S → 0.001 → Enter → Ctrl+A → Scale |
| RealFlight shows "missing spinner" warning | No `~CS_SPINNER` paired with engine | Add a small cylinder named `~CS_SPINNER` as child of `~CS_ENGINE` |
| Control surface rotates around wrong axis | Pivot not on hinge line | Re-do Step 5 for that control surface |
| FBX import fails in RealFlight | TGA not in same folder as FBX | Move both files to the same directory |
| Model appears inside-out in RealFlight | Inverted face normals | In Blender Edit Mode: select all → Mesh → Normals → Recalculate Outside |
| VTOL motors spin sideways | Y-axis not pointing upward | In Blender: select `~CS_ENGINE_2` → R → X → 90 → Enter, re-export |
| Model too large / too small in RealFlight | Wrong scale at export | Re-export with Scale = 1.0, or re-apply 0.001 scale fix in Blender |
| RealFlight crashes in Physics Editor | Complex model overwhelms editor | Save after every change; restart RealFlight and continue |
| Parts detach from fuselage in sim | Missing parent-child relationship | Re-do Step 6 parenting for the detaching part |

---

## References

- [RealFlight Forum: Box Plane Example Workflow](https://forums.realflight.com/index.php?threads/update-the-box-plane-example.33629/)
- [ArduPilot Discourse: Creating a RealFlight Model with Blender](https://discuss.ardupilot.org/t/creating-a-realflight-model-with-blender/116967)
- [RealFlight Quick Reference: Object Naming Convention](https://forums.realflight.com/index.php?threads/quick-reference-for-object-naming-convention.28693/)
- [robertlong13 Blender Add-on (GitHub)](https://github.com/robertlong13/io_scene_realflight)
- [Making VTOL Plane in RealFlight (Forum)](https://forums.realflight.com/index.php?threads/making-vtol-plane-in-realflight.60507/)
