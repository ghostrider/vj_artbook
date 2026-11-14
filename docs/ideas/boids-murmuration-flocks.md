---
tags:
  - nature
  - flocking
  - agents
  - particles
---

# Boids Murmuration Flocks

> Three local rules, thousands of agents, one dark body that folds, splits and breathes with the kick.

![A murmuration of starlings, several hundred birds forming a dense cloud](https://upload.wikimedia.org/wikipedia/commons/thumb/7/7d/Murmuration_11-2025.jpg/1280px-Murmuration_11-2025.jpg)
*A murmuration of starlings, several hundred birds forming a dense cloud. Image: [Skander zarrad](https://commons.wikimedia.org/wiki/File:Murmuration_11-2025.jpg), [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0).*

## The idea

Every bird looks only at its neighbours. It steers away from crowding, matches their heading and drifts toward their centre. Nobody leads. Still, the whole cloud turns as one. Craig Reynolds called the model "boids", short for "bird-oid object" ([Wikipedia](https://en.wikipedia.org/wiki/Boids), [Reynolds](https://www.red3d.com/cwr/boids/)).

On a club screen the flock is a single living body made of thousands of tiny marks. It stretches into a ribbon, thins into single birds, then a wave of turning runs through it. The music does not strobe the image. It changes how the crowd behaves: the kick pulls the flock together, the bass loosens it, a predator on the beat tears it apart.

The risk is the uniform swirl. With static weights and no predator or goal, a flock only circles and gets boring on a big screen. The design below keeps it moving with a beat-driven predator, turn waves and slow parameter glides.

## Origins and references

- **Craig Reynolds, boids (1986/87).** Presented at SIGGRAPH 1987 as "Flocks, herds and schools: A distributed behavioral model". First boid film: *Stanley and Stella in: Breaking the Ice*. [Reynolds](https://www.red3d.com/cwr/boids/)
- **Craig Reynolds, steering behaviors.** The wider family (seek, flee, wander, flow-field following) and the open-source OpenSteer. [Steering behaviors](https://www.red3d.com/cwr/steer/)
- **Ichiro Aoki (1982).** Closely related earlier work. [Wikipedia](https://en.wikipedia.org/wiki/Boids)
- ***Batman Returns* (1992).** First feature film to use boids, for bat swarms and penguin armies. [Reynolds](https://www.red3d.com/cwr/boids/)
- **STARFLAG project.** Filmed real flocks in 3D. Each starling reacts to about seven neighbours, regardless of distance. [The Contrarian](https://contrarian.ca/2012/06/20/the-seventh-starling/)
- **Robert Hodgin, *Murmuration*.** A starling-style flock reworked over about two decades: roughly 1,000 CPU boids, then 40,000 on the GPU, then 200,000 to 500,000 in Houdini. A friend's compute-shader port ran at 30+ fps with over a million. [Hodgin](https://roberthodgin.com/project/murmuration)
- **Sebastian Lague, "Coding Adventure: Boids" (2019).** A popular Unity tutorial. [YouTube](https://www.youtube.com/watch?v=bqtqltqcQhw)

## Visual language

Thousands of small agents form one fluid body. Look for dense dark ribbons, sudden thinning, waves of turning that pass through the cloud, splits and merges. Single-colour points or tiny oriented wedges work best, on a dark or a light field.

The default "Night" look: pale silver birds on true black. Dense, near areas glow warm cream (#EDE6D6). Thin, far areas fade to cool grey-blue (#2E3644). Additive blending is what turns tiny marks into smoke. The "Dusk" look reverses it: near-black birds (#0A0A0E) on a dim violet-to-amber gradient (#141830 to #4A3557 to #B8704A), closer to the photo.

Motion is slow and large. Birds fly at 8 to 16 units per second in a cloud about 100 units across, so a bird needs roughly 6 to 12 seconds to cross the flock. A warm orange accent (#FF6A3D) appears only at events: the front of a turn wave and the zone around the predator.

## See it

- [Craig Reynolds, Boids](https://www.red3d.com/cwr/boids/) — the original background page on the model, with links to demos.
- [Robert Hodgin, Murmuration](https://roberthodgin.com/project/murmuration) — project page of a large starling-style flock simulation, with an image.
- [Coding Adventure: Boids](https://www.youtube.com/watch?v=bqtqltqcQhw) — Sebastian Lague's tutorial video (channel Sebastian Lague) showing the rules in motion.
- [I Didn't know Birds use Math in Murmurations! - Smarter Every Day 234](https://www.youtube.com/watch?v=4LWmRuB-uNU) — channel SmarterEveryDay, on murmuration behaviour.

## Code design

The flock simulation runs in a sim host (Unreal Engine 5.7, TouchDesigner or Godot). It reaches Resolume over Spout (same machine) or NDI (second machine). Resolume handles colour, compositing and post. Wire has no vertex shaders ([Resolume ISF](https://resolume.com/support/en/isf)), so it cannot draw thousands of moving points. It gets the colourize effect and a small fallback flock.

Core design:

- **State.** Each boid keeps position and velocity in 32-bit floats. Steering reads the previous state and writes a new buffer, then they swap (ping-pong). The [three.js GPGPU birds example](https://raw.githubusercontent.com/mrdoob/three.js/dev/examples/webgl_gpgpu_birds.html) does this with two float textures.
- **Rules.** Separation, alignment, cohesion, plus flee (predator), a soft spherical bound, curl noise, goal and wave forces. Neighbours are the `k` nearest (default 7, after the STARFLAG finding), inside a search radius and a field of view. A minimum speed stops the flock from collapsing into a buzzing ball. Default values are starting points to tune by eye, not values from a source.
- **Neighbour search.** The naive version is O(n²). Reynolds notes that sorting by location brings it close to linear ([Reynolds](https://www.red3d.com/cwr/boids/)). Preferred method: a uniform grid with counting sort, the same sort-then-find-cell-starts idea as the [GPU Gems 3 broad-phase chapter](https://developer.nvidia.com/gpugems/gpugems3/part-v-physics-simulation/chapter-32-broad-phase-collision-detection-cuda). Alternatives are the built-in Niagara and TouchDesigner neighbour tools.
- **Render.** Each boid is a velocity-stretched wedge, drawn additively into a float density buffer. Tonemap, pack density, event glow and depth into RGB, and send that over Spout or NDI. Resolume colourizes it.
- **Murmuration, not swirl.** A predator on a beat-synced path, dives on bar downbeats, turn waves on drops, and a kick "breath" on cohesion. These are design proposals, not sourced values.

=== "Resolume Wire"
    - Colourize patch: effect texture input into an ISF node with the colourize shader (loaded under Attributes, Fragment Shader, per [Resolume ISF](https://resolume.com/support/en/isf)). Expose `near`, `mid`, `far`, `accent`, `skyMode` and an exposure trim.
    - Mask patch (optional): two clips plus the flock texture, `mix(B, A, smoothstep(0.05, 0.3, flock.r))`, to reveal one clip inside the flock and another outside.
    - A full flock is not practical in Wire. There are no vertex shaders, and fragment passes cannot scatter, so no neighbour grid.
    - Fallback W0 (design proposal, untested): one multipass ISF with about 1,024 2D boids (estimate). Each boid keeps a cache of its `k` best neighbours, refreshed from its cached neighbours, their neighbours and a few random boids. A gather render at 640x360 draws it. It uses `PASSES`, `TARGET`, `PERSISTENT` and `FLOAT` from the [ISF multipass spec](https://docs.isf.video/ref_multipass.html). Resolume does not document these for Wire, so test whether your version keeps buffers between frames.
    - In Arena: Spout source `Flock` on its own layer, the Wire colourize effect on that clip, then Bloom (threshold 0.75). Blend mode Normal for Night, Multiply for Dusk or matte, Add over video.

=== "Unreal Engine 5.7"
    - One Niagara emitter, Sim Target GPUCompute, fixed bounds, local space off. Boids are particles with no lifetime kill. Particle attributes are the persistent state.
    - Neighbour search with a Neighbor Grid 3D data interface (`MaxNeighborsPerCell` around 16; [Epic API](https://dev.epicgames.com/documentation/unreal-engine/API/Plugins/Niagara/UNiagaraDataInterfaceNeighborGri-)). Three simulation stages: clear the grid, populate it from the particles, then steer by reading the 27 cells around each particle. Girardot's public Niagara boids system uses this data interface ([80.lv](https://80.lv/articles/check-out-a-new-niagara-based-boids-system-set-up-in-ue5-3)). Function names have changed between engine versions, so check the list in 5.7 first.
    - Render with a wedge mesh or velocity-aligned sprites, an unlit additive material, into an RGBA16F render target. A post-process material tonemaps and packs it, then your Spout or NDI plugin sends it. Or render in-scene with a sky for a "real" Dusk.
    - Remote Control: a Blueprint actor with `Tightness`, `Spacing`, `Neighbours`, `Speed`, `Predator`, `Shape` and the functions `Dive`, `TurnWave`, `Burst`, `Reseed`, bound to OSC and MIDI.
    - Boid count: 50,000 to 300,000 on a current desktop GPU (estimate, not measured). Profile with `stat GPU`.

=== "GODOT"
    - Godot 4, Forward+ or Mobile. Compute shaders need the RenderingDevice renderers ([compute shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/compute_shaders.html)). Results are shown through [Texture2DRD](https://docs.godotengine.org/en/stable/classes/class_texture2drd.html).
    - State: two pairs of RGBA32F storage images (position and velocity, A and B), swapped each tick. Grid buffers are plain storage buffers. Free all RIDs yourself.
    - Neighbour search: grid with counting sort as four short dispatches (clear, bin, prefix sum, scatter), then the steering dispatch. Avoid `sync()`, and keep dispatches short to avoid Windows TDR resets, as the docs warn.
    - Render: a MultiMeshInstance3D whose vertex shader reads position and velocity from the state texture and builds the wedge. Unshaded, additive, in a SubViewport with HDR, then a grade shader.
    - No built-in Spout or NDI output. Run fullscreen on its own output, or use a community addon.
    - Boid count: 100,000 to 500,000 (estimate). This is the leanest path.

TouchDesigner is the recommended sim host on the Windows VJ machine. With POPs, a Feedback POP holds the state, the [Neighbor POP](https://docs.derivative.ca/Neighbor_POP) finds nearby points, and a GLSL POP does the steering. Estimate: 50,000 to 200,000 boids at 60 fps. In older builds, GLSL TOPs with a persistent neighbour cache work instead. That cache method is an untested proposal.

## On stage

**Audio reactivity**

- Kick: breath. Cohesion jumps ×2.5 with a 250 ms release, and exposure rises 25 % for 200 ms.
- Bass: spacing rises up to +0.8 so the flock loosens while bass sustains.
- Snare or clap: a small local turn wave at a random boid, at most one per beat.
- Hats: heading jitter of ±4° on 10 % of the boids, a shimmer.
- Loudness: top speed +0 to 30 %, with slow smoothing, so the flock gets faster in loud sections and not on every hit.
- Keep audio off the neighbour count `k` and the search radius on short time scales. Rapid neighbour changes look like jitter, not music.

**MIDI mapping**

| control | does |
|---------|------|
| BeatStep Pro knob 1 | tightness (cohesion) |
| knob 2 | spacing (separation) |
| knob 3 | neighbours `k`, 2 to 16 (low is swarm-like, high gives long ribbons) |
| knob 4 | speed, 0 freezes the flock |
| knob 5 | predator strength and radius |
| pad 1 | predator dive now |
| pad 2 | turn wave |
| pad 3 | burst (spacing ×3 for 1 beat) |
| pad 4 | reseed |
| pads 5-7 | dive every 1, 2 or 4 bars |
| pad 8 | Night/Dusk toggle |
| KeyStep 37 mod strip | shape (pulls the flock into a logo) |

**Structure**

- Phrase (8 or 16 bars): glide `k` between 4 (loose, grainy) and 12 (tight ribbon) over 2 bars.
- Build-up: speed up, tighten spacing, raise `k`, predator dives every bar, then every half bar in the last 2 bars.
- Drop: a turn wave from the centre with an orange flash on its front. Separation jumps for 1 beat so the cloud bursts open, then everything glides back over 2 bars.
- Break: slower, more noise, predator parked far away. The flock thins into a drifting veil. Setting `dt` to 0 freezes the cloud while the camera keeps orbiting.

**Layering**

- Night: flock over black in Normal. Over video or a UE scene use Add or Screen, since Normal hides the video in the black gaps.
- Dusk: sky gradient below, flock in Multiply. Birds darken whatever is underneath, even live camera.
- Matte: video on a layer below, flock in Multiply, so the video shows only through the birds.
- Mask: use the flock density as a luma mask to reveal one clip inside and another outside.
- Order: black floor, colourize, Bloom (threshold 0.75), then 1 % grain last to hide 8-bit banding.

**Performance**

- The cost is neighbour reads, not memory. Lower the candidate cap before the boid count. Keep the cap: dense clumps on every kick otherwise cause frame time spikes.
- 30,000 to 60,000 boids already read as a murmuration on a 1080p wall (design estimate). More boids give a smokier body, not more detail.
- Render the density buffer at 0.75× and let bloom hide the upscale. On low-resolution LED walls, render at native size with 1 px points.
- LED walls: keep true black, cap highlights near 90 % and the Dusk sky at about 55 % luminance, or the sky will draw more power than the birds.
- Wide strips: stretch the bound into an ellipsoid along the long axis so the flock uses the full width.
- Pitfalls: in-place state updates, no minimum speed, a box-shaped bound (birds pile in corners), static weights with no predator, half-float positions.

## Variations

1. **Predator-driven.** A MIDI-note-controlled or beat-orbiting predator that triggers splits.
2. **Topological k-neighbour flocks.** A fader changes `k` live, from a loose swarm to a tight one.
3. **Density as image.** Render the flock into a density buffer and use it as a luminance mask for feedback or video.
4. **Multi-species.** Two or three flocks in different colours (#6FD3FF, #FFC94A) that avoid or chase each other on different beats.
5. **Letters or shapes.** A goal field makes the flock briefly form a logo, then dissolve on the drop.

## Did you know?

- Boids first appeared in a feature film in 1992, in *Batman Returns* ([Reynolds](https://www.red3d.com/cwr/boids/)).
- Real starlings follow a topological rule: about seven neighbours, not a fixed distance. In dense flocks the interaction range shrinks ([The Contrarian](https://contrarian.ca/2012/06/20/the-seventh-starling/)).
- Hodgin's flock simulation grew from about 1,000 CPU agents to over a million in a GPU compute-shader port ([Hodgin](https://roberthodgin.com/project/murmuration)).
- The boid model has been applied to swarm robotics, data visualisation and optimisation ([Wikipedia](https://en.wikipedia.org/wiki/Boids)).

## Sources

- [Boids, Wikipedia](https://en.wikipedia.org/wiki/Boids)
- [Craig Reynolds, Boids](https://www.red3d.com/cwr/boids/)
- [Craig Reynolds, Steering Behaviors](https://www.red3d.com/cwr/steer/)
- [Robert Hodgin, Murmuration](https://roberthodgin.com/project/murmuration)
- [The seventh starling, The Contrarian](https://contrarian.ca/2012/06/20/the-seventh-starling/)
- [Coding Adventure: Boids (YouTube)](https://www.youtube.com/watch?v=bqtqltqcQhw)
- [Smarter Every Day 234 (YouTube)](https://www.youtube.com/watch?v=4LWmRuB-uNU)
- [Commons file page](https://commons.wikimedia.org/wiki/File:Murmuration_11-2025.jpg)
- [Resolume ISF](https://resolume.com/support/en/isf)
- [ISF multipass spec](https://docs.isf.video/ref_multipass.html)
- [three.js GPGPU birds example](https://raw.githubusercontent.com/mrdoob/three.js/dev/examples/webgl_gpgpu_birds.html)
- [NVIDIA GPU Gems 3, ch. 32: Broad-Phase Collision Detection with CUDA](https://developer.nvidia.com/gpugems/gpugems3/part-v-physics-simulation/chapter-32-broad-phase-collision-detection-cuda)
- [Niagara Neighbor Grid 3D, Epic API](https://dev.epicgames.com/documentation/unreal-engine/API/Plugins/Niagara/UNiagaraDataInterfaceNeighborGri-)
- [Girardot's Niagara boids, 80.lv](https://80.lv/articles/check-out-a-new-niagara-based-boids-system-set-up-in-ue5-3)
- [Neighbor POP, Derivative](https://docs.derivative.ca/Neighbor_POP)
- [Godot compute shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/compute_shaders.html)
- [Godot Texture2DRD](https://docs.godotengine.org/en/stable/classes/class_texture2drd.html)
