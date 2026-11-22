# Lenia Continuous Artificial Life — research

## Origin
Lenia is a family of cellular automata made by Bert Wang-Chak Chan, an independent researcher in Hong Kong. It is meant as a continuous generalisation of Conway's Game of Life: continuous space, time and state, with Life as a special case of discrete Lenia ([Wikipedia](https://en.wikipedia.org/wiki/Lenia)). The main paper, "Lenia: Biology of Artificial Life", reports more than 400 species in 18 families, many found by interactive evolutionary search ([arXiv 1812.05433](https://arxiv.org/abs/1812.05433)). A MATLAB v2.0 version dates from 2017 on the author's timeline ([Lenia portal](https://chakazul.github.io/lenia.html)). Recognition: winner of the 2018 Virtual Creatures Contest at GECCO (Kyoto), honorable mention in the ALIFE Art Award 2018 (Tokyo), and ISAL Outstanding Publication of 2019 ([Wikipedia](https://en.wikipedia.org/wiki/Lenia)). A follow-up, "Lenia and Expanded Universe", adds 3D and higher dimensions, multiple kernels and multiple channels ([arXiv 2005.03742](https://arxiv.org/abs/2005.03742)).

## Key works and artists
- **Bert Chan, Lenia (2018 onward)**: the original system, with a browser version, Python, MATLAB and R/Jupyter code, MIT licensed ([GitHub](https://github.com/Chakazul/Lenia)). Named species include Orbium (disc-shaped glider), Scutium (shield-shaped, omnidirectional mover), Paraptera, Gyrorbium and Hexadentium ([arXiv 1812.05433](https://arxiv.org/abs/1812.05433)).
- **Lenia: Expanded Universe (2020)**: multi-channel and multi-kernel worlds with self-replication, emission, growth by ingestion and "virtual eukaryotes" with cell-type differentiation ([arXiv 2005.03742](https://arxiv.org/abs/2005.03742)).
- **Flow-Lenia (2022/23)**: Plantec, Hamon, Etcheverry, Oudeyer, Moulin-Frier and Chan add mass conservation and localised parameters, so many different creatures can coexist and mix in one world ([arXiv 2212.07906](https://arxiv.org/abs/2212.07906)).
- Chan's portal lists art uses: a VRChat "Lenia Room", the film "Thoughts on Life", a virtual creature module for Artbreeder, and Interspecifics' "Codex Virtualis" shown at Ars Electronica ([Lenia portal](https://chakazul.github.io/lenia.html)). The Interspecifics project pages themselves do not name Lenia, so treat the link as claimed by Chan only.

## How it works
Every step has three stages ([arXiv 1812.05433](https://ar5iv.labs.arxiv.org/html/1812.05433)):
1. **Potential**: each pixel takes a weighted sum of its neighbours. The weights come from a ring-shaped kernel (a smooth bump copied into concentric rings with chosen heights).
2. **Growth**: a bell-shaped growth function with centre mu and width sigma maps the potential to -1..+1. Near mu the cell grows, elsewhere it decays.
3. **Update**: add growth times a small time step dt to the state, then clip to 0..1.

Kernel radius R, mu, sigma and dt are the creature's "genes". Species occupy continuous regions of this parameter space and morph into each other smoothly ([arXiv 1812.05433](https://arxiv.org/abs/1812.05433)). The convolution is the heavy part, which suits a GPU shader or FFT; Chan's Python version supports GPU ([GitHub](https://github.com/Chakazul/Lenia)).

## Visual language
Soft, translucent blobs with fuzzy edges and ring-like internal structure, like microscope footage of plankton or amoebae. Creatures glide, rotate, pulse, split and bounce off each other (the paper calls them geometric, metameric, fuzzy, resilient and adaptive, [Wikipedia](https://en.wikipedia.org/wiki/Lenia)). The state is a single grey channel, so colour is entirely a choice: palettes, gradient maps, or channel offsets by age or velocity. Rhythm comes from slow, breathing, organic motion rather than hard beats.

## Live potential
- **Kick**: inject a blob of cells (a stamp of a known species) at a position, or a short dt boost so creatures lurch forward.
- **Bass / sustained energy**: modulate growth centre mu or width sigma; small shifts make creatures swell, dissolve or morph into neighbouring species (the parameter-space structure from the paper above).
- **Hi-hats**: sprinkle small noise seeds, or flicker the colour palette.
- **BPM**: tie dt or a periodic "reseed" to the clock; trigger species swaps per bar.
- **MIDI faders (KeyStep/BeatStep)**: kernel radius, mu, sigma, dt, palette, injection brush size. Pads can load named species.
- **Layering**: output as a grey mask or a gradient-mapped colour layer through Spout or NDI into Resolume; use as displacement or luma matte over footage; feed back into Unreal as a Niagara texture.
- **Boring risks**: a stable creature that simply glides; wrong parameters make everything die or fill the screen. Needs ongoing injection, parameter drift and safeguards (auto reseed when total mass drops near zero).

## Variations
- **Beat-seeded ecology**: each drum voice stamps a different species, so kick, snare and hats compete as visible populations.
- **Footage as food**: feed a camera or Resolume clip luma into the field as a source/inhibitor, so creatures swarm around bright shapes.
- **Lenia as displacement**: use the field as a normal/height map to refract a second clip.
- **Multi-channel colour**: use a 3-channel Lenia (Expanded Universe) so RGB are interacting species ([arXiv 2005.03742](https://arxiv.org/abs/2005.03742)).
- **Flow-Lenia mixing**: coexisting species with mass conservation, so beats move matter instead of creating it ([arXiv 2212.07906](https://arxiv.org/abs/2212.07906)).

## Facts
- Conway's Game of Life is a special case of discrete Lenia, with a square kernel and a rectangular growth rule ([Wikipedia](https://en.wikipedia.org/wiki/Lenia)).
- The multi-channel architecture resembles a recurrent residual convolutional network, linking Lenia to neural cellular automata ([arXiv 2005.03742](https://arxiv.org/abs/2005.03742)).
- In 3D Lenia, stable creatures are often spherical and sometimes arrange internal voids like Platonic solids ([arXiv 2005.03742](https://arxiv.org/abs/2005.03742)).
- Lenia creatures keep their boundaries on collision in some cases ("individuality") and some self-replicate ([arXiv 2005.03742](https://arxiv.org/abs/2005.03742)).
- Chan's code is MIT licensed, and the browser version lets you pick species, add random cells and export GIFs ([GitHub](https://github.com/Chakazul/Lenia), [web demo](https://chakazul.github.io/Lenia/JavaScript/Lenia.html)).
- In Flow-Lenia, update-rule parameters can be localised in the world itself, so different "species" can sit side by side ([arXiv 2212.07906](https://arxiv.org/abs/2212.07906)).

## Visual references
- [Lenia web demo](https://chakazul.github.io/Lenia/JavaScript/Lenia.html) — interactive browser simulation with species list (Orbium, Scutium), brush and GIF export.
- [Lenia - Mathematical Life Forms (YouTube, channel Bert Chan (Azul))](https://youtu.be/iE46jKYcI4Y) — author's video of Lenia creatures in motion.
- [Lenia: Expanded Universe 1080p (YouTube, channel Bert Chan (Azul))](https://youtu.be/HT49wpyux-k) — companion video for multi-channel, 3D and self-replication results.
- [Lenia portal](https://chakazul.github.io/lenia.html) — hub with demos, videos, Shadertoy and VRChat links.
- [Lenia GitHub repo](https://github.com/Chakazul/Lenia) — code and screencap folder.

## Lead image
NONE. No free-licensed Lenia image (Commons file page or similar Unsplash/Pexels photo) could be opened and confirmed; the Commons category was not reachable.

## Quotation images
none

## Sources
- [arXiv 1812.05433, Lenia: Biology of Artificial Life](https://arxiv.org/abs/1812.05433)
- [ar5iv version](https://ar5iv.labs.arxiv.org/html/1812.05433)
- [arXiv 2005.03742, Lenia and Expanded Universe](https://arxiv.org/abs/2005.03742)
- [arXiv 2212.07906, Flow-Lenia](https://arxiv.org/abs/2212.07906)
- [Wikipedia: Lenia](https://en.wikipedia.org/wiki/Lenia)
- [Lenia portal](https://chakazul.github.io/lenia.html)
- [GitHub Chakazul/Lenia](https://github.com/Chakazul/Lenia)
- [Lenia web demo](https://chakazul.github.io/Lenia/JavaScript/Lenia.html)
- [ALIFE encyclopedia: Lenia](https://alife.org/encyclopedia/software-platforms/lenia/)
- [Interspecifics Codex Virtualis](https://interspecifics.cc/work/category/bacteria/)
