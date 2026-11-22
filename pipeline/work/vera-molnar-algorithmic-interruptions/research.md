# Vera Molnar Algorithmic Interruptions — research

## Origin
Vera Molnár was born in Budapest on 5 January 1924, moved to Paris in the late 1940s and died there on 7 December 2023. In 1960 she co-founded the Groupe de Recherche d'Art Visuel (GRAV) ([Wikipedia](https://en.wikipedia.org/wiki/Vera_Moln%C3%A1r)). From the late 1950s and through the 1960s she worked by hand on grid paper with self-set rules for colour, shape, line and direction. She called this her "machine imaginaire" (imaginary machine) ([V&A](https://www.vam.ac.uk/blog/museum-life/vera-molnar-machine-imaginaire-the-dance-of-hands-and-machine-thinking), [Artnet](https://news.artnet.com/art-world/in-pictures-vera-molnar-pompidou-2446604)). In 1968 she got access to a computer lab (an IBM mainframe at the Sorbonne), taught herself Fortran and made plotter drawings, which the Pompidou describes as the first digital drawings with a plotter-linked computer by an artist in France ([Morgan](https://www.themorgan.org/morganmobile/order-and-disorder/molnar), [Toledo Museum](https://infiniteimages.toledomuseum.org/?p=811), [Centre Pompidou](https://www.centrepompidou.fr/en/program/calendar/event/PA7jRZ5)). In the 1970s she began adding a measured amount of disorder to simple geometric compositions ([Centre Pompidou](https://www.centrepompidou.fr/en/program/calendar/event/PA7jRZ5)).

