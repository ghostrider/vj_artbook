---
tags:
  - topology
  - sculpture
  - geometry
  - 3d
---

# Borromean Rings

> Three loops, no two linked, yet none can leave: cut one and the whole knot falls apart.

![3D illustration of intertwined rings on a black background](https://images.unsplash.com/photo-1632373564036-cc8b7ce520c6?w=1400)
*Similar look, not the original work. Photo: [Shubham Dhage](https://unsplash.com/photos/a-group-of-intertwined-rings-on-a-black-background-80FNtdZ6L98) on [Unsplash](https://unsplash.com/license).*

## The idea

Three loops. Take any two and they are not linked. Take all three and they cannot be pulled apart. Cut one loop and the other two fall free. Mathematicians call this a Brunnian link ([Gunn and Sullivan, Bridges 2008](https://archive.bridgesmathart.org/2008/bridges2008-63.pdf)).

The motif is old. The name comes from the coat of arms of the Borromeo family in northern Italy. The valknut, three interlaced triangles, appears on 7th-century Norse image stones, and a lost 13th-century French manuscript showed the rings as a Trinity symbol. Peter Tait put them in his 1876 knot catalogue, and the Ballantine beer logo gave them the nickname "Ballantine rings" ([Wikipedia](https://en.wikipedia.org/wiki/Borromean_rings)).

On a club screen the figure works because the drama is simple. Six over/under crossings, three colours, a 3-fold symmetry that loops cleanly on a bar. And it has a story that fits a drop: link, cut, relink.

## Origins and references

This chapter stands on two pieces of work. The video art lineage comes from the IMU logo video. The sculpture lineage comes from the Verhoeffs.

- **Charles Gunn and John M. Sullivan, "The Borromean Rings: A Video about the New IMU Logo"** (Bridges Leeuwarden 2008, pp. 63-70). A 5-minute computer-graphics video that premiered at the opening ceremony of ICM 2006. Scenes: Buckminster Fuller's jitterbug whose traced golden rectangles form the rings, photos of real Borromean rings, several ring shapes ending in the tight form, woven-rope texture, transparent tubes around a thin core, and a dissolve that removes two components. It ends on the logo ([paper page](https://archive.bridgesmathart.org/2008/bridges2008-63.html), [paper PDF](https://archive.bridgesmathart.org/2008/bridges2008-63.pdf), [IMU](https://www.mathunion.org/outreach/imu-logo-history)). Almost every scene in this chapter's design is a live version of one of these.
- **John Sullivan, IMU logo.** The logo shows the "tight" Borromean rings seen along the 3-fold axis. Adopted in 2006, announced at ICM 2006 in Madrid on 22 Aug 2006 ([IMU logo page](https://www.mathunion.org/outreach/logos/borromean-rings), [IMU logo history](https://www.mathunion.org/outreach/imu-logo-history)).
- **Tom and Koos Verhoeff, mitered Borromean sculptures** (Bridges 2015). Three families: hexagonal rings (including the octahedral "Trinity II"), cube weavings with right-angled zigzag beams ("Trinity III", a single band of non-convex octagons), and a closed triangular braid giving three identical loops ([paper](https://archive.bridgesmathart.org/2015/bridges2015-53.pdf)).
- **John Robinson**, sculptures Creation, Intuition and Genesis, cited in the Verhoeff paper ([Bridges 2015](https://archive.bridgesmathart.org/2015/bridges2015-53.pdf)).
- **Numberphile, "Borromean Olympic Rings"** (2012): explainer on the rings and their extension to odd numbers of rings ([archive](https://rosetta.to/u/numberphile/borromean-olympic-rings-numberphile)).

![IMU logo: the tight Borromean rings seen along their 3-fold axis](https://www.mathunion.org/fileadmin/IMU/Logo/IMU-logo-wt.png)
*IMU logo, John Sullivan, 2006. © International Mathematical Union. Shown as a quotation for reference; source: [mathunion.org](https://www.mathunion.org/outreach/imu-logo-history).*

## Visual language

Three interlocking loops in mutually perpendicular planes. Seen along the 3-fold axis they flatten into the logo view: a triskelion-like hexagon of crossings. The over/under alternation at the six crossings is the whole drama.

Two looks, taken from the two lineages:

- **Soft and optical** (Gunn/Sullivan): fat woven rope, glass-like tubes around a thin glowing core, rings dissolving away.
- **Hard and faceted** (Verhoeff): straight mitered beams, hexagons, zigzag cube weavings, in steel or bronze.

Colour is flat, one per ring: vermilion #E63B2E, gold #F5B700, cobalt #1E6FD9 on true black. Motion is a slow, linear spin. A 120 degree turn about the (1,1,1) axis maps the figure onto itself, so the loop point is invisible. One moment of violence: the cut, with ember edges #FF6A1A.

## See it

- [Gunn and Sullivan video, paper page](https://archive.bridgesmathart.org/2008/bridges2008-63.html) — abstract and PDF of the paper describing the IMU logo video (the video itself is not embedded)
- [IMU logo history, video download](https://www.mathunion.org/outreach/imu-logo-history) — states the 5-min video is downloadable as a .mov (IMULogo-big.mov)
- [Numberphile, Borromean Olympic Rings](https://www.youtube.com/watch?v=x6Ml4AEt0kk) — explainer with physical rings
- [Verhoeff sculptures paper](https://archive.bridgesmathart.org/2015/bridges2015-53.pdf) — photos and renders of the three sculpture families
- [Wolfram Demonstration, IMU rings](https://www.wolframcloud.com/objects/demonstrations/BorromeanRingsLogoOfTheInternationalMathematicalUnionIMU-source.nb) — interactive tight rings in icosahedral frame

## Code design

The set plays as four scenes: **Jitterbug** (a wireframe polyhedron breathes from cuboctahedron to icosahedron and its corners thicken into rings), **Families** (ellipse, hexagonal beam, square-beam cube weave, closed braid), **Glass** (clear tubes with a glowing core and rope stripes) and **Cut** (one ring burns away on the drop, the other two slide apart, then rejoin on the next phrase).

The geometry is one formula. The loops run through the cyclic permutations of the points (0, ±a, ±b). The same 12 points are the vertices of Fuller's jitterbug: a cuboctahedron at a = b, a regular icosahedron at a/b = 1/phi (the three golden rectangles of the textbook picture, [Wikipedia](https://en.wikipedia.org/wiki/Borromean_rings)), an octahedron at a = 0. One `jitter` parameter moves between them.

Each ring is a 2D outline (ellipse, hexagon or rectangle) swept by a tube cross-section (round, hex or square). Square and hex sections give mitered corners in the manner of the Verhoeff beams. The tube radius is capped at a safe value so tubes never touch. The braid is a Lissajous 1:2 plait of three strands around a main circle, crossfaded with a dither because an SDF morph would let tubes pass through each other. This is not the true minimal-length tight shape, which has 42 smooth pieces ([Bridges 2008 PDF](https://archive.bridgesmathart.org/2008/bridges2008-63.pdf)). The exact Verhoeff sculptures need a DCC model built from the paper ([Bridges 2015](https://archive.bridgesmathart.org/2015/bridges2015-53.pdf)).

Layer stack: black background, optional backdrop for refraction, rings as an SDF raymarch with premultiplied alpha (Normal blend), jitterbug wireframe (Add), optional afterimage, then bloom, slight chromatic aberration and grain. Rings stay Normal because Add would light up every on-screen overlap and lose the over/under reading.

=== "Resolume Wire"
    - **Render**: one custom ISF node "BorromeanRings" (rings plus wireframe) into an optional feedback node, Bloom and output. Keep it to a single ISF pass. Resolume documents ISF loading in connection with Wire ([Resolume ISF](https://resolume.com/support/en/isf)); test it in your Arena version before the gig.
    - **Control**: beat position divided by (4 x bars per 120 degrees), then fract gives `spinPhase`. Jitter, family, thick, glass and fov come from exposed floats, each through a `Smooth` (0.25 s; 0 for family steps).
    - **Cut state machine**: trigger, counter 0 to 1 to 0. Ramp `cut` 0 to 1 over 1 beat, then `slide` 0 to 1 over 2 beats. The rejoin trigger plays slide back, then cut back. `cutRing` from a 3-way selector or MIDI note mod 3.
    - **Audio**: low band to kick (attack 5 ms, release 180 ms), high band to hat (1 ms / 90 ms), mid band integrated to rope scroll.
    - **Arena as host**: the same ISF as an effect reads the layer below as `inputImage` for glass refraction. Bloom ([in Resolume since 7.2](https://resolume.com/blog/19880)) and FeedbackPro ([Arena 7.18.2+](https://www.resolume.com/blog/23113)) handle post and afterimage.

=== "Unreal Engine 5.7"
    - **Geometry**: one tube mesh per ring (384 x 24 closed cylinder grid). The material shapes everything in World Position Offset from the centre curve, frame and tube offset, and passes the analytic normal through a Vertex Interpolator. The braid is a fourth mesh set with its own WPO, crossfaded by dither.
    - **Materials**: solid (rope as analytic normal), glass (Translucent, refraction IOR 1.45, Fresnel opacity), core (Unlit additive, ring colour x 3). Dissolve is an Opacity Mask from the cut gate, plus an ember band.
    - **Niagara** `NE_CutEmbers`: GPU sim placing particles on the cut front, curl noise, additive sprites from #FF6A1A to #FFF4E0.
    - **Jitterbug**: a Blueprint computes the 12 vertices and drives instanced cylinders and spheres.
    - **Control**: `BP_Borromean` writes a Material Parameter Collection and rotates the actor about (1,1,1). CineCamera on that axis; depth maps to focal length, orthographic for the pure logo view.
    - **Remote Control**: preset with jitter, family, thick, glass, depth, spin, tumble and functions TriggerCut(ring) and Rejoin, bound to MIDI CC and OSC (e.g. `/borromean/jitter`).
    - **Output**: Spout or NDI into Arena. Test alpha propagation; otherwise render on black and use Add or luma key.

=== "GODOT"
    - **Option A (2D)**: a full-screen `ColorRect` with a `canvas_item` shader holding the raymarch and wireframe. A GDScript on the root computes the control values and sets uniforms. `backdrop` sampler for refraction.
    - **Option B (3D)**: three `MeshInstance3D` with a 384 x 24 tube `ArrayMesh` built once in GDScript, shaped in a spatial `vertex()` shader. Glass uses `hint_screen_texture`, cores use `unshaded, blend_add`. Embers via `GPUParticles3D` moved along the dissolve front.
    - **Feedback**: `SubViewport` with Clear Mode Never plus a ping-pong pair.
    - **Output**: no built-in Spout or NDI. Use a community extension (check it against your Godot version) or capture the window.

## On stage

**Audio reactivity**

| band | drives | smoothing |
|------|--------|-----------|
| kick (40-120 Hz) | tube radius +12 %, capped at the safe radius | attack 5 ms, release 180 ms |
| bass (60-250 Hz) | core glow 1.0-2.5x in glass mode | attack 80 ms, release 600 ms |
| mids (250 Hz-2 kHz) | rope scroll speed | attack 150 ms, release 1 s |
| hats (6-12 kHz) | white spark at each of the six crossings | attack 1 ms, release 90 ms |

Spin speed does not react to audio, so the 3-fold loop stays locked. The cut is never audio-triggered. A false cut mid-phrase breaks the story.

**MIDI (KeyStep 37 / BeatStep Pro)**

| control | parameter |
|---------|-----------|
| BSP encoder 1 | jitter (cuboctahedron, icosahedron, octahedron collapse) |
| BSP encoder 2 | family, snapped on pads 1-4 |
| KeyStep mod strip | thick |
| BSP encoder 3 | glass |
| BSP encoder 4 | depth (fov 8-60 degrees; narrow is the flat logo view) |
| BSP pad 16 / pad 15 | cut / rejoin |
| KeyStep C3 / D3 / E3 | cut ring 1 / 2 / 3 |

**Timing.** Default spin: 120 degrees per 2 bars. Change family only on phrase starts. On the drop, cut on beat 1 (dissolve over 1 beat, slide over 2 beats), then hold 4-8 bars. Rejoin on the next phrase start. In a break: glass on, logo view, slow spin, afterimage at 40 %.

**Layering.** The ring alpha is a good matte: put footage above and let it show only inside the rings (solid mode works best). Output as Spout or NDI with alpha over feedback, grids or particles. Never run feedback during the cut, the embers turn into streaks.

**Tips.** Rotate the three colours by one slot every 8 bars together with a 120 degree turn, or you get a colour pop. Keep HDR peaks at or below 4 so bloom does not wash out the dark gaps between rings. Render at 0.5-0.75x on large walls. Drop the rope texture where stripes get thinner than about 4 px at the wall's pixel pitch, or LED refresh produces moiré. Glass on pure black has nothing to refract; give it a soft-box environment or a dim backdrop. A plain rotating wireframe is boring. It needs material, light and the link, cut, relink story.

## Variations

- Morph between families: ellipses, Verhoeff-style hexagons, cube zigzag weave, triangular braid.
- Jitterbug motion that traces the rings, after the Gunn/Sullivan video.
- Solo ending, as in the video: cut two rings one beat apart and leave one in the logo view.
- Odd-number Brunnian links (5, 7 rings) stepping up with the energy of the set.
- Faceted beam rendering against glassy tube rendering, crossfaded by a fader.
- Mono steel or bronze "sculpture" scene.

## Did you know?

- Freedman and Skora proved that no Brunnian link can be exactly circular. Ellipses of tiny eccentricity work ([Wikipedia](https://en.wikipedia.org/wiki/Borromean_rings)).
- The IMU logo video was rendered from more than 5,000 RenderMan frames, built from the authors' jReality Java scene graph and assembled in Final Cut Pro. A stereo 3D version for theatres was also made ([Bridges 2008 PDF](https://archive.bridgesmathart.org/2008/bridges2008-63.pdf)).
- The tight Borromean rings have pyritohedral symmetry and are under 0.1 % shorter than a version made of unit-circle arcs ([Bridges 2008 PDF](https://archive.bridgesmathart.org/2008/bridges2008-63.pdf)).
- The complement of the link splits into two ideal regular octahedra; its hyperbolic volume is about 7.3277 ([Wikipedia](https://en.wikipedia.org/wiki/Borromean_rings)).

## Sources

- <https://archive.bridgesmathart.org/2015/bridges2015-53.pdf>
- <https://archive.bridgesmathart.org/2008/bridges2008-63.html>
- <https://archive.bridgesmathart.org/2008/bridges2008-63.pdf>
- <https://www.mathunion.org/outreach/imu-logo-history>
- <https://www.mathunion.org/outreach/logos/borromean-rings>
- <https://en.wikipedia.org/wiki/Borromean_rings>
- <https://rosetta.to/u/numberphile/borromean-olympic-rings-numberphile>
- <https://www.youtube.com/watch?v=x6Ml4AEt0kk>
- <https://www.wolframcloud.com/objects/demonstrations/BorromeanRingsLogoOfTheInternationalMathematicalUnionIMU-source.nb>
- <https://resolume.com/support/en/isf>
- <https://resolume.com/blog/19880>
- <https://www.resolume.com/blog/23113>
