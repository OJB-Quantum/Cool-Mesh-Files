# Cool-Mesh-Files
A collection of cool and often useful geo files and other mesh files for use in Gmsh. Created by Onri Jay Benally.

Below is the primary website for the open source mesh generation tool known as Gmsh: <https://gmsh.info>

----

# SVG/PNG → Gmsh `.geo` Generator (2D + 3D, Edge/Face-Refined Mesh Density)

This repository provides a Google Colab–friendly Python script that converts either:

- **SVG (Scalable Vector Graphics)** files, or
- **PNG (Portable Network Graphics)** images with a transparent background (alpha channel)

into a **Gmsh `.geo`** geometry script, while also configuring a **mesh density strategy** that is:

- **finer near boundaries** (2D edges / 3D outer faces), and
- **coarser away from boundaries** (toward 2D interiors / 3D volume interiors).

The workflow is designed for multiphysics use cases (electrostatics, thermal, mechanics, CFD, lithography mask simulation) where boundary fidelity typically dominates accuracy.

---

## How mesh density is controlled

### 2D: Edge-refined sizing (distance to boundary curves)

In **2D mode**, the script:
1. Builds one or more **Plane Surfaces** from the extracted silhouette polygons.
2. Collects all **boundary curves** with:
   - `bndCurves[] = Boundary{ Surface{...}; };`
3. Applies a **background mesh-size field** driven by distance to those curves:

- `Field[1] = Distance;` with `Field[1].CurvesList = {bndCurves[]};`
- `Field[2] = Threshold;` mapping distance to element size:
  - near edges: `SizeMin = lcMin`
  - away from edges: `SizeMax = lcMax`
  - transition region controlled by `DistMin` and `DistMax`
- `Background Field = 2;`

Result: triangles become **smaller near edges** and **larger in the interior**, as permitted by the geometry.

---

### 3D: Face-refined sizing (distance to boundary surfaces)

In **3D mode**, the script:
1. Extrudes the 2D surfaces into **volumes** using `Extrude {0, 0, thickness} { Surface{...}; };`
2. Collects the **outer boundary surfaces** of the resulting volume set:
   - `bndSurfaces[] = Boundary{ Volume{vols[]}; };`
3. Applies the same **Distance → Threshold** sizing pattern, except driven by **distance to surfaces**:

- `Field[1] = Distance;` with `Field[1].SurfacesList = {bndSurfaces[]};`
- `Field[2] = Threshold;` mapping distance to size
- `Background Field = 2;`

Result: elements become **denser near outer faces** (the “skin”) and **coarser deeper inside the volume**, provided the volume has enough thickness for a meaningful distance-to-face gradient.

---

## Key “control knobs” for mesh density

The script exposes the main mesh-density parameters in a configuration object:

- `lc_min_mm` — target element size near edges/faces (fine)
- `lc_max_mm` — target element size away from edges/faces (coarse)
- `dist_min_mm` — distance from the boundary where the mesh stays at `lc_min_mm`
- `dist_max_mm` — distance from the boundary where the mesh reaches `lc_max_mm`
- `distance_sampling` — sampling density for the distance field evaluation (higher can improve robustness on complex boundaries)

### Practical tip for thin 3D extrusions
If extrusion thickness is `t`, then the deepest interior is only about `t/2` from a boundary face. To ensure the mesh can actually reach `lc_max_mm` in the interior, a good rule of thumb is:

- `dist_max_mm <= t/2`

---

## Disabling competing sizing sources (field-dominant behavior)

To make the background field dominate the mesh size, the script disables other automatic size contributions:

- `Mesh.MeshSizeExtendFromBoundary = 0;`
- `Mesh.MeshSizeFromPoints = 0;`
- `Mesh.MeshSizeFromCurvature = 0;`

This helps ensure that the Distance/Threshold strategy is the primary driver of the final element sizes.

---

## Acronym glossary

- **SVG** — Scalable Vector Graphics  
- **PNG** — Portable Network Graphics  
- **Gmsh** — Mesh generator and geometry scripting tool  
- **GEO** — Gmsh geometry script file (`.geo`)  
- **DPI** — Dots Per Inch  
- **CFD** — Computational Fluid Dynamics  