## Key works and artists
- *Interruptions*, 1968: black ink on Benson plotter paper, a 25 x 25 grid of lines, each randomly rotated, with random interruptions that erase sections and leave voids ([Toledo Museum](https://infiniteimages.toledomuseum.org/?p=811), [Morgan](https://www.themorgan.org/morganmobile/order-and-disorder/molnar)).
- *Interruptions*, 1969: plotter drawing in the National Gallery of Art, Washington (gift of the artist, 2018.38.1) ([NGA](https://www.nga.gov/artworks/216160-interruptions)). The V&A also holds a 1969 *Interruptions*, plus *(Des)Ordres* (1974) and *Structure of Squares* (1974) ([V&A](https://www.vam.ac.uk/blog/museum-life/vera-molnar-machine-imaginaire-the-dance-of-hands-and-machine-thinking)).
- *Interruptions à recouvrements* (Disturbances through overlappings), 1969: a layered series that reuses the first drawing's computed logic with varied scale, shape and line thickness. *Structure de quadrilatères*, 1987, is a related square series ([CMU IDeATe](https://courses.ideate.cmu.edu/15-104/f2018/2018/10/05/vera-molnars-generative-art)).
- *Speak to the Eye* (Parler à l'oeil), Centre Pompidou, 28 Feb to 26 Aug 2024: retrospective including *Four Randomly Distributed Elements* (1959), *A Stroll Between Order and Chaos* (1975) and *160 Squares Pushed to the Limit* (1976) ([Centre Pompidou](https://www.centrepompidou.fr/en/program/calendar/event/PA7jRZ5), [Artnet](https://news.artnet.com/art-world/in-pictures-vera-molnar-pompidou-2446604)).

## How it works
Start with a strict, repetitive structure, such as a grid of squares or lines. For each cell, a small algorithm decides position, rotation, size or whether the element is drawn at all. Randomness is fed into a logical rule, not left to chance alone ([CMU IDeATe](https://courses.ideate.cmu.edu/15-104/f2018/2018/10/05/vera-molnars-generative-art)). In *Interruptions*, the two disturbances are random rotation of each line and random erasure of segments. She changed one parameter at a time and treated the result as a dialogue with the machine that yields surprise ([Toledo Museum](https://infiniteimages.toledomuseum.org/?p=811)). Her stated aim was a "disturbed equilibrium": an abstract composition works when it mixes order and disorder ([Morgan](https://www.themorgan.org/morganmobile/order-and-disorder/molnar)). The random element is meant as an "aesthetic shock" against symmetrical order ([CMU IDeATe](https://courses.ideate.cmu.edu/15-104/f2018/2018/10/05/vera-molnars-generative-art)).

## Visual language
Thin black lines or squares on a plain ground, in a dense even grid. Most cells obey the rule, a few break it: a tilted line, a hole, an overlapping square. The tension comes from the contrast between the rigid grid and the few deviations, which the eye hunts for. Works are often serial: many variations of one rule shown side by side. Monochrome plotter look (ink on pale paper, or white ink on salmon paper in *Structure de quadrilatères*) ([CMU IDeATe](https://courses.ideate.cmu.edu/15-104/f2018/2018/10/05/vera-molnars-generative-art)).

## Live potential
(Design suggestions, not claims from sources.)
- Kick: raise the disorder amount briefly, or re-seed the random pattern on each hit so the grid "snaps" into a new broken state, then relaxes back to order.
- Bass: drives rotation amplitude or displacement of all cells; sustained bass loosens the grid.
- Hi-hats: toggle single cells or lines on and off (the interruptions), a fast sparse flicker.
- BPM: step the seed per bar, or tween between two seeds over a beat; breathing grid density on phrases.
- MIDI faders: disorder amount, grid density (e.g. 8 to 64), rule select (rotate / erase / displace / overlap), line thickness, colour pair.
- Layering: it is line art on a flat ground, so it keys cleanly over footage, works as a mask or displacement map, and can be fed as Spout/NDI input into Resolume or Unreal as a stencil.
- Boring risk: one fixed rule at constant disorder looks like a screensaver. Pure 100% randomness loses the Molnar tension. Keep the contrast: long stretches of order, then ruptures.

## Variations
- Disorder as a travelling wave: a band of chaos sweeps across the strict grid on the beat, leaving order behind it.
- Healing grid: cells that were broken return to their rule over a few bars, so order and disorder breathe with the track.
- Rule chain on a fader: crossfade between rule sets (rotate, erase, overlap, size) like clips.
- Colour plotter pens: swap black ink for a pen-plotter palette, with new "pen" per hi-hat or per bar.
- Footage-driven disorder: use the luminance of a live camera/clip as the per-cell disorder map, so the image shows through where the grid breaks.

## Facts
- *Interruptions* (1968) begins from a 25 x 25 grid of randomly rotated lines, with random interruptions erasing parts ([Toledo Museum](https://infiniteimages.toledomuseum.org/?p=811)).
- She had worked out her algorithmic process by hand for years before computer access ([Toledo Museum](https://infiniteimages.toledomuseum.org/?p=811), [V&A](https://www.vam.ac.uk/blog/museum-life/vera-molnar-machine-imaginaire-the-dance-of-hands-and-machine-thinking)).
- She learned both Fortran and BASIC and ran work via punch cards and a plotter ([Artnet](https://news.artnet.com/art-world/in-pictures-vera-molnar-pompidou-2446604), [Wikipedia](https://en.wikipedia.org/wiki/Vera_Moln%C3%A1r)).
- Her 22-volume "Diary" holds diagrams, photos and documents recording her works ([Centre Pompidou](https://www.centrepompidou.fr/en/program/calendar/event/PA7jRZ5)).
- Awards include the DAM Digital Arts Award (2005) and the AWARE award (2018) ([Wikipedia](https://en.wikipedia.org/wiki/Vera_Moln%C3%A1r)).
- The Morgan's 1968 *Interruptions* measures 32 x 32 cm ([Morgan](https://www.themorgan.org/morganmobile/order-and-disorder/molnar)).

## Visual references
- [Interruptions, 1968, Toledo Museum](https://infiniteimages.toledomuseum.org/?p=811) — museum page with the plotter drawing and method text
- [Interruptions, 1968, The Morgan](https://www.themorgan.org/morganmobile/order-and-disorder/molnar) — page on the 32 x 32 cm plotter drawing
- [Interruptions, 1969, National Gallery of Art](https://www.nga.gov/artworks/216160-interruptions) — collection record of the 1969 plotter drawing
- [Machine Imaginaire, V&A blog](https://www.vam.ac.uk/blog/museum-life/vera-molnar-machine-imaginaire-the-dance-of-hands-and-machine-thinking) — hand and plotter works including Interruptions and (Des)Ordres
- [Speak to the Eye in pictures, Artnet](https://news.artnet.com/art-world/in-pictures-vera-molnar-pompidou-2446604) — photos of the 2024 Pompidou retrospective works

## Lead image
NONE. Wikimedia Commons file pages (e.g. a 1969 *Interruptions à recouvrements* file and a 1996 portrait) could not be opened from this environment, so licence and image URL could not be verified. No similar Unsplash/Pexels photo was checked.

## Quotation images
none

## Sources
- [V&A blog](https://www.vam.ac.uk/blog/museum-life/vera-molnar-machine-imaginaire-the-dance-of-hands-and-machine-thinking)
- [Toledo Museum, Interruptions](https://infiniteimages.toledomuseum.org/?p=811)
- [The Morgan, Interruptions](https://www.themorgan.org/morganmobile/order-and-disorder/molnar)
- [NGA, Interruptions](https://www.nga.gov/artworks/216160-interruptions)
- [CMU IDeATe, Vera Molnar's Generative Art](https://courses.ideate.cmu.edu/15-104/f2018/2018/10/05/vera-molnars-generative-art)
- [Centre Pompidou, Speak to the Eye](https://www.centrepompidou.fr/en/program/calendar/event/PA7jRZ5)
- [Artnet, in pictures](https://news.artnet.com/art-world/in-pictures-vera-molnar-pompidou-2446604)
- [Wikipedia, Vera Molnár](https://en.wikipedia.org/wiki/Vera_Moln%C3%A1r)
