# Physarum Slime Mould Networks — research

## Origin
The living organism, *Physarum polycephalum*, is an acellular slime mould (a myxomycete), first described in 1822 and used as a model organism for motility and chemotaxis ([Wikipedia](https://en.wikipedia.org/wiki/Physarum_polycephalum)). Its network-building became famous through two experiments: a maze-solving study (Nakagaki, Yamada, Tóth, Nature, 2000) and a 2010 Science study in which oat flakes marked Tokyo and 36 surrounding towns and the mould grew a network resembling the real rail system ([Wikipedia](https://en.wikipedia.org/wiki/Physarum_polycephalum)).

The agent model that artists use is from Jeff Jones, "Characteristics of pattern formation and evolution in approximations of Physarum transport networks" (2010). Sage Jenson's write-up credits it as the basis of his model ([Sage Jenson](https://cargocollective.com/sagejenson/physarum)). Amanda Ghassaei's WebGL demo also names Jones 2010 and Jenson as its sources ([gpu-io demo](https://apps.amandaghassaei.com/gpu-io/examples/physarum/)). I could not open the Jones paper (the UWE repository returned 403), so the model details below come from Jenson's page.

## Key works and artists
- **Sage Jenson, Physarum model and *Tapestry* (2019).** Jenson adapted Jones's algorithm to real time. *Tapestry* is a graphic score that traces a path through the model's parameter space, first shown at *Dismantling*, a show with Kit Kuksenok ([Verse](https://verse.works/series/tapestry-by-sage-jenson)). Jenson's page reports 5–10 million particles in real time on a GTX 1070 ([Sage Jenson](https://cargocollective.com/sagejenson/physarum)).
- ***Dismantling* (Berlin, 2019).** A live performance and exhibition at Workshop on Forster that used the Physarum simulation, exploring a shifting locus of agency between observer and representation ([Jenson & Kuksenok](https://ksen0.github.io/speculative-sim/)).
- **Primordium (2022).** Jenson's first web-based solo show, curated by symbios.wiki. Its agents sense, navigate and modify their environment, and the patterns are emergent within a parameter space ([This Is Paper](https://thisispaper.com/mag/primordium-sage-jenson)).
- **Heather Barnett.** An artist who co-creates with living *Physarum polycephalum*, as a biology-based take on self-organising systems ([TED](https://www.ted.com/talks/heather_barnett_what_humans_can_learn_from_semi_intelligent_slime)).
- **Amanda Ghassaei, Physarum Transport Network.** A WebGL fragment-shader demo where touch or drag adds attractant, with presets and video recording ([demo](https://apps.amandaghassaei.com/gpu-io/examples/physarum/)).

## How it works
Two layers ([Sage Jenson](https://cargocollective.com/sagejenson/physarum)):
- Agents: each has a position and heading, and three sensors (front-left, front, front-right). They compare the trail value at the sensors and turn left, turn right or keep going.
- Trail map: a 2D grid. Agents deposit into it each tick.

Each tick is deposit, diffuse (a 3x3 mean filter), then decay (a multiplicative fade). Jenson's listed parameters are sensor distance, sensor size, sensor angle, step size, rotation angle, deposition amount, decay factor, deposit size and diffuse size.

Agents follow the trails they and others lay, so trails reinforce. Weak ones fade and strong ones become the vein-like network. Jones's original includes a collision step that allows at most one particle per cell. Jenson usually skips it because the patterns looked better, and it also lets the step run in parallel ([Sage Jenson](https://cargocollective.com/sagejenson/physarum)). The whole loop maps to GPU compute or fragment shaders.

## Visual language
- Glowing filaments that branch, merge and loop into reticulated, labyrinthine or spotted fields.
- Continuous slow motion: networks drift, reorganise and breathe, rather than cut.
- Colour comes from the trail intensity, so it is easy to map one hue ramp or palette on the trail value (this is a design suggestion, not a sourced claim).
- Jenson's page shows GIFs of network growth, a "contagion" spread from the centre and an "explosion" ([Sage Jenson](https://cargocollective.com/sagejenson/physarum)). I could not see the images, only the captions.

## Live potential
These are proposals built on the parameter list above, not sourced claims.
- **Kick:** a short burst of high deposit, or a scatter that re-randomises headings and positions of a fraction of agents. The network shatters, then regrows between kicks.
- **Bass:** sensor angle or rotation angle modulation. This changes the topology from fine veins to fat blobs or spirals.
- **Hi-hats:** a brief decay-factor dip or flash on the trail value, which makes a shimmer without changing structure.
- **BPM:** reset or spawn agents on a bar or phrase, so the network regrows in sync with the music.
- **MIDI faders:** sensor distance, sensor angle, step size, decay, diffuse. On a KeyStep or BeatStep, these map to knobs. Presets (like Ghassaei's) can map to pads.
- **Layering:** output the trail map as Spout or NDI to Resolume. Use it as a luma matte, a displacement map or an additive overlay on video or on Unreal scenes.
- **Boring on a big screen:** a single settled network that stops changing, low contrast, a single hue and a low agent count. Fix by keeping steady perturbation (attractor points, a reseed, parameter drift).

## Variations
- **Attractor from audio:** spectrum bins drop attractant sources at fixed screen positions, so the network grows toward the bass and treble spots (Ghassaei's demo already lets users add attractant by touch).
- **Contagion mode:** infected agents lay a second trail channel that spreads. On the beat, infect a few agents and colour them differently ([Sage Jenson](https://cargocollective.com/sagejenson/physarum)).
- **Text or logo seed:** deposit a 1000a mask into the trail map so the network grows around or along the letters.
- **Multi-species:** two or three agent populations with separate trail maps, attracted to their own trail and repelled by the other's. Each gets one hue (design proposal).
- **Network as geometry:** read the trail map into an Unreal Niagara system or a height field, so the filaments become 3D terrain or lights.

## Facts
- The Tokyo experiment used oat flakes for Tokyo and 36 surrounding towns, and the mould's network had efficiency and fault tolerance comparable to the real rail system ([Wikipedia](https://en.wikipedia.org/wiki/Physarum_polycephalum)).
- In the maze experiment (2000), the mould withdrew from dead ends and kept the shortest route between two food sources ([Wikipedia](https://en.wikipedia.org/wiki/Physarum_polycephalum)).
- Jenson's version typically runs 5–10 million particles in real time on a GTX 1070 (a 2016-era GPU), with C++/GLSL in openFrameworks ([Sage Jenson](https://cargocollective.com/sagejenson/physarum)).
- Jenson usually drops Jones's one-particle-per-cell collision step because the patterns look better ([Sage Jenson](https://cargocollective.com/sagejenson/physarum)).
- *Tapestry* was released as a single ERC-721 token on 12 April 2023, presented by Feral File ([Verse](https://verse.works/series/tapestry-by-sage-jenson)).
- Jenson's work was used in a live performance in which agency shifts between observer and representation ([Jenson & Kuksenok](https://ksen0.github.io/speculative-sim/)).

## Visual references
- [Sebastian Lague, Coding Adventure: Ant and Slime Simulations (YouTube, channel: Sebastian Lague)](https://www.youtube.com/watch?v=X-iSQQgOd1A) — the slime part shows agents forming trails and veins in a GPU sim.
- [Heather Barnett, What humans can learn from semi-intelligent slime (YouTube, channel: TED)](https://www.youtube.com/watch?v=2UxGrde1NDA) — 2014 TED talk with the living organism growing networks. Title and channel confirmed via oEmbed.
- [Amanda Ghassaei, Physarum Transport Network](https://apps.amandaghassaei.com/gpu-io/examples/physarum/) — live WebGL demo with presets (page opened; the visuals themselves are only seen when running).
- [Sage Jenson, Physarum](https://cargocollective.com/sagejenson/physarum) — write-up with network, contagion and explosion GIFs (page opened; images not visible to my tool).
- [Sage Jenson, Tapestry on Verse](https://verse.works/series/tapestry-by-sage-jenson) — artwork page for the Physarum phase-space score.
- [Jenson & Kuksenok, speculative sim](https://ksen0.github.io/speculative-sim/) — project page for *Dismantling*.

## Sources
- [Wikipedia: Physarum polycephalum](https://en.wikipedia.org/wiki/Physarum_polycephalum)
- [Sage Jenson, Physarum write-up](https://cargocollective.com/sagejenson/physarum)
- [Amanda Ghassaei, gpu-io Physarum demo](https://apps.amandaghassaei.com/gpu-io/examples/physarum/)
- [Verse: Tapestry by Sage Jenson](https://verse.works/series/tapestry-by-sage-jenson)
- [Jenson & Kuksenok, speculative sim](https://ksen0.github.io/speculative-sim/)
- [This Is Paper: Primordium](https://thisispaper.com/mag/primordium-sage-jenson)
- [TED: Heather Barnett](https://www.ted.com/talks/heather_barnett_what_humans_can_learn_from_semi_intelligent_slime)
