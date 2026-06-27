**Testrender — Test Runner Overview**

- **Purpose:** Lightweight test runner that renders scenes described in XML to produce images used for regression testing.
- **Where to look:** Example shader: [testsuite/render-shaderball/neutral.osl](testsuite/render-shaderball/neutral.osl).

**XML Scene Format (basic)**

- **Root:** `<scene>` encloses the whole description.
- **Camera:** `<camera>` element sets position, target, and projection.
- **Integrator / renderer settings:** `<integrator>` (type, samples, etc.).
- **Objects:** `<object>` or `<mesh>` entries reference geometry files and transforms.
- **Shaders:** `<shader name="..." file="...">` attaches materials to objects; parameters are nested elements or attributes.
- **Outputs:** `<output name="beauty" file="out.exr" format="exr"/>` defines image outputs and expected filenames.
- **Reference:** A `<reference file="...">` element or attribute points to the golden image used for comparisons.

Minimal example snippet:

```xml
<scene>
  <camera eye="0 0 5" target="0 0 0" />
  <integrator type="path" samples="64" />
  <object file="sphere.obj" shader="neutral_shader" />
  <output name="beauty" file="out.exr" />
  <reference file="ref/out_ref.exr" />
</scene>
```

**How testrender is used**

- The runner parses the XML, sets up the scene, invokes the renderer/compiler (project test harness), and writes output images.
- Tests compare produced images against reference images; small pixel/variance differences indicate regressions.

**Rendering the shaderball scene (quick steps)**

- Build or install the tools so `oslc` and `testrender` are on your PATH (or use the build output, e.g. `build/bin/oslc`, `build/bin/testrender`).
- In the `testsuite/render-shaderball` directory, compile the shaders (creates `.oso` bytecode):

```bash
oslc *.osl
```

- Render the scene (choose meter or centimeter version):

```bash
testrender scene-meter.xml
# or
testrender scene-centimeter.xml
```

Note: `testrender` will load the compiled `.oso` shader blobs and the referenced geometry/textures. If your build places the binaries in `build/bin`, run `build/bin/testrender` instead.

**Shaderball scene components (what each file does)**

- `scene-meter.xml` / `scene-centimeter.xml`: Scene description (camera, shader groups, model filename). Choose meter vs centimeter to match the scene scale.
- `neutral.osl` / `neutral.oso`: The neutral shader applied to the shaderball swatch — samples a grey texture and uses a diffuse BRDF.
- `matte.osl` / `matte.oso`: Simple Lambertian material used for background and panels.
- `emitter.osl` / `emitter.oso`: Emissive shader used for the light panels in the scene.
- `envmap.osl` / `envmap.oso`: Environment mapping shader (if used by the scene).
- `shaderball-scene-light-geo-*.obj` + `.mtl`: Geometry for the shaderball and surrounding light/ground panels; the `.mtl` maps material names (e.g. `neutral`, `emitterLeft`) to the `ShaderGroup` names in the XML.
- `maps/neutral.ACEScg.exr` (referenced by `neutral.osl`): neutral texture used by the neutral shader (place under `maps/` or adjust `texture_path` parameter).

**Common issues & tips**

- Ensure compiled `.oso` files are next to the `.osl` files or on `OSL_SHADER_PATH` so `testrender` can find them.
- If textures are missing, set `texture_path` to an absolute or repo-relative path, or copy textures into `testsuite/render-shaderball/maps/`.
- For reproducible reference renders, use the same XML (meter/centimeter), integrator settings, and fixed random seeds where available.



