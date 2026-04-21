# Sliced Model Shader — Three.js Journey

Quick recap of the **Sliced Model Shader** lesson from [Three.js Journey](https://threejs-journey.com/) by Bruno Simon.

## What this project covers

This project demonstrates how to create a procedural "slice" effect on a 3D model using custom GLSL code while keeping physically based rendering from Three.js materials. It uses `three-custom-shader-material` to inject shader logic into standard materials, so the model keeps realistic lighting and shadows while part of its surface is dynamically cut away in screen-time.

- **Angular slicing in the fragment shader** using `atan`, angle normalization, and `discard`.
- **Interactive slice controls** with `lil-gui` (`uSliceStart`, `uSliceArc`) to art-direct the cut in real time.
- **Custom shader injection** on top of `MeshStandardMaterial` via `CustomShaderMaterial`.
- A **custom depth material** using the same shader logic so shadows stay consistent with sliced geometry.
- A **backface tint trick** via `patchMap` to color the inside/cut-facing area differently.
- A full scene setup with **HDR environment lighting**, loaded GLTF model, and soft shadows.

## What I built

- Set up a Three.js scene with camera, orbit controls, tone mapping, and shadow-enabled renderer.
- Loaded `gears.glb` with `GLTFLoader` and `DRACOLoader`.
- Applied two materials by mesh name:
  - `outerHull` -> sliced custom shader material
  - other meshes -> standard `MeshStandardMaterial`
- Added slice uniforms:
  - `uSliceStart` to rotate where the cut begins
  - `uSliceArc` to control how much of the model is removed
- In the vertex shader:
  - passed `csm_Position.xyz` to the fragment shader as `vPosition`
- In the fragment shader:
  - computed polar angle from `vPosition`
  - shifted and wrapped that angle
  - discarded fragments inside the selected angular arc
- Added a custom depth material with the same shader so cast shadows match the visible sliced result.
- Used `patchMap` + `gl_FrontFacing` to paint backfaces in a distinct color and emphasize the cut.

## What I learned

### 1) How to extend standard Three.js materials without losing PBR

- `three-custom-shader-material` lets me inject custom GLSL logic into existing Three.js materials.
- I keep familiar lighting behavior from `MeshStandardMaterial` while adding procedural effects.
- This is often faster and safer than rewriting a full `ShaderMaterial`.

### 2) How fragment-level slicing works

- `atan(y, x)` gives an angular coordinate around the model.
- Offsetting and wrapping the angle with `mod` makes the slice controllable and stable.
- `discard` removes fragments directly in the fragment shader for a clean cutout effect.

### 3) Why local/object space position is useful in shaders

- Passing `csm_Position` from vertex to fragment gives spatial data for procedural masks.
- Position-driven logic is great for effects like slicing, gradients, and directional reveals.
- This pattern is reusable for many non-UV based shader effects.

### 4) Why custom depth materials matter for shader effects

- If only the visible material is modified, shadows can become incorrect.
- Using a matching custom depth material keeps shadow maps aligned with the rendered surface.
- This is key for believable real-time lighting when using `discard` or deformation.

### 5) How to treat front and back faces differently

- `gl_FrontFacing` allows conditional styling based on face orientation.
- Coloring backfaces helps communicate thickness and makes cut surfaces readable.
- `patchMap` is a practical way to inject this behavior into the base material pipeline.

### 6) Practical workflow gains for shader iteration

- `lil-gui` speeds up experimentation by exposing uniforms in real time.
- HDR environments quickly improve material feedback and reflections.
- Keeping controls interactive makes shader debugging and look-dev much easier.

## Run the project

```bash
npm install
npm run dev
```

## Credits

Part of the **Three.js Journey** course by Bruno Simon.
