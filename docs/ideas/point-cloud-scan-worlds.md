---
tags:
  - photogrammetry
  - point-cloud
  - 3d
  - particles
---

# Point Cloud Scan Worlds

> A real place, rebuilt from millions of glowing points, that crumbles to dust on the break and snaps back on the drop.

![Orthographic view of a registered lidar point cloud, about 23 million points over 18 seconds, of a San Francisco intersection, coloured by intensity times range from dark blue to orange](https://upload.wikimedia.org/wikipedia/commons/thumb/b/bd/Ouster_OS1-64_lidar_point_cloud_of_intersection_of_Folsom_and_Dore_St%2C_San_Francisco.png/1280px-Ouster_OS1-64_lidar_point_cloud_of_intersection_of_Folsom_and_Dore_St%2C_San_Francisco.png)
*Registered lidar point cloud of a San Francisco intersection, coloured from dark blue to orange. Image: [Daniel L. Lu (Dllu)](https://commons.wikimedia.org/wiki/File:Ouster_OS1-64_lidar_point_cloud_of_intersection_of_Folsom_and_Dore_St,_San_Francisco.png), [CC BY 4.0](https://creativecommons.org/licenses/by/4.0).*

## The idea

A point cloud is a set of points in 3D space. Each has X, Y, Z and often a colour. Lidar and structured-light scanners produce them, and so does photogrammetry software that reconstructs points from overlapping photos ([Wikipedia](https://en.wikipedia.org/wiki/Point_cloud)). Normally the cloud is a stepping stone towards a mesh. Here the raw points are the picture.

The look is dust in the shape of a place. Points are dense where surfaces faced the scanner and empty where it could not see. Those gaps are part of the image. The camera drifts slowly, and parallax is what turns the dust into a room.

On a club screen this works because the content is a real space that can fall apart. A dissolve fader moves every point from its scan position to a noise cloud and back. Kicks push a shockwave through the volume. A thin slab sweeps through the scan once per bar. The scene could be a forest, a statue, or the venue itself, scanned that afternoon.

## Origins and references

- **Radiohead, "House of Cards" (2008).** Directed by James Frost, with no conventional cameras. Exteriors and the party scene came from a Velodyne scanner with 64 rotating lasers. Thom Yorke's close-ups used Geometric Informatics' structured-light "GeoVideo" system. Part of the data was released as open source. [James Frost case study](https://www.jamesfrost.tv/case-studies/radiohead).
- **Andy Shauf, "Clove Cigarette" (2020).** Leica lidar and iPhone 4K video became photogrammetry point clouds in Metashape, assembled in Unreal Engine into one continuous camera move through about half a billion points. Unreal handled about 20 million on screen. [No Film School](https://nofilmschool.com/music-video-lidar-point-clouds-unreal-engine).
- **A$AP Rocky, "Helicopter" (2026).** Performances captured with a 56-camera RGB-D array and rendered as dynamic Gaussian splats. About 30 minutes of footage came to roughly 1 TB of PLY sequences. [Radiancefields.com](https://radiancefields.com/a-ap-rocky-releases-helicopter-music-video-featuring-gaussian-splatting).
- **Synth Splat** by Lorie Chen, Steph Ng and Nicholas Pudjarminta. A browser visualiser where Gaussian splat worlds crossfade between tracks and audio analysis drives shaders and lighting. [World Labs showcase](https://www.worldlabs.ai/labs/showcase/synth-splat).
- **3D Gaussian Splatting** (Kerbl, Kopanas, Leimkühler, Drettakis; SIGGRAPH 2023). Soft, oriented Gaussians optimised from photos, rendered at 30 fps or more at 1080p. [arXiv 2308.04079](https://arxiv.org/abs/2308.04079).

## Visual language

**Shape.** Glittering dust, soft round points, true black between them. Surfaces read as density, not as solid faces. Holes and scan shadows stay in.

**Colour.** Two modes. *Photo* uses the points' own RGB, a little desaturated. *Lidar* is a false-colour ramp from deep blue through cyan and amber to orange, after the Ouster lead image. Highlights are warm white. On a dark wall the mood is cool, forensic and dreamlike: a memory of a place, not a picture of it.

**Motion.** Slow fly-through along a spline that eases in and out. Orbits around a scanned person or object. A slicing plane sweeping through the volume. Lidar scan-line rings.

**Rhythm.** Dissolve and reassemble. In the break the scan drifts apart into a noise cloud. On the drop it flies back in with a small overshoot. GSOPs shows the same idea in TouchDesigner, with scenes dissolving into particles and reassembling ([Radiancefields.com](https://radiancefields.com/gsops-brings-native-gaussian-splatting-to-touchdesigner)).

## See it

- [Radiohead, "House of Cards" (official video)](https://www.youtube.com/watch?v=8nTFjVm9sTQ) — the original lidar and structured-light look, official Radiohead channel.
- [Synth Splat, World Labs showcase](https://www.worldlabs.ai/labs/showcase/synth-splat) — audio-reactive Gaussian splat worlds crossfading with a DJ set.
- [GSOPs article](https://radiancefields.com/gsops-brings-native-gaussian-splatting-to-touchdesigner) — splats dissolving into physics particles and back in TouchDesigner.
- [No Film School: Clove Cigarette breakdown](https://nofilmschool.com/music-video-lidar-point-clouds-unreal-engine) — behind the scenes of a lidar and photogrammetry point-cloud world in Unreal.
- [A$AP Rocky "Helicopter" write-up](https://radiancefields.com/a-ap-rocky-releases-helicopter-music-video-featuring-gaussian-splatting) — volumetric performance rendered as Gaussian splats.

## Code design

The point cloud is drawn in a 3D host: Unreal Engine 5.7, TouchDesigner or Godot. It reaches Resolume over Spout (same machine) or NDI (second machine). Wire cannot draw it, because it has no vertex shaders ([Resolume ISF](https://resolume.com/support/en/isf)). Resolume does compositing, colour and post.

Every point is stateless: its position each frame is computed from its rest position `p0` and a per-point random seed. Nothing drifts over a long set, and effects can be scrubbed backwards. Points are drawn as instanced soft-disc sprites, additive, depth test on, depth write off. Size on screen is `pointSize · focalPx · worldRadius / viewDepth`, clamped to 1 to 8 px. If the unclamped size drops below 1 px, brightness is multiplied by size squared, so far points dim instead of shimmering on an LED wall.

**Capture to stage**

1. **Source.** Own scan: film or photograph the place and reconstruct it (Metashape as in Clove Cigarette; PostShot from video as in [Derivative tutorial 05](https://derivative.ca/community-post/tutorial/touchdesigner-tutorial-05-3d-gaussian-splatting-trainingrefiningmanipulating); Niantic's capture tools export splats as SPZ or PLY, [Niantic Spatial](https://www.nianticspatial.com/products/capture)). Or a CC0 mesh from [Poly Haven](https://polyhaven.com/license), turned into points with CloudCompare [Mesh > Sample points](https://www.cloudcompare.org/doc/wiki/index.php/Mesh%5CSample_points). Or the Radiohead data, which was released for remixing; check its current licence before any commercial use.
2. **Clean.** In CloudCompare, crop to a box. Delete floaters, the operator, tripods and any faces of people who did not agree to be scanned. Level the floor to Z = 0 and move the centre of interest to the origin.
3. **Decimate.** Use [Edit > Subsample](https://cloudcompare.org/doc/wiki/index.php/Edit%5CSubsample). Spatial first (5 to 10 mm for a room, 2 to 3 cm for a street or forest), then Random to hit the exact count, for example 4,194,304 (2048 squared). Keep a 25 % copy as a fallback for the show machine.
4. **Format per host.** UE LiDAR plugin: LAS/LAZ or E57 (PLY is not on its list). TouchDesigner: Point File In TOP, EXR recommended. Niagara and Godot: two baked textures. `posTex` (RGBA32F) holds XYZ normalised to the scene bounds with the seed in A. `colTex` (RGBA8) holds RGB. Point i sits at texel `(i mod W, floor(i / W))`.
5. **Shuffle** the order before baking. A "first N points" cut is then a uniform thinning of the whole scene, which gives a free density fader.
6. **Two-scan morph prep.** Bake both scans to the same texel count. Sort each by a 3D Morton (Z-order) code before the final shuffle, with the same permutation for both, so point i in scan A lands near point i in scan B. This pairing is a design proposal and untested.

**Effects**

- **Dissolve.** Per point, the scatter target is `p0` plus curl noise times 0.5 to 6 m. A smoothstep staggered by the seed makes points leave one by one, so the room crumbles instead of fading. Reassemble over 1 beat with ease-out-back.
- **Kick shockwave.** Each kick stores the time since the hit. A ring front travels at 8 m/s, pushes points outward by up to 0.4 m and adds brightness. Keep the last three kicks so fast patterns overlap.
- **Bass warp.** Low-frequency curl noise displaces the whole room, so it breathes instead of jittering.
- **Hi-hat sparkle.** A small random fraction of points flashes at triple brightness for one frame, reseeded at 30 Hz.
- **Slicing slab.** Distance to a plane, thresholded with a soft edge, 2 to 20 cm wide. Ambient light level 1 means the slab is a highlight on a lit scan. Ambient 0.05 shows only the slab, the medical-scan look.
- **Lidar rings.** Mask by elevation angle in 64 bands, after the 64 lasers of the "House of Cards" Velodyne.
- **Two-place morph.** `p = mix(p0A, p0B, m_i)` with a staggered `m_i` and a small upward arc, so points do not cross the middle in a straight line. Untested.
- **Camera.** A Catmull-Rom spline through 4 to 8 hand-placed points, one segment per 4 or 8 bars, always eased. Keep a strong foreground element 1 to 3 m from the camera, because parallax is the effect.

Passes per frame: scan points (instanced), optional data pass (R = depth, G = slice mask, B = shockwave), soft Reinhard tonemap `c / (1 + 0.6c)` and Spout/NDI send, then a single-pass colour ISF in Wire.

=== "Resolume Wire"
    Wire does the colour stage only. Patch `ScanColor` as an effect with the scan as input and the data pass (`ScanData`, via a Spout or NDI source node) as a second texture. One ISF node mixes Photo (desaturated source) and Lidar (five-stop ramp on depth, times luminance) with `colorMode`, `saturation`, `rampShift` and `sliceTint`. Without the data stream, Lidar mode falls back to a ramp on luminance. It is less accurate but works with any Spout source.

    Patch `ScanTrail` for break sections: input mixed with the previous frame, decay 0.92, zoom 1.003. If your Wire version does not support persistent ISF buffers, build the loop with Wire's own feedback node. Version support varies, so test it.

    Wire has no vertex shaders, as of the 7.22 note ([Resolume ISF](https://resolume.com/support/en/isf)). It cannot render the points.

=== "Unreal Engine 5.7"
    Two routes. Pick one per scene.

    **Route A, LiDAR Point Cloud plugin.** Fastest for big static scans. Import LAS/LAZ/E57 and save the asset so it can stream. Performance is governed by the plugin's global point budget. Slice with a Lidar Clipping Volume in "Clip Outside" mode, scaled to about 5 cm thick and moved by a Blueprint from the bar phase. Epic warns that moving points at runtime does not reassign them to LOD nodes, so use this route for fly-throughs and slicing, not for dissolve or shockwave ([Epic](https://dev.epicgames.com/documentation/unreal-engine/lidar-point-cloud-plugin-overview-in-unreal-engine)).

    **Route B, Niagara GPU.** For all live effects. Import `posTex` as an uncompressed HDR EXR and `colTex`. A 2048 squared texture holds 4.19 million points. Use a GPU Compute emitter with fixed bounds, Spawn Burst Instantaneous at W times H, infinite lifetime. Store `ExecutionIndex` as `PointId`. In Particle Update, sample both textures and apply morph, dissolve, shockwave, warp and slice in one Scratch Pad module, all from User parameters. Sprite Renderer, Face Camera, with an unlit additive radial-disc material. Expose the User parameters and spline time in a Remote Control Preset and bind them to OSC from Resolume and MIDI from the BeatStep Pro. Send the frame over Spout or NDI. An optional data pass can leave as a second stream from a Scene Capture 2D at half resolution.

=== "GODOT"
    Use a `MultiMeshInstance3D` with a MultiMesh of `QuadMesh` (0.01 by 0.01). Set `use_colors` and `use_custom_data` before `instance_count`, because they can only change while the count is 0. `visible_instance_count` is the live density fader and does not resize buffers ([Godot MultiMesh](https://docs.godotengine.org/en/stable/classes/class_multimesh.html)). Leave per-instance transforms at identity and fetch positions from `posTex` by `INSTANCE_ID` in the vertex shader, so the CPU uploads nothing per point.

    Spatial shader with `render_mode unshaded, blend_add, depth_draw_never, cull_disabled`. The vertex stage fetches the texel, applies the effects, then billboards the quad. The fragment stage draws the soft disc ([spatial shader reference](https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/spatial_shader.html)). For very high counts, an `ArrayMesh` with `PRIMITIVE_POINTS` and `POINT_SIZE` is an alternative. Test it on the show GPU first. Drive the uniforms from an OSC receiver script or Godot's MIDI input. Render a `SubViewport` at output resolution and send it to Resolume via NDI or a Spout extension. Godot has no built-in Spout sender.

=== "TouchDesigner"
    The only host with verified splat support. Point File In TOP loads PLY or EXR into a texture ([Derivative](https://docs.derivative.ca/Point_File_In_TOP)). Effects go in a GLSL material on instanced geometry, or in POPs. Output with Spout Out TOP or NDI Out TOP. TouchDesigner is also a convenient baker for the Unreal and Godot textures: load the cloud, then write the texture out as EXR.

    For Gaussian splats, the Derivative palette component [gsLoader](https://derivative.ca/UserGuide/Palette:gsLoader) loads .spz and .ply into a POP. GSOPs (beta, needs TouchDesigner 2025.32820 or later, free non-commercial Lite tier) adds Convert to ParticleSystem for the dissolve, Thin Splat for the budget, Frustum Delete, and splat blending, globally or by texture ([Radiancefields.com](https://radiancefields.com/gsops-brings-native-gaussian-splatting-to-touchdesigner)). Splat rendering was not verified in Unreal, Godot or Resolume. In those hosts, treat splat PLY files as plain coloured point clouds.

## On stage

**Audio reactivity.**

| band | drives | attack / release |
|------|--------|------------------|
| kick (40 to 120 Hz, onset) | shockwave trigger, +20 % point size for 80 ms | trigger; decay 400 ms; 120 ms retrigger lockout |
| bass (40 to 200 Hz, RMS) | warp amount, camera FOV breath | 60 ms / 600 ms; gate below 15 % |
| low-mid (200 to 800 Hz) | ambient lift of +0.1 in the build | 200 ms / 1 s |
| hats (6 to 12 kHz, onset) | sparkle, optional colour-mode flicker | 5 ms / 120 ms |
| full-band RMS | bloom intensity 0.8 to 1.2 | 100 ms / 800 ms |

Analyse audio in the 3D host, or send envelopes from Resolume as OSC. Use the BPM clock, not audio, for the slice sweep and the camera, because audio-timed sweeps drift.

**MIDI (KeyStep 37 / BeatStep Pro).**

| parameter | range | what it does | mapping |
|-----------|-------|--------------|---------|
| dissolve | 0 to 1 | solid scan to dust cloud | BeatStep Pro fader or KeyStep mod strip; drop macro 1 to 0 |
| pointSize | 0.3 to 3 | fine sparkle to almost solid surfaces | knob |
| slice ambient | 0 to 1 | how much is lit outside the slab | knob; slab position runs per bar |
| pulse | 0 to 2 | kick shockwave strength | kick envelope times knob |
| warp | 0 to 2 | bass breathing | bass envelope times knob |
| morph | 0 to 1 | scene A to scene B | fader, or a 2-bar ramp on a pad |

Pads: colour mode, slice axis (cycles X, Y, Z every press). Also keep a density fader (the first-N cut) as a safety valve.

**Structure.**

- **Build:** ambient falls from 1 to 0.1 over the last 8 bars, so only the slab shows. Slab width narrows from 20 cm to 3 cm.
- **Drop:** dissolve 1 to 0 over one beat, ambient snaps to 1, a short chromatic aberration pulse, and the camera cuts to the next spline segment.
- **Break:** dissolve 0 to 0.8 over 4 bars, the feedback layer comes in, the camera slows to 0.2 m/s, bass warp rises.
- **Phrase (16 or 32 bars):** morph to the second scan over 2 bars, or switch colour mode with a one-beat crossfade.

**Layering.** Layer 1 is solid black. The scan layer sits over it as a Spout/NDI source, with the Wire colour effect, Bloom (threshold 0.7), Chromatic Aberration (0 by default, mapped to a drop button) and 1 to 2 % grain. Add a duplicate of the scan with the Wire feedback effect in Add mode at 0 to 40 %, only when dissolve is above 0.5. On the solid scan it smears the parallax. When combining with other content, use Add so the black gaps stay open. The data stream's slice channel can serve as a luma mask, so a video layer only shows inside the sweeping slab. Do not sharpen: it turns soft discs into hard squares.

**LED wall.** Keep true black, clamp highlights to about 95 %, and keep the average picture level under 25 % in the dense solid state. Large white areas on a big wall are blinding and draw a lot of power. LED processors often crush the deep blue `#0B1A4A` into black. If it disappears, lift it to `#12245E`. Render at the wall's pixel map, not scaled 16:9, and set the FOV on the short axis.

**Performance.**

- Point budgets at 60 fps are my estimates for an RTX 3070 to 4080 class GPU at 1080p, not measured. Verify on the show machine. UE LiDAR plugin 5 to 15 M; Niagara GPU sprites with the full stack 2 to 4 M; TouchDesigner instancing 1 to 4 M; Godot MultiMesh quads 1 to 2 M; TouchDesigner splats via GSOPs 0.5 to 2 M.
- For reference, the Clove Cigarette team handled about 20 M points on screen in Unreal for an offline render ([No Film School](https://nofilmschool.com/music-video-lidar-point-clouds-unreal-engine)).
- Cost scales with points times covered pixels. Large point size close to the camera is the real killer. Clamp at 8 px, lower at 4K.
- The shuffled order lets you drop to 50 % density mid-show without a visible hole.
- NDI adds about one frame of latency. Use same-machine Spout for kick-synced shockwaves.
- Save imported assets in the UE LiDAR plugin before the show. Unsaved assets stay in RAM and cannot stream.
- Crop and tile very large raw scans before subsampling, and keep the stage copy small.

## Variations

1. **Dust-to-solid.** The room appears only on the drop.
2. **Slicing planes.** A thin plane sweeps once per bar and only the slab is lit: a medical-scan or lidar-sweep look.
3. **Two-place morph.** Blend a club and a forest point by point with a fader. The Morton-sorted pairing is untested.
4. **Scan the venue.** Capture the Schaffhausen venue the afternoon before and play it as dust during the set, with the crowd inside the scan. Check venue rules and avoid faces.
5. **False-colour lidar mode.** Recolour any RGB cloud by depth or intensity and flip modes on hi-hats.

## Did you know?

- "House of Cards" used no cameras, and the crew drove the lidar bus at about 10 mph for clean scans. It won a D&AD Pencil and was included in the V&A's "Decode" exhibition ([James Frost](https://www.jamesfrost.tv/case-studies/radiohead)).
- The Clove Cigarette team chose Unreal over Cinema 4D partly because the X-Particles plugin would have cost about $1,000, while Unreal was free ([No Film School](https://nofilmschool.com/music-video-lidar-point-clouds-unreal-engine)).
- Splat files can be exported as SPZ, described as about 90 % smaller, or as PLY from Niantic's capture tools ([Niantic Spatial](https://www.nianticspatial.com/products/capture)).
- Kyle McDonald's home structured-light scanning guide needs only a camera and a projector ([Gizmodo](https://gizmodo.com/5437326/recreate-an-extraordinary-radiohead-video-with-ordinary-tech-and-this-guide)).

## Sources

Licensing: Poly Haven assets are CC0 ([Poly Haven licence](https://polyhaven.com/license)). The Radiohead data was released as open source for remixing; check its current licence before commercial use. Check the licence of every asset, and do not assume a public scan is free to use.

- [Wikipedia: Point cloud](https://en.wikipedia.org/wiki/Point_cloud)
- [James Frost: Radiohead case study](https://www.jamesfrost.tv/case-studies/radiohead)
- [arXiv 2308.04079, 3D Gaussian Splatting](https://arxiv.org/abs/2308.04079)
- [No Film School: Clove Cigarette](https://nofilmschool.com/music-video-lidar-point-clouds-unreal-engine)
- [Radiancefields.com: Helicopter](https://radiancefields.com/a-ap-rocky-releases-helicopter-music-video-featuring-gaussian-splatting)
- [Radiancefields.com: GSOPs](https://radiancefields.com/gsops-brings-native-gaussian-splatting-to-touchdesigner)
- [World Labs: Synth Splat](https://www.worldlabs.ai/labs/showcase/synth-splat)
- [Radiohead, "House of Cards" (YouTube)](https://www.youtube.com/watch?v=8nTFjVm9sTQ)
- [Derivative: gsLoader](https://derivative.ca/UserGuide/Palette:gsLoader)
- [Derivative: Point File In TOP](https://docs.derivative.ca/Point_File_In_TOP)
- [Derivative tutorial 05](https://derivative.ca/community-post/tutorial/touchdesigner-tutorial-05-3d-gaussian-splatting-trainingrefiningmanipulating)
- [Niantic Spatial: Capture](https://www.nianticspatial.com/products/capture)
- [Poly Haven licence](https://polyhaven.com/license)
- [Gizmodo: recreate the Radiohead video](https://gizmodo.com/5437326/recreate-an-extraordinary-radiohead-video-with-ordinary-tech-and-this-guide)
- [Commons file page (lead image)](https://commons.wikimedia.org/wiki/File:Ouster_OS1-64_lidar_point_cloud_of_intersection_of_Folsom_and_Dore_St,_San_Francisco.png)
- [Resolume ISF](https://resolume.com/support/en/isf)
- [Epic: LiDAR Point Cloud Plugin overview](https://dev.epicgames.com/documentation/unreal-engine/lidar-point-cloud-plugin-overview-in-unreal-engine)
- [CloudCompare: Edit > Subsample](https://cloudcompare.org/doc/wiki/index.php/Edit%5CSubsample)
- [CloudCompare: Mesh > Sample points](https://www.cloudcompare.org/doc/wiki/index.php/Mesh%5CSample_points)
- [Godot: MultiMesh](https://docs.godotengine.org/en/stable/classes/class_multimesh.html)
- [Godot: spatial shader reference](https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/spatial_shader.html)
