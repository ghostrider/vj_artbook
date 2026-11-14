# Point Cloud Scan Worlds — design

## Look

A real place made of glittering dust floats on a black LED wall. It might be the club itself, scanned that afternoon, or a forest or a CC0 statue. Millions of tiny round points sit densely where surfaces faced the scanner and thin out to nothing in the scan shadows behind objects. The camera drifts slowly, and parallax is what makes the dust read as a room. That is the "House of Cards" and "Clove Cigarette" look the research describes ([James Frost](https://www.jamesfrost.tv/case-studies/radiohead), [No Film School](https://nofilmschool.com/music-video-lidar-point-clouds-unreal-engine)). There are two colour modes. **Photo** uses the points' own RGB, a little desaturated. **Lidar** is a false-colour depth ramp from deep blue to orange, after the Ouster lead image ([Commons](https://commons.wikimedia.org/wiki/File:Ouster_OS1-64_lidar_point_cloud_of_intersection_of_Folsom_and_Dore_St,_San_Francisco.png)). Rhythm comes from three events. A bright slicing slab sweeps through the volume once per bar. Every kick sends a radial shockwave that pushes points outward before they spring back. In the break the whole scan dissolves into a drifting noise cloud, and it snaps back together on the drop. The mood is cool, forensic and dreamlike: a memory of a place, not a picture of it.

## Layer stack

The point cloud is rendered in a 3D host: Unreal Engine 5.7, TouchDesigner or Godot. It reaches Resolume over Spout (same machine) or NDI (second machine). Wire cannot draw it, because it has no vertex shaders ([Resolume ISF](https://resolume.com/support/en/isf)). Resolume handles compositing, colour and post only.

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Background | solid #000000 | Normal | 100 % | true black; the gaps in the scan are part of the image |
| 2 | Point data (not visible) | rest positions `p0`, colour `c0` and a per-point random seed, stored as float textures (UE/TD/Godot) or the native point asset (UE LiDAR plugin) | — | — | 1–8 M points depending on host (see Performance) |
| 3 | Scan render (3D host) | instanced soft-disc sprites, additive into an RGBA16F target. Includes dissolve, shockwave, bass warp and slice mask in the vertex stage | Normal into Resolume | 100 % | Spout/NDI source `ScanWorld`, 1920×1080 or wall native |
| 4 | Data pass (optional, 3D host) | second stream: R = normalised view depth, G = slice mask, B = shockwave mask | not visible | — | Spout source `ScanData`, feeds layer 5 in Wire |
| 5 | Colour stage | Wire/ISF effect on layer 3: Photo/Lidar ramp mix, slice glow tint from layer 4 | effect on layer 3 | — | see Color scheme |
| 6 | Feedback trail (optional) | Resolume layer 3 duplicated with a short feedback (Wire, Technique 6) | Add | 0–40 % | used only in breaks, never on the dense scan |
| 7 | Post | Bloom → light Chromatic Aberration on drops → Grain 1–2 % | effects on composition | — | order in Blending and compositing |

## Techniques

**1. Capture to stage (offline pipeline)**
1. **Source.** (a) *Own scan:* film or photograph the place and reconstruct it. The Clove Cigarette team used Leica lidar plus iPhone 4K video in Metashape ([No Film School](https://nofilmschool.com/music-video-lidar-point-clouds-unreal-engine)). Derivative's tutorial trains splats in PostShot from video ([Derivative tutorial 05](https://derivative.ca/community-post/tutorial/touchdesigner-tutorial-05-3d-gaussian-splatting-trainingrefiningmanipulating)). Niantic's capture tools (including Scaniverse data) export Gaussian splats as SPZ or PLY ([Niantic Spatial](https://www.nianticspatial.com/products/capture)). (b) *CC0:* Poly Haven scanned models are CC0 meshes ([Poly Haven licence](https://polyhaven.com/license)). Turn them into points with CloudCompare **Mesh > Sample points**, which takes a total count or a surface density and can carry colours and normals across ([CloudCompare wiki](https://www.cloudcompare.org/doc/wiki/index.php/Mesh%5CSample_points)). (c) *Radiohead data:* released for remixing ([James Frost](https://www.jamesfrost.tv/case-studies/radiohead)). Check its licence before paid use.
2. **Clean.** In CloudCompare: crop to the region you want with a box. Delete floaters (isolated specks in the sky), the operator, tripods and any faces of people who did not agree to be scanned. Level the floor plane to Z = 0 and move the scene's centre of interest to the origin. Every effect below assumes the origin is the "stage centre".
3. **Decimate to the stage budget.** CloudCompare **Edit > Subsample** offers Random (count or percentage), Spatial (minimum distance between points) and Octree (keeps the point nearest each cell centre) ([CloudCompare wiki](https://cloudcompare.org/doc/wiki/index.php/Edit%5CSubsample)). Use **Spatial** first, so that dense areas near the scanner do not eat the budget: 5–10 mm for a room, 2–3 cm for a street or forest. Then use **Random** to hit the exact count the host needs (e.g. 4,194,304 = 2048²). Keep a 25 % version as a fallback for the show machine.
4. **Format per host.** *UE LiDAR plugin:* LAS/LAZ (8/12/16-bit), E57 or ASCII XYZ/PTS/TXT. Meters are converted to UU at 1 UU = 1 cm ([Epic](https://dev.epicgames.com/documentation/unreal-engine/lidar-point-cloud-plugin-overview-in-unreal-engine)). PLY is not on that list, so export LAS from CloudCompare. *TouchDesigner:* Point File In TOP reads .ply, .obj, .xyz/.pts/.csv/.txt and .exr. Derivative recommends EXR, because it is binary ([Derivative](https://docs.derivative.ca/Point_File_In_TOP)). *Niagara and Godot:* bake two textures. `posTex` RGBA32F holds XYZ normalised to the scene bounds in 0–1, with the seed in A. `colTex` RGBA8 holds RGB, with lidar intensity in A if you have it (else 1). Point i sits at texel `(i mod W, floor(i / W))`. TouchDesigner is a convenient baker: load with Point File In TOP, then write the texture out as EXR.
5. **Shuffle the order** before baking (random permutation). A "first N points" cut, via `visible_instance_count` or a spawn count, is then a uniform thinning of the whole scene, not a missing corner. That gives a free density fader.
6. **Two-scan morph prep.** Bake both scans to the *same* texel count. Sort each by a 3D Morton (Z-order) code of the normalised position before the final shuffle, using the same permutation for both. Point i in scan A then lands near point i in scan B, so the morph flows instead of exploding. This is a design proposal and untested.

**2. Point sprite rendering**
1. One instance per point: a camera-facing quad (2 triangles), or a hardware point where the host supports `POINT_SIZE` (Godot, [docs](https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/spatial_shader.html)). Quads look the same on every GPU, while point-size limits vary by driver.
2. Screen size: `sizePx = pointSize · focalPx · worldRadius / viewDepth`, clamped to 1–8 px. Keep a world radius about equal to the decimation spacing (Technique 1.3), so surfaces close up without becoming blobs.
3. Sub-pixel fade: if the unclamped `sizePx < 1`, multiply brightness by `sizePx²`. Far points then dim instead of shimmering. This matters on LED walls, where aliasing crawls.
4. Fragment: `r = length(uv·2 − 1)`, `alpha = exp(−r² · 4)`, discard if `r > 1`. Output `colour · alpha · brightness`, unlit.
5. Additive blending, depth test on, depth write off. There is no sorting, and density shows up as brightness, the "glowing dust" look. Clamp the accumulation later in tonemapping (Shaders, pass 3).

**3. Dissolve / reassemble ("dust-to-solid")**
1. Per point: `n = curlNoise(p0 · 0.15 + t · 0.05)` (per metre; vary speed with `warpSpeed`). The scatter target is `p0 + n · scatterRadius`, with scatterRadius 0.5–6 m.
2. Stagger: `d_i = smoothstep(seed_i · 0.6, seed_i · 0.6 + 0.4, dissolve)`. As `dissolve` rises, points leave one by one. The room crumbles; it does not fade.
3. Directional option: replace `seed_i` with `0.7 · saturate(dot(p0, dir) / extent · 0.5 + 0.5) + 0.3 · seed_i`. The scan then erodes from one side, like wind.
4. Position: `p = mix(p0, scatter, easeInOutCubic(d_i))`. Brightness: `· mix(1, 0.6, d_i)`, so the dust is dimmer than the solid scan.
5. Reassemble on the drop: drive `dissolve` 1 → 0 over 1 beat with ease-out-back (overshoot 1.2). Points fly in and settle with a small bounce.
6. This mirrors the dissolve-to-particles-and-reassemble idea GSOPs shows in TouchDesigner ([Radiancefields.com](https://radiancefields.com/gsops-brings-native-gaussian-splatting-to-touchdesigner)). The version here is stateless, with no simulation, so it can be scrubbed backwards.

**4. Audio displacement**
1. **Kick shockwave:** each kick stores `tk` (time since the hit). Then `front = tk · 8 m/s`, `r = |p0 − c|` with `c` = stage centre, and `env = exp(−((r − front) / 0.6)²) · exp(−tk · 2.5)`. Displacement is `p += normalize(p0 − c) · pulse · env · 0.4 m`, and brightness gets `+ env · 1.5`. Keep the last 3 kicks so fast patterns overlap.
2. **Bass warp:** `p += curlNoise(p0 · 0.4 + t · 0.2) · warp · bassEnv · 0.15 m`. Low frequency makes the room breathe, not jitter.
3. **Hi-hat sparkle:** `if hash(seed_i, floor(t · 30)) < sparkle · hatEnv · 0.03` the point flashes at 3× brightness and 1.5× size for that frame.
4. All displacement is computed from `p0` each frame (stateless), so nothing drifts over a long set.

**5. Slicing planes and lidar rings**
1. Slab: `s = dot(p0, nrm) − slicePos`, `mask = 1 − smoothstep(w/2, w/2 + feather, |s|)` with w 2–20 cm and feather 2 cm.
2. Brightness: `ambient + mask · sliceGain`. `ambient` 0.05–1.0: at 1 the whole scan is lit and the slab is a highlight; at 0.05 only the slab shows (the medical-scan look).
3. `slicePos` sweeps from −extent to +extent once per bar (sawtooth), or ping-pongs every 2 bars (triangle). `nrm` switches between X, Y and Z every 4 bars.
4. Lidar rings (variation): `elev = atan(p0.z − sensorZ, length(p0.xy))`, `ringMask = smoothstep(0.85, 1, fract(elev · 64 / FOVrad))`, after the 64 lasers of the "House of Cards" Velodyne ([James Frost](https://www.jamesfrost.tv/case-studies/radiohead)). Rotate a bright azimuth wedge once per bar for the spinning-scanner sweep.
5. UE plugin shortcut: a **Lidar Clipping Volume** in "Clip Outside" mode, scaled to a thin box and animated, gives a hard-edged slab with no material work. A level supports up to 16 volumes ([Epic](https://dev.epicgames.com/documentation/unreal-engine/lidar-point-cloud-plugin-overview-in-unreal-engine)).

**6. Two-place morph**
1. `p = mix(p0A, p0B, m_i)` with `m_i = smoothstep(seed_i · 0.5, seed_i · 0.5 + 0.5, morph)`. Colour is mixed with the same `m_i`.
2. Add an arc so points don't pass through the middle in a straight line: `p += up · sin(π · m_i) · 0.5 m`.
3. In TouchDesigner with splats, GSOPs blends loaded splats globally or by texture ([Radiancefields.com](https://radiancefields.com/gsops-brings-native-gaussian-splatting-to-touchdesigner)), so a mask image can morph one region while another holds.

**7. Camera**
A Catmull-Rom spline through 4–8 hand-placed points, one segment per 4 or 8 bars. Speed eases in and out per segment, never constant. Add a 0.3–0.8° slow roll and a 2 % FOV breath on the bass envelope. Parallax is the effect, so keep a strong foreground element within 1–3 m of the camera.

## Shaders

Render order per frame: (1) point vertex+fragment → (2) optional data pass → (3) tonemap/output in the 3D host → (4) colour ISF in Wire/Resolume. There is no ping-pong in the core effect; everything is stateless from `p0`.

**Pass 1: Scan points (3D host, instanced)**
- Purpose: draw every point with all live deformations.
- Inputs: `posTex` (RGBA32F), `colTex` (RGBA8) or native attributes. Optional `posTexB`, `colTexB` (morph). Uniforms: `time`, `bpmPhase` (0–1 per bar), `dissolve`, `pointSize`, `pulse`, `kickT[3]`, `warp`, `bassEnv`, `hatEnv`, `sparkle`, `slicePos`, `sliceNrm`, `sliceWidth`, `ambient`, `morph`, `boundsMin`, `boundsMax`.
- Vertex stage: (1) `i` = instance ID, fetch texel `(i mod W, i / W)`. (2) `p0 = mix(boundsMin, boundsMax, pos.rgb)`, `seed = pos.a`. (3) Apply morph (T6), then dissolve (T3), then shockwave and bass warp (T4). (4) Compute the slice mask and brightness (T5) and pass them to the fragment stage. (5) Expand the quad corner in view space by `sizePx` (T2) and pass `uv`.
- Fragment stage: soft disc (T2.4) × colour × brightness; additive output.
- Output: RGBA16F colour target at output resolution.

**Pass 2: Data (optional)**
- The same geometry with a cheap fragment, rendered into a second RGBA8 target. R = view depth normalised by the near/far of the scene bounds, G = slice mask, B = shockwave envelope. Use max blending (or additive, clamped) so the strongest value wins.
- Output resolution: half of pass 1 is enough (960×540). Resolume scales it up.

**Pass 3: Tonemap and send (3D host)**
- Input: pass 1. Per pixel: `c = c / (1 + c · 0.6)` (soft Reinhard). This keeps dense overlaps from clipping to flat white blobs. Gamma to sRGB, then out over Spout/NDI as 8-bit.

**Pass 4: Colour stage (ISF in Wire, single pass)**
- Inputs: `inputImage` (ScanWorld), `dataImage` (ScanData), uniforms `colorMode` (0 = Photo, 1 = Lidar), `saturation`, `sliceTint`, `rampShift`.
- Per pixel: (1) `lum = dot(c, (0.2126, 0.7152, 0.0722))`. (2) Photo = desaturate `c` to `saturation`. (3) Lidar = `ramp(fract(depth + rampShift)) · lum · 1.4`, with the ramp from Color scheme. (4) `mix(Photo, Lidar, colorMode)`. (5) `+ sliceTint · G · lum`. (6) Clamp so the max channel is ≤ 0.95.
- Without the data stream, Lidar mode falls back to a ramp on `lum`. That is less accurate, but it works with any Spout source.

## Color scheme

| role | hex | use |
|------|-----|-----|
| base / black | #000000 | background, scan gaps |
| ramp 0 (far / low) | #0B1A4A | deep blue |
| ramp 1 | #1E5AA8 | blue |
| ramp 2 | #3FC1C9 | cyan |
| ramp 3 | #F2A541 | amber |
| ramp 4 (near / high) | #FF6A1A | orange |
| highlight (slab, shockwave) | #FFF4E0 | warm white |
| shadow (dust in dissolve) | #5A6478 | grey-blue tint, mixed in at `d_i` × 50 % |

- **Mapping:** Lidar mode uses a 5-stop linear gradient ramp on normalised depth (or on lidar intensity × range, if `colTex.a` holds intensity, as in the Ouster image). Photo mode uses source RGB at 70–85 % saturation, with contrast +10 %, because phone scans are often flat.
- **Over time:** `rampShift` drifts +0.02 per bar. Switch `colorMode` on the phrase (16 bars), with a 1-beat crossfade. Hi-hat rolls can flip colorMode for 1/16 notes in a build.
- **LED wall:** black is true 0. Clamp highlights to about 95 %, and keep average picture level under 25 % in the dense solid state; lower `pointSize` or `ambient` if needed. Large white areas on a big wall are blinding and draw a lot of power. Gamma: check the ramp on the actual wall, because LED processors often crush the deep blue #0B1A4A into black. Lift it to #12245E if it disappears.

## Blending and compositing

- Points to target: **additive** inside the 3D host (Technique 2.5). Overlap becomes glow, with no sort cost.
- Scan layer in Resolume: **Normal** over black. When combining with other content, use **Add**, so the black gaps stay transparent-looking without a key.
- Masks: the data stream's G channel (slice) can be used as a luma mask for another Resolume layer. Video then only shows inside the sweeping slab.
- Feedback (layer 6, break only): Wire feedback of the scan layer with 0.92 decay and a 1.003 zoom, mixed at 0–40 % in **Add**. Use it only when `dissolve` > 0.5. On the solid scan it smears the parallax, which is the whole point of the look.
- Post chain, in order: (1) **Bloom**: threshold 0.7, intensity 0.8–1.2, radius medium (Resolume's built-in Bloom, or bloom in the 3D host's post-process). (2) **Chromatic Aberration**: 0 normally, pulse to a small offset (≈ 2 px at 1080p) for 1 beat on the drop. (3) **Grain**: 1–2 %, which hides 8-bit banding in the dark ramp. Do not sharpen; it turns soft discs into hard squares.

## Motion and timing

- Camera: one spline segment per 4 bars (calm) or 8 bars (breakdown). Translation speed 0.2–1.5 m/s. Every segment eases in and out.
- Slice: 1 sweep per bar at 120–130 BPM. Halve it to 1 per 2 bars above 140 BPM, so it stays readable.
- Shockwave: every kick. Front speed 8 m/s (room scale), or 20 m/s for street scale.
- Sparkle: hats, at a 30 Hz random reseed.
- **Build:** `ambient` falls from 1 to 0.1 over the last 8 bars, so only the slab is visible. Slice width narrows from 20 cm to 3 cm.
- **Drop:** `dissolve` goes 1 → 0 over 1 beat with ease-out-back, `ambient` snaps to 1, chromatic aberration pulses, and the camera cuts to the next spline segment.
- **Break:** `dissolve` goes 0 → 0.8 over 4 bars (ease-in), the feedback layer comes in, the camera slows to 0.2 m/s and bass warp rises.
- Phrase (16/32 bars): switch to the second scan with `morph` over 2 bars, or switch colorMode.

## Parameters

| name | range | default | what it does on stage | suggested MIDI/BPM mapping |
|------|-------|---------|-----------------------|----------------------------|
| dissolve | 0–1 | 0 | solid scan ↔ drifting dust cloud | BeatStep Pro fader / KeyStep mod strip; drop macro 1 → 0 |
| pointSize | 0.3–3 | 1 | grain of the dust: fine sparkle ↔ thick, almost solid surfaces | knob |
| slice | 0–1 (ambient), + width 0.02–0.2 m | ambient 1, width 0.08 | how much of the scan is lit outside the sweeping slab | knob for ambient; slab position auto per bar |
| pulse | 0–2 | 0.8 | strength of the kick shockwave push and flash | kick envelope × knob |
| warp | 0–2 | 0.3 | bass breathing of the whole room | bass envelope × knob |
| morph | 0–1 | 0 | scene A ↔ scene B point by point | fader, or a 2-bar BPM ramp on a pad |

Also, as toggles rather than stage parameters: `colorMode` (pad), slice axis (pad, cycles X/Y/Z), density (the first-N cut from Technique 1.5).

## Build recipes

### Resolume Arena
- Layer 1: solid black (Solid Color source) or empty.
- Layer 2: Spout/NDI source `ScanWorld`. Effects in order: the Wire colour effect (pass 4), **Bloom** (threshold 0.7), **Chromatic Aberration** (0, mapped to a drop button), and grain.
- Layer 3 (optional): Spout source `ScanData`, opacity 0, as an input for the Wire effect. Alternatively, use its G channel as a mask on a video layer above.
- Layer 4 (optional): duplicated scan with the Wire feedback effect, blend **Add**, opacity mapped to a fader.
- Dashboard: map `dissolve`, `pointSize`, `slice`, `pulse`, `warp` and `morph` as Resolume dashboard controls. Send them to the 3D host as OSC (Resolume OSC output), so one fader controls both machines.
- Custom ISF fits only in the colour stage and feedback. Resolume's support pages describe ISF through Wire ([Resolume ISF](https://resolume.com/support/en/isf)).

### Resolume Wire
- Patch `ScanColor` (effect): inputs are the effect input (ScanWorld) and a second texture input (ScanData via a Spout/NDI source node). An ISF node runs the single-pass colour shader (pass 4). Expose `colorMode`, `saturation`, `rampShift` and `sliceTint` (colour picker, default #FFF4E0).
- Patch `ScanTrail` (effect): input → mix with the previous frame (a feedback node, decayed by 0.92, zoomed by 1.003) → output. Expose `decay` and `zoom`. If your Wire version does not support multipass or persistent ISF buffers, build the loop with Wire's own feedback node, not inside ISF. Version support varies, so test it.
- No point rendering in Wire: it has no vertex shaders, as of the 7.22 note ([Resolume ISF](https://resolume.com/support/en/isf)).

### Unreal Engine 5.7
Two routes. Pick one per scene.

**Route A: LiDAR Point Cloud plugin (fastest, best for large static scans)**
1. Enable the LiDAR Point Cloud plugin. Import LAS/LAZ/E57. Place the asset in the level.
2. Performance is governed by the plugin's global point budget, a cap on how many points are drawn, which the system shares across all visible clouds. Set it with the console variables listed in the plugin reference ([Epic](https://dev.epicgames.com/documentation/unreal-engine/lidar-point-cloud-plugin-overview-in-unreal-engine)). Assets stream on demand once saved.
3. Slicing: a Lidar Clipping Volume ("Clip Outside"), scaled to 5 cm thick and moved by a Blueprint from `bpmPhase` (Technique 5.5).
4. Limit: Epic warns that moving points at runtime does not reassign them to LOD nodes ([Epic](https://dev.epicgames.com/documentation/unreal-engine/lidar-point-cloud-plugin-overview-in-unreal-engine)). Do not use this route for dissolve or shockwave. Use it for fly-throughs and slicing.

**Route B: Niagara GPU (all live effects)**
1. Bake `posTex` (EXR, RGBA32F; import with compression set to HDR, uncompressed) and `colTex` (Technique 1.4). A 2048² texture holds 4.19 M points.
2. Niagara System, GPU Compute Sim, fixed bounds set to the scan bounds plus `scatterRadius`. Emitter Spawn: Spawn Burst Instantaneous, count = W×H. Lifetime infinite (no Kill), so the particles persist.
3. Particle Spawn: store `ExecutionIndex` as an int attribute `PointId`. Particle Update (every frame, stateless): compute the texel UV from `PointId`, sample `posTex` and `colTex` with Texture Sample data interfaces, then apply morph → dissolve → shockwave → warp → slice in one Scratch Pad module (or a Custom HLSL node), all from User parameters. Write `Position`, `Color` and `SpriteSize`.
4. Renderer: Sprite Renderer, facing mode Face Camera. The material is Unlit, Additive, a radial-gradient disc (Technique 2.4), with Particle Color × emissive gain. Size comes from `SpriteSize`, with a min screen-size clamp in the material if needed.
5. Post-process volume: bloom off or low in UE if Resolume does bloom; exposure fixed (manual); tonemapper on. Render at the output resolution.
6. Remote Control: expose the Niagara User parameters (`Dissolve`, `PointSize`, `Pulse`, `Warp`, `Morph`, `SliceAmbient`, `SliceWidth`, `ColorMode`, `KickTrigger`) and the camera spline time in a Remote Control Preset. Bind them to OSC from Resolume and MIDI from the BeatStep Pro.
7. Output: send the final frame over Spout (same PC) or NDI (second PC) with your existing sender plugin. Send the data pass (Route B only) as a second Spout stream from a Scene Capture 2D rendering to a render target, at half resolution.

### Godot
1. Node setup: a `MultiMeshInstance3D` with a MultiMesh of `QuadMesh` (0.01 × 0.01). Set `use_colors` and `use_custom_data` *before* `instance_count`, because they can only be changed while the count is 0. Use `visible_instance_count` as the live density fader; it does not resize the buffers ([Godot MultiMesh](https://docs.godotengine.org/en/stable/classes/class_multimesh.html)). Leave per-instance transforms at identity and fetch positions from `posTex` by `INSTANCE_ID` in the vertex shader, so the CPU never uploads per-point data.
2. Spatial shader, `render_mode unshaded, blend_add, depth_draw_never, cull_disabled`. In the vertex stage: texel fetch by `INSTANCE_ID`, effects (Techniques 3–5), then billboard the quad in view space and size it per Technique 2. The fragment stage draws the soft disc. `unshaded` skips lighting and `blend_add` is additive ([Godot spatial shader](https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/spatial_shader.html)).
3. Alternative for very high counts: an `ArrayMesh` with `PRIMITIVE_POINTS`, writing `POINT_SIZE` and shaping the disc with `POINT_COORD` (same docs). Test it on the show GPU first.
4. Uniforms to expose: `dissolve`, `point_size`, `pulse`, `kick_t` (vec3), `warp`, `bass_env`, `hat_env`, `slice_pos`, `slice_width`, `slice_ambient`, `morph`, `color_mode`. Drive them from an OSC receiver script, or from Godot's MIDI input events.
5. Output: render a `SubViewport` at the output resolution with an `Environment` (glow off, or bloom low), and send it to Resolume via NDI or a Spout extension. Godot has no built-in Spout sender.

### TouchDesigner (third host, and the only verified splat path)
- Points: Point File In TOP loads PLY or EXR into a texture ([Derivative](https://docs.derivative.ca/Point_File_In_TOP)). Effects go in a GLSL material on instanced geometry, or in POPs. Output via Spout Out TOP / NDI Out TOP.
- Gaussian splats: the Derivative palette component **gsLoader** loads .spz and .ply splats into a POP as points with splat attributes ([Derivative](https://derivative.ca/UserGuide/Palette:gsLoader)). **GSOPs** (beta, TouchDesigner 2025.32820 or later, free non-commercial Lite tier) adds Convert to ParticleSystem (dissolve), Thin Splat (budget), Frustum Delete and splat blending ([Radiancefields.com](https://radiancefields.com/gsops-brings-native-gaussian-splatting-to-touchdesigner)). For splats, use TouchDesigner. The research did not verify splat rendering in UE, Godot or Resolume, so in those hosts treat splat PLY files as plain coloured point clouds.

## Audio reactivity

| band | drives | attack / release | threshold |
|------|--------|------------------|-----------|
| kick (40–120 Hz, onset) | shockwave trigger (`kickT` reset), +20 % pointSize for 80 ms | trigger; envelope decay 400 ms | onset > 1.4 × 1 s running mean, 120 ms retrigger lockout |
| bass (40–200 Hz, RMS) | `bassEnv` × warp; camera FOV breath | 60 ms / 600 ms | gate below 15 % to keep quiet passages still |
| low-mid (200–800 Hz) | `ambient` lift (+0.1) in the build | 200 ms / 1 s | none |
| hats (6–12 kHz, onset) | `hatEnv` → sparkle; optional colorMode flicker | 5 ms / 120 ms | onset > 1.6 × mean |
| full-band RMS | bloom intensity 0.8 → 1.2 | 100 ms / 800 ms | — |

Do the analysis in the 3D host, or send envelopes from Resolume's audio analysis as OSC. Prefer BPM clock over audio for the slice sweep and camera, because audio-timed sweeps drift. Use audio only for amplitudes and triggers.

## Performance tips

- **Point budgets at 60 fps** (my estimates for an RTX 3070–4080 class GPU at 1080p, not measured; verify on the show machine). UE LiDAR plugin: point budget 5–15 M; the Clove Cigarette team handled about 20 M on screen in Unreal for an offline render ([No Film School](https://nofilmschool.com/music-video-lidar-point-clouds-unreal-engine)). Niagara GPU sprites with the full effect stack: 2–4 M. TouchDesigner instancing: 1–4 M. Godot MultiMesh quads: 1–2 M (points primitive higher). TD splats via GSOPs: 0.5–2 M, using Thin Splat and Frustum Delete.
- Cost scales with **points × covered pixels**. Large `pointSize` close to the camera is the real killer (overdraw), not the point count. Clamp sizePx at 8 px, and lower the clamp at 4K.
- The shuffled order (Technique 1.5) lets you drop to 50 % density mid-show with no visible hole. Map density to a safety fader.
- Tall, wide and odd-aspect LED walls: render at the wall's pixel map (e.g. 3840×640), not 16:9 scaled. Set the camera FOV on the *short* axis, and use a wider spline so parallax reads across the width. Sub-pixel fade (Technique 2.3) matters most on low-resolution, large-pitch walls.
- Use 16-bit half positions only for normalised textures and scenes under about 20 m; beyond that, use 32-bit float, or points jitter in a grid.
- NDI adds about one frame of latency. Same-machine Spout is preferred for kick-synced shockwaves.
- UE LiDAR plugin: save imported assets before the show. Unsaved assets stay in RAM and cannot stream ([Epic](https://dev.epicgames.com/documentation/unreal-engine/lidar-point-cloud-plugin-overview-in-unreal-engine)).
- For very large raw scans (tens of millions of points), crop and tile them before subsampling, and keep the stage copy small. Faster iteration, and it is safer on a laptop.

## Further references

- [Epic: LiDAR Point Cloud Plugin overview](https://dev.epicgames.com/documentation/unreal-engine/lidar-point-cloud-plugin-overview-in-unreal-engine): formats, point budget, clipping volumes, runtime edits.
- [Derivative: Point File In TOP](https://docs.derivative.ca/Point_File_In_TOP) and [gsLoader](https://derivative.ca/UserGuide/Palette:gsLoader): loading point and splat files into TouchDesigner.
- [CloudCompare wiki: Edit\Subsample](https://cloudcompare.org/doc/wiki/index.php/Edit%5CSubsample) and [Mesh\Sample points](https://www.cloudcompare.org/doc/wiki/index.php/Mesh%5CSample_points): decimation and mesh-to-points.
- [No Film School: Clove Cigarette](https://nofilmschool.com/music-video-lidar-point-clouds-unreal-engine): a full lidar and photogrammetry to Unreal production breakdown.
