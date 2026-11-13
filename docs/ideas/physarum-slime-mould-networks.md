---
tags:
  - biology
  - agents
  - particles
  - network
---

# Physarum Slime Mould Networks

> Millions of tiny agents follow each other's trails until a living web of glowing veins grows across the wall.

![Six-panel illustration (after Nakagaki's study) of slime mould growing from one food source into an optimised network linking food points over 26 hours](https://upload.wikimedia.org/wikipedia/commons/thumb/e/ea/Physarum_polycephalum_network.jpg/1280px-Physarum_polycephalum_network.jpg)
*Six-panel illustration (after Nakagaki's study) of slime mould growing from one food source into an optimised network linking food points over 26 hours. Image: [Tim Tim (VD fr)](https://commons.wikimedia.org/wiki/File:Physarum_polycephalum_network.jpg), [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0).*

## The idea

*Physarum polycephalum* is an acellular slime mould, first described in 1822 and used as a model organism for motility and chemotaxis ([Wikipedia](https://en.wikipedia.org/wiki/Physarum_polycephalum)). It became famous for its networks. In a 2010 Science study, oat flakes marked Tokyo and 36 surrounding towns, and the mould grew a network resembling the real rail system ([Wikipedia](https://en.wikipedia.org/wiki/Physarum_polycephalum)).

Artists use a simple agent model of this behaviour, from Jeff Jones (2010). Each agent has a position, a heading and three sensors. It reads the trail ahead, turns toward the stronger side and leaves a trail of its own. Trails reinforce, weak ones fade, and the strong ones harden into veins ([Sage Jenson](https://cargocollective.com/sagejenson/physarum)).

On a club screen it works because it is slow and alive. Nothing cuts. The network drifts, tears, heals and reorganises, so the music can be heard in how the structure behaves instead of in flashing.

## Origins and references

- **Jeff Jones, 2010.** "Characteristics of pattern formation and evolution in approximations of Physarum transport networks", the agent model behind most artistic versions ([credited by Jenson](https://cargocollective.com/sagejenson/physarum)). The research could not open the paper itself, so model details here come from Jenson's page.
- **Sage Jenson.** Adapted the model to real time. His page reports 5–10 million particles on a GTX 1070 ([write-up](https://cargocollective.com/sagejenson/physarum)). *Tapestry* is a graphic score through the model's parameter space ([Verse](https://verse.works/series/tapestry-by-sage-jenson)). *Primordium* (2022) was his first web-based solo show ([This Is Paper](https://thisispaper.com/mag/primordium-sage-jenson)).
- ***Dismantling*, Berlin, 2019.** A live performance and exhibition by Jenson and Kit Kuksenok using the simulation ([project page](https://ksen0.github.io/speculative-sim/)).
- **Heather Barnett.** Works with the living organism as a self-organising system ([TED](https://www.ted.com/talks/heather_barnett_what_humans_can_learn_from_semi_intelligent_slime)).
- **Amanda Ghassaei.** A WebGL demo where touch adds attractant, with presets ([demo](https://apps.amandaghassaei.com/gpu-io/examples/physarum/)).
- **Nakagaki, Yamada, Tóth, 2000.** The maze study: the mould withdrew from dead ends and kept the shortest route between two food sources ([Wikipedia](https://en.wikipedia.org/wiki/Physarum_polycephalum)).

## Visual language

A black wall slowly fills with glowing veins. Thin filaments branch, merge and close into loops. Depending on the parameters, the image moves between fine fibre fields, honeycomb cells, fingerprint swirls and dotted colonies. Veins thicken where many agents agree and fade where they leave.

Motion is continuous and slow. Close up, a cloud of agents seems to crawl along the veins. On a kick, a small share of agents loses its way, the network tears and heals within the beat. On the drop, a burst from the centre throws the structure outward and it re-forms as a new topology.

The default colour is bioluminescent: black floor, deep teal for weak trails, green for veins, warm cream for the densest junctions. An optional magenta "contagion" can spread through the network on the beat. Contrast comes from the black gaps between veins.

## See it

- [Coding Adventure: Ant and Slime Simulations](https://www.youtube.com/watch?v=X-iSQQgOd1A) (YouTube, Sebastian Lague) — the slime part shows agents forming trails and veins in a GPU sim.
- [What humans can learn from semi-intelligent slime](https://www.youtube.com/watch?v=2UxGrde1NDA) (YouTube, TED) — Heather Barnett's 2014 talk with the living organism growing networks.
- [Amanda Ghassaei, Physarum Transport Network](https://apps.amandaghassaei.com/gpu-io/examples/physarum/) — live WebGL demo with presets.
- [Sage Jenson, Physarum](https://cargocollective.com/sagejenson/physarum) — write-up with network, contagion and explosion GIFs.
- [Sage Jenson, Tapestry on Verse](https://verse.works/series/tapestry-by-sage-jenson) — artwork page for the Physarum phase-space score.
- [Jenson & Kuksenok, speculative sim](https://ksen0.github.io/speculative-sim/) — project page for *Dismantling*.

## Code design

The agent simulation runs in a sim host (TouchDesigner, Unreal Engine 5.7 or Godot) and reaches Resolume over Spout or NDI. Resolume handles colour, compositing and post. Wire cannot run the full simulation, because it has no vertex shaders and fragment passes cannot scatter ([Resolume ISF](https://resolume.com/support/en/isf)).

The core loop, after Jones as described by Jenson ([Sage Jenson](https://cargocollective.com/sagejenson/physarum)):

1. Each agent samples the trail at three points (front-left, front, front-right) and turns toward the strongest. Then it moves and wraps around the edges.
2. Each agent deposits into a trail map.
3. The trail is blurred with a 3x3 mean and faded by a decay factor (default 0.90, Ghassaei's value).
4. The trail is tonemapped (`v = 1 − exp(−t · exposure)`), clamped to a true black floor, and sent as an 8-bit image. Resolume maps it through a five-stop colour ramp.

There is no collision step: Jenson usually skips Jones's one-agent-per-cell rule because the patterns look better and every agent can run in parallel ([Sage Jenson](https://cargocollective.com/sagejenson/physarum)). Agent counts below are starting-point estimates, not benchmarks. A useful density is about 0.35 agents per trail pixel (Ghassaei's default), which is about 323 k agents at 1280×720.

Five presets from Ghassaei's source give five distinct topologies ([gpu-io source](https://raw.githubusercontent.com/amandaghassaei/gpu-io/main/examples/physarum/index.js)). The design jumps between them without resetting the agents, so the network morphs over 1–3 s.

| preset | sensor distance (px) | sensor angle (°) | step size (px) | rotation angle (°) |
|--------|----|----|----|----|
| Fibers | 18 | 5.5 | 2 | 45 |
| Fingerprint | 14 | 70 | 1.5 | −25 |
| Honeycomb | 7.5 | 90 | 2 | −45 |
| Net | 18 | 90 | 2 | −16 |
| Dots | 26 | 5.5 | 1.5 | −70 |

=== "Resolume Wire"
    Wire runs the colourize stage, not the simulation. The main patch takes the Spout texture, passes it through an ISF node with a five-colour ramp (R = trail, G = contagion, B = agent density for sparkle) and exposes the colours as parameters in Arena. Load the shader under Attributes → Fragment Shader ([Resolume ISF](https://resolume.com/support/en/isf)). An optional second patch displaces video by the trail gradient (0–20 px).

    In Arena, send a white logo or spectrum spots from a dedicated layer through Advanced Output to a Spout output `PhysarumInject`. The sim reads it as attractant.

    Fallback, untested and an estimate only: a grid-agent version as one multipass ISF with persistent float buffers ([ISF multipass spec](https://docs.isf.video/ref_multipass.html)). Agents live one per cell, so this is Jones's collision rule. At 640×360 and about 25 % occupancy that is roughly 58 k agents, with a coarser, blockier look. Resolume's ISF page does not mention multipass or persistent buffers, so test that the buffers keep their content across frames in your Wire version first.

=== "Unreal Engine 5.7"
    One Niagara GPU emitter, `NS_Physarum`. Agents are particles with a custom `Heading` attribute, spawned by one burst (start with 0.5–2 M, check `stat GPU`). They live until the system resets, and your own module writes positions directly.

    The trail is a Grid2D Collection at 1280×720 with half-float format ([Grid2D Collection API](https://dev.epicgames.com/documentation/en-us/unreal-engine/python-api/class/NiagaraDataInterfaceGrid2DCollection)). A second grid, `Deposit`, is cleared each tick. Stage order: particle update (sense, turn, move), a deposit stage over particles, a diffuse-and-decay stage over the grid, and an export stage that writes the graded image to a Render Target 2D, as in [80.lv's render-target walkthrough](https://80.lv/articles/niagara-system-integration-with-render-targets-in-unreal-engine-5). Grid function names vary between engine versions, so search the Grid2D node list in 5.7.

    Send the render target out with a Spout or NDI plugin, or use it as World Position Offset on a dense plane so the veins become terrain. Control comes from a Remote Control Preset bound to OSC and MIDI, with variables such as Topology, Scale, Turn, Speed, Persistence and Scatter, and triggers for Explode, Infect and Reseed.

=== "GODOT"
    Godot 4 with Forward+ or Mobile, using compute shaders on the main RenderingDevice so the trail can be shown directly through a Texture2DRD ([compute shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/compute_shaders.html), [Texture2DRD](https://docs.godotengine.org/en/stable/classes/class_texture2drd.html)). Record dispatches inside `RenderingServer.call_on_render_thread`.

    Agents live in a storage buffer of vec4 (`x, y, heading, flags`), 16 bytes each. The trail is two storage images in A/B ping-pong, plus a `R32_UINT` deposit image written with atomic add. Per frame: one dispatch for agent update and deposit, one for diffuse and decay, then swap. Free RIDs yourself, and avoid `sync()`. Start with 1–4 M agents as an estimate. Godot has no built-in Spout or NDI output, so run it fullscreen on its own output or use a community addon.

    TouchDesigner is the design's recommended sim host on a Windows VJ machine (a feedback-TOP loop for agents, instanced points for deposit). Lague's [Slime-Simulation](https://github.com/SebLague/Slime-Simulation) (Unity, GPL-3.0) is a good companion for the Godot compute route.

## On stage

**Audio reactivity.** Counted in beats, not frames, so events stay on the grid if the frame rate drops.

| band | drives |
|------|--------|
| Kick, 40–120 Hz | scatter 4 % of agents for one tick, exposure +30 % for about 250 ms |
| Bass, 60–250 Hz | rotation angle ±15°, slow, so topology bends instead of flickering |
| Snare / clap | small explosion at a random point |
| Hats, 6–12 kHz | brief decay dip (−0.03, about 80 ms), a shimmer without structural change |
| Spectrum, 8 bands | attractant discs on a circle, so agents grow toward active frequencies |
| RMS loudness | substeps 1 → 2, only in drop sections |

**MIDI.** BeatStep Pro knobs 1–6: topology (sensor angle), scale (sensor distance), turn (rotation angle), speed (step size), persistence (decay), scatter. Pads 1–5 jump to the presets Fibers, Fingerprint, Honeycomb, Net, Dots (glide, no reset). Pad 6 is an explosion, pad 7 infects agents, pad 8 reseeds from the centre. The KeyStep 37 mod strip sets inject gain for the logo or audio attractors. Map the controls in the sim host, since the parameters live there. Use Arena for colour, opacity and bloom, and share tempo with Ableton Link.

**Layering.** Over black, use Normal. Over video or a UE scene, use Add or Screen, never Normal, because the black gaps would hide the picture. For matte mode, put video below and the trail in Multiply, so the picture shows only inside the veins. Do not add Resolume feedback on top: the trail is already the feedback loop, and more smears the filaments into fog. Post chain: bloom (threshold 0.7), then 1.5 % grain last, which hides banding in the dark teal.

**Arc of a set.**

- Build-up, last 8 bars: decay 0.90 → 0.97, step size 2 → 3, scatter from every beat to every eighth.
- Drop: explosion from the centre, switch to Net, infect about 500 agents. Magenta spreads over the first 4 bars.
- Break: step size 0.3 and scatter off. Step 0, decay 1.0 and diffuse 0 freezes the network as a still image.
- Every 8–16 bars: reseed from a small disc in the centre, so the network grows out again.

**Tips.** A settled network is the main way this goes boring. Keep a slow sensor-angle drift (±10° per 8 bars) running. On LED walls, keep everything below v = 0.02 true black, cap highlights near 90 %, and do not lift the floor. Size the trail to the wall: if it is not 720 px high, scale distances by `trailHeight / 720`. Send the tonemapped 8-bit image, never raw trail values. Pitfalls: half-float agent positions (visible grid banding), resetting agents on every preset change (loses the morph), and bloom with a low threshold.

## Variations

- **Attractor from audio.** Spectrum bins drop attractant at fixed screen positions, so the network grows toward the bass and treble spots.
- **Contagion mode.** Infected agents lay a second trail channel that spreads ([Sage Jenson](https://cargocollective.com/sagejenson/physarum)). On the beat, infect a few agents and colour them magenta.
- **Text or logo seed.** Inject a 1000a mask so the network grows around or along the letters.
- **Multi-species.** Two or three populations with separate trail maps, each attracted to its own trail and repelled by the others, one hue each (amber and blue in the design).
- **Network as geometry.** Use the trail as a height field or Niagara input, so filaments become 3D terrain or lights.
- **Displacement.** Offset video UVs by the trail gradient so the picture ripples along the veins.

## Did you know?

- The Tokyo experiment used oat flakes for Tokyo and 36 surrounding towns, and the mould's network showed efficiency and fault tolerance comparable to the real rail system ([Wikipedia](https://en.wikipedia.org/wiki/Physarum_polycephalum)).
- In the 2000 maze experiment, the mould withdrew from dead ends and kept the shortest route between two food sources ([Wikipedia](https://en.wikipedia.org/wiki/Physarum_polycephalum)).
- Jenson's version runs 5–10 million particles in real time on a GTX 1070, in C++ and GLSL with openFrameworks ([Sage Jenson](https://cargocollective.com/sagejenson/physarum)).
- *Tapestry* was released as a single ERC-721 token on 12 April 2023, presented by Feral File ([Verse](https://verse.works/series/tapestry-by-sage-jenson)).

## Sources

- [Wikipedia: Physarum polycephalum](https://en.wikipedia.org/wiki/Physarum_polycephalum)
- [Sage Jenson, Physarum write-up](https://cargocollective.com/sagejenson/physarum)
- [Amanda Ghassaei, gpu-io Physarum demo](https://apps.amandaghassaei.com/gpu-io/examples/physarum/)
- [Amanda Ghassaei, gpu-io Physarum source](https://raw.githubusercontent.com/amandaghassaei/gpu-io/main/examples/physarum/index.js)
- [Verse: Tapestry by Sage Jenson](https://verse.works/series/tapestry-by-sage-jenson)
- [Jenson & Kuksenok, speculative sim](https://ksen0.github.io/speculative-sim/)
- [This Is Paper: Primordium](https://thisispaper.com/mag/primordium-sage-jenson)
- [TED: Heather Barnett](https://www.ted.com/talks/heather_barnett_what_humans_can_learn_from_semi_intelligent_slime)
- [Sebastian Lague, Coding Adventure (YouTube)](https://www.youtube.com/watch?v=X-iSQQgOd1A)
- [Sebastian Lague, Slime-Simulation (GitHub)](https://github.com/SebLague/Slime-Simulation)
- [Resolume ISF](https://resolume.com/support/en/isf)
- [ISF multipass spec](https://docs.isf.video/ref_multipass.html)
- [Epic: Grid2D Collection API](https://dev.epicgames.com/documentation/en-us/unreal-engine/python-api/class/NiagaraDataInterfaceGrid2DCollection)
- [Epic: fluid simulation overview](https://dev.epicgames.com/documentation/en-us/unreal-engine/fluid-simulation-in-unreal-engine---overview)
- [80.lv: Niagara render targets in UE5](https://80.lv/articles/niagara-system-integration-with-render-targets-in-unreal-engine-5)
- [Godot: compute shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/compute_shaders.html)
- [Godot: Texture2DRD](https://docs.godotengine.org/en/stable/classes/class_texture2drd.html)
