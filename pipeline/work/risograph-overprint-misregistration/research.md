# Risograph Overprint and Misregistration — research

## Origin
- The Risograph is a brand of digital stencil duplicator made by Riso Kagaku Corporation and introduced in Japan in 1980 ([Wikipedia: Risograph](https://en.wikipedia.org/wiki/Risograph)). The company goes back to Noboru Hayama, who started mixing inks at his kitchen sink after WWII and founded a mimeograph printing company in 1946 ([Wikipedia: Riso Kagaku](https://en.wikipedia.org/wiki/Riso_Kagaku_Corporation)).
- Its lineage is the mimeograph plus the home stencil gadget Print Gocco (1977, over ten million sold, retired 2008). In Japan the machine is mostly an office and school printer, about 90 per cent of use; the artist scene is a later resurgence ([It's Nice That](https://www.itsnicethat.com/articles/the-view-from-tokyo-risograph-in-japan-print-graphic-design-261124)).
- It became popular with artists in the 2010s for zines and comics ([Wikipedia](https://en.wikipedia.org/wiki/Risograph)). The animation branch is newer: print every frame, scan, re-assemble (Julia Schimautz, Hiromu Oka, 2021 onward).

## Key works and artists
- **Hiromu Oka, "Hodo Station" opening titles (TV Asahi, 2021).** Hundreds of Riso prints; motion designed in After Effects, one file per colour exported and printed in layers, then scanned and animated. He also wrote custom shaders to imitate the Riso texture on screen. Printing by Hand Saw Press. Over 1,000 A3 sheets for about 15 seconds on six paper types ([It's Nice That, process](https://www.itsnicethat.com/articles/hiromu-oka-hodo-station-process-animation-051222), [It's Nice That, intro](https://www.itsnicethat.com/articles/hiromu-oka-animation-130422), [Vimeo, Oka](https://vimeo.com/639447319)).
- **Julia Schimautz, "Every Thought You've Ever Had" (Pan Well, 2021).** 1,102 frames printed on 12 A3 sheets in yellow, orange, bright red, teal and black at Dream Press, scanned and composed in After Effects ([People of Print](https://peopleofprint.com/animation/julia-schimautz-every-thought-youve-ever-had-music-video)).
- **Julia Schimautz and Francis Broek (DTAN, Berlin), animated Riso pieces and "In Bloom" (with New Eyes, 2021).** Colour and movement explored in After Effects, then printed; mis-registrations make the result partly unpredictable ([It's Nice That](https://www.itsnicethat.com/articles/julia-schimautz-project-animation-021122)).
- **A 70-frame Schimautz loop** printed in three inks, yellow, fluorescent pink and blue ([Boing Boing](https://boingboing.net/2023/07/09/animation-plays-with-the-explosively-bright-colors-of-a-risograph-machine.html)).
- Other riso animators listed: Rapapawn, Suejee Lee, Magdalene Kennedy's own method ([Kennedy](https://magdalenekennedy.substack.com/p/risograph-animation)).

## How it works
- The machine burns tiny holes in a master with a thermal head, wraps it round a drum, and pushes soy-based ink through the holes onto uncoated paper. No heat fixing ([Wikipedia](https://en.wikipedia.org/wiki/Risograph)).
- One drum equals one ink colour, so a multicolour print needs one pass per colour. Passes never line up perfectly, which gives misregistration; white gaps can show between colours, and designers use trapping (slight overlaps) to hide this ([College for Creative Studies](https://campus.collegeforcreativestudies.edu/imaging-center/?p=3493)).
- Inks are translucent, so layers mix where they overlap, and layering gives more vivid colour than many digital print methods. Registration is "rarely perfect" ([Kennedy](https://magdalenekennedy.substack.com/p/risograph-animation)).
- Screen recipe: separate footage into 2 to 3 channels, threshold or halftone each, tint with a spot ink, multiply blend on a paper colour. A generative web example does the same: blue and fluoro pink stripes, pink halftone dots, yellow disc turning green or orange on overlap, multiply to mimic translucent ink ([UX Pilot Riso Club](https://uxpilot.ai/labs/creative-direction-playground/riso-club)).

## Visual language
- Fluorescent pink, blue, yellow, teal, orange on off-white paper; overlaps produce new colours (purple, green, orange).
- Grainy gradients and halftone dots instead of smooth tone; speckle and uneven ink density.
- Plates slightly offset, so edges get coloured fringes and thin slivers of paper show.
- Frame-to-frame jitter: each hand-printed frame sits slightly differently, so the image breathes ([It's Nice That](https://www.itsnicethat.com/articles/julia-schimautz-project-animation-021122)). Oka calls the unevenness of paper and ink a form of "information unevenness" ([Eye Magazine](https://eyemagazine.com/feature/article/jump-cuts-hiromu-oka)).

## Live potential
- **Kick**: push plate offsets apart for a frame then settle back; or flip ink colours.
- **Bass**: grain and halftone dot size or density; ink opacity.
- **Hi-hats**: tiny per-frame jitter of each plate (random sub-pixel shake), 8 to 12 fps stepped time to mimic printed frames.
- **BPM**: tempo-locked plate rotation or drift; swap ink palettes per phrase.
- **MIDI faders**: offset amount, grain, threshold per channel, paper colour, palette select, number of plates (1 to 3).
- **Layering**: works as a final grade over Unreal or Resolume content; also use as a per-layer effect so only some sources get the print treatment. Lay a paper texture under it with multiply.
- **Boring risks**: static offset looks like a cheap chromatic aberration; constant 3-colour palette becomes a preset. Change thresholds and palette, let offsets vary per region, and cut between full-print and clean frames.

## Variations
- **Plates by audio band**: kick drives plate 1, mid plate 2, hi-hats plate 3, so each ink moves with its own instrument.
- **Stepped print animation**: freeze 6 to 12 fps with random registration per frame, imitating the scanned-frame technique.
- **Drum run-out**: ink density fades like a drum running dry, then refills on a drop.
- **Paper stock switch**: swap between newsprint, kraft and white on fader, changing multiply behaviour.
- **Real scan loop**: print a few real frames, scan, and use as an overlay texture (grain, fibre, ink pooling), as Oka does with the physical process.

## Facts
- The name is not from "lithograph"; it plays on a Japanese word for "ideal" ([It's Nice That](https://www.itsnicethat.com/articles/the-view-from-tokyo-risograph-in-japan-print-graphic-design-261124)).
- A good Riso lasts about 100,000 masters and 5,000,000 copies; it is cheaper per page than copiers above about 100 copies ([Wikipedia](https://en.wikipedia.org/wiki/Risograph)).
- Oka needed more than 1,000 A3 sheets for about 15 seconds of final animation ([It's Nice That](https://www.itsnicethat.com/articles/hiromu-oka-hodo-station-process-animation-051222)).
- Oka first used Risograph in 2019 and was reminded of the Print Gocco his parents used ([It's Nice That](https://www.itsnicethat.com/articles/hiromu-oka-animation-130422)).
- Schimautz's music video used 1,102 frames on only 12 A3 sheets ([People of Print](https://peopleofprint.com/animation/julia-schimautz-every-thought-youve-ever-had-music-video)).
- Registration "will never be perfect", so misregistration is inherent to the medium, not a defect to add ([CCS](https://campus.collegeforcreativestudies.edu/imaging-center/?p=3493)).

## Visual references
- [Pan Well, Every Thought You've Ever Had (official video)](https://www.youtube.com/watch?v=VdAVN8zzV38) — music video by Julia Schimautz built from Riso-printed frames; channel Pan Well.
- [Hodo Station 2021 opening, Hiromu Oka](https://vimeo.com/639447319) — news-show titles, every cut printed and scanned on a Riso; uploaded by Hiromu Oka.
- [It's Nice That: Schimautz animations](https://www.itsnicethat.com/articles/julia-schimautz-project-animation-021122) — stills and GIFs of grainy, offset, saturated Riso animation.
- [It's Nice That: Oka, Hodo Station](https://www.itsnicethat.com/articles/hiromu-oka-hodo-station-process-animation-051222) — stills and process of colour-separated layers.
- [Boing Boing: 70-frame riso animation](https://boingboing.net/2023/07/09/animation-plays-with-the-explosively-bright-colors-of-a-risograph-machine.html) — pink, blue, yellow animation.
- [UX Pilot Riso Club](https://uxpilot.ai/labs/creative-direction-playground/riso-club) — a generative web version with offset stripes, halftone dots and multiply.

## Lead image
NONE. Wikimedia Commons file pages and the Unsplash search could not be opened or confirmed to show a riso overprint look, so no free image was verified.

## Quotation images
none

## Sources
- https://en.wikipedia.org/wiki/Risograph
- https://en.wikipedia.org/wiki/Riso_Kagaku_Corporation
- https://www.itsnicethat.com/articles/the-view-from-tokyo-risograph-in-japan-print-graphic-design-261124
- https://www.itsnicethat.com/articles/hiromu-oka-hodo-station-process-animation-051222
- https://www.itsnicethat.com/articles/hiromu-oka-animation-130422
- https://www.itsnicethat.com/articles/julia-schimautz-project-animation-021122
- https://peopleofprint.com/animation/julia-schimautz-every-thought-youve-ever-had-music-video
- https://boingboing.net/2023/07/09/animation-plays-with-the-explosively-bright-colors-of-a-risograph-machine.html
- https://campus.collegeforcreativestudies.edu/imaging-center/?p=3493
- https://magdalenekennedy.substack.com/p/risograph-animation
- https://eyemagazine.com/feature/article/jump-cuts-hiromu-oka
- https://uxpilot.ai/labs/creative-direction-playground/riso-club
- https://vimeo.com/639447319
- https://www.youtube.com/watch?v=VdAVN8zzV38
