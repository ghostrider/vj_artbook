# Tritone Red, White and Black — research

## Origin
**Is "tritone" a word?** Yes, twice over, and the two meanings are unrelated.
- **Print:** a duotone is a halftone made by layering one contrasting halftone colour over another, traditionally black plus a second ink such as blue, yellow, brown or red ([Wikipedia: Duotone](https://en.wikipedia.org/wiki/Duotone)). Tritones (three inks) and quadtones (four) are the same idea with more plates. In Photoshop you start from a grayscale image and choose Image > Mode > Duotone, and the same dialog is used for tritones and quadtones ([Real World Print Production, O'Reilly](https://www.oreilly.com/library/view/real-world-print/9780321637598/9780321637598_ch09lev2sec13.html)). Note that in print a tritone is usually black plus two colours, where each ink gets its own tone curve. The 1000a look is a related but harder thing: only three flat output colours, no continuous tones.
- **Music:** a tritone is an interval of three whole tones, six semitones, which splits the octave exactly in half (600 cents) in 12-tone equal temperament; the "devil in music" (*diabolus in musica*) nickname is documented from the early 18th century, while tales of singers being punished for it are likely fanciful ([Wikipedia: Tritone](https://en.wikipedia.org/wiki/Tritone)). Fun naming overlap for a book about sound and picture, but do not mix them up in the chapter.

**The look itself** has no single origin. Red, white and black is the core palette of Russian Constructivism and Suprematist-influenced poster design (El Lissitzky's 1919 poster *Beat the Whites with the Red Wedge*, a red wedge cutting into a white circle, a lithographic Bolshevik propaganda poster ([Wikipedia](https://en.wikipedia.org/wiki/Beat_the_Whites_with_the_Red_Wedge))), and was passed on through record sleeves and conceptual art (see below). The technical side (threshold, posterise, gradient map, LUT) is standard image processing.

## Key works and artists
- **El Lissitzky, *Beat the Whites with the Red Wedge*, 1919.** The reference image for the whole idea: three flat colour areas and one diagonal gesture. [Wikipedia](https://en.wikipedia.org/wiki/Beat_the_Whites_with_the_Red_Wedge).
- **Kraftwerk, *The Man-Machine* (1978).** The V&A describes the cover as a limited palette of red, black and white (band in red shirts and black ties on a staircase, angled text; the back has black-outlined geometric shapes on white), inspired by El Lissitzky ([V&A](https://collections.vam.ac.uk/item/O1192920/the-man-machine-record-cover/)). Karl Klefisch did the artwork; the back cover adapts a graphic from Lissitzky's children's book *About Two Squares* ([Wikipedia](https://en.wikipedia.org/wiki/The_Man-Machine)).
- **Lissitzky echoes on record covers:** Ryuichi Sakamoto *B-2 Unit* (1980, red and black squares on the labels), The Wake *Something Outside* (1983, the Red Wedge poster with the slogans removed), Franz Ferdinand *This Fffire* (2004, a variation on the Red Wedge) ([John Coulthart](https://www.johncoulthart.com/feuilleton/2014/08/18/el-lissitzky-record-covers)). Franz Ferdinand's 2003 identity by Matthew Cooper freely references Rodchenko and Lissitzky ([Fonts In Use](https://www.fontsinuse.com/uses/17163/franz-ferdinand-franz-ferdinand-andamp-single)).
- **Barbara Kruger.** Her signature is black-and-white found photographs with white-on-red captions in Futura Bold Oblique or Helvetica Ultra Condensed ([Wikipedia](https://en.wikipedia.org/wiki/Barbara_Kruger)). The most direct art-world example of red/white/black plus a photograph that is crushed to black and white. Good model for a "text slams in on the beat" layer.
- **Theodore Ushev, *Tower Bawher* (2005).** A 3-minute abstract animated short, a fast tour of Constructivist art with references to Vertov, the Stenberg brothers, Rodchenko, Lissitzky and Popova ([NFB](https://www.nfb.ca/film/tower_bawher/), [Wikipedia](https://en.wikipedia.org/wiki/Tower_Bawher)). Motion reference for how Constructivist geometry moves.
- **Sin City (2005).** Mostly black and white with colour kept or added to selected objects, shot on Sony CineAlta F950 ([Wikipedia](https://en.wikipedia.org/wiki/Sin_City_(film))). It is the "spot colour on a crushed monochrome image" cousin, not strictly three colours.

No verifiable red/white/black VJ or AV set was found in this pass; the owner's original set is the best live reference.

## How it works
1. **Reduce to luminance** (greyscale).
2. **Crush contrast**: steep curve, levels or a threshold so the histogram piles up at the ends.
3. **Quantise to three levels** (below threshold A = black, between A and B = red, above B = white), or equivalently a **gradient map** with three hard stops. In Photoshop terms a gradient map maps the grey range onto the stops of a gradient; with hard stops there are no in-betweens.
4. **Or use a 3-colour LUT**: a LUT is a lookup table that remaps every input colour. Resolume has a LUT effect you can put on a clip, layer, group or composition; it accepts only `.cube` files, has an opacity slider and a blend mode ([Resolume LUT manual](https://resolume.com/support/de/lut)). A `.cube` file can be generated by a few lines of script (e.g. a 1D or 3D table that maps luma to black/red/white). Resolume Wire can also build and glitch LUTs ([CDM, 13 Feb 2025](https://cdm.link/tag/lut)).

Why hard steps: the three flat areas mean the picture is read as shape, not texture. Edges of the thresholds become the drawing.

## Visual language
- Flat poster fields, no gradients: reads as screen print, stencil, propaganda poster.
- Silhouette vs. mid-tone: the red band is the "middle grey" and carries the form; white is light, black is shadow (or the reverse).
- Diagonals, wedges, circles, big type: the Constructivist vocabulary matches the palette.
- Strong rhythm because edges flicker as the threshold crosses noise in the source (film grain, video noise become strobing contour lines).
- Red can be a pure primary or shift toward vermilion, crimson or hot pink; the hue is the only "mood" control.

## Live potential
- **Faders:** threshold A, threshold B (gap between them = amount of red), contrast/gamma before the quantiser, red hue/brightness, swap order (which tone becomes red: shadows, mids, highlights), invert.
- **Audio:** kick drives the threshold up/down, so the image snaps from poster to silhouette; hi-hats nudge the second threshold or flip red/white; BPM-synced step between "black, red, white" assignments on a 3-step sequence (BeatStep Pro friendly, MIDI-mappable).
- **Layering:** works as a final effect on a group in Resolume (LUT/colour effect on the whole composition) so everything beneath becomes three colours; also on a Spout/NDI feed from Unreal or TouchDesigner. Over a mixed bus it unifies unrelated clips into one visual world, a useful trick for hybrid sets.
- **Boring risks:** static threshold on low-contrast footage gives mush; constant full red gets heavy over a long set; no motion in the source means no life. Fixes: feed it high-contrast, high-motion sources (faces, hands, text, geometry, feedback), and keep black the dominant colour with red as the accent.

## Variations
1. **Beat-gated red:** red appears only on the kick; between hits the image is pure black/white.
2. **Wedge mask:** use a rotating wedge or circle (Lissitzky) to decide where the red tone is allowed, instead of thresholding it from the picture.
3. **Palette drift:** the red hue slowly moves while black and white stay locked, or a second accent swaps in for a breakdown.
4. **Kruger layer:** white-on-red caption bars with bold condensed type over the crushed image, text from a MIDI-keyed list.
5. **Spot colour mode:** same pipeline but black/white with red only in the top band of luminance, the Sin City reading.

## Facts
- "Tritone" in print is a real colour-mode term next to duotone and quadtone, set up through the Duotone dialog in Photoshop ([O'Reilly](https://www.oreilly.com/library/view/real-world-print/9780321637598/9780321637598_ch09lev2sec13.html)).
- Duotone traditionally pairs black with a second colour such as blue, yellow, brown or red ([Wikipedia](https://en.wikipedia.org/wiki/Duotone)).
- Musical tritone = 600 cents, exactly half an octave in equal temperament ([Wikipedia](https://en.wikipedia.org/wiki/Tritone)).
- Resolume only imports `.cube` LUTs ([Resolume](https://resolume.com/support/de/lut)).
- The Man-Machine's back cover adapts a graphic from Lissitzky's *About Two Squares* ([Wikipedia](https://en.wikipedia.org/wiki/The_Man-Machine)).
- Kruger's red-and-white caption style uses Futura Bold Oblique or Helvetica Ultra Condensed ([Wikipedia](https://en.wikipedia.org/wiki/Barbara_Kruger)).

## Visual references
- [V&A: The Man-Machine record cover](https://collections.vam.ac.uk/item/O1192920/the-man-machine-record-cover/) — red/black/white Lissitzky-inspired sleeve, front and back
- [John Coulthart: El Lissitzky record covers](https://www.johncoulthart.com/feuilleton/2014/08/18/el-lissitzky-record-covers) — gallery of covers borrowing the red/black/white Lissitzky look
- [Fonts In Use: Franz Ferdinand identity](https://www.fontsinuse.com/uses/17163/franz-ferdinand-franz-ferdinand-andamp-single) — Constructivist-referencing band identity, images of the sleeves
- [Wikipedia: Beat the Whites with the Red Wedge](https://en.wikipedia.org/wiki/Beat_the_Whites_with_the_Red_Wedge) — the poster itself, page with image
- [Wikipedia: Barbara Kruger](https://en.wikipedia.org/wiki/Barbara_Kruger) — page with examples of her white-on-red text over B&W photographs
- [NFB: Tower Bawher](https://www.nfb.ca/film/tower_bawher/) — animated Constructivist short, watchable on the NFB site

## Lead image
NONE. The Wikimedia Commons file pages for the Lissitzky poster could not be opened in this run, so licence and author could not be checked. Candidate for the editor to verify: a Commons file of *Beat the Whites with the Red Wedge* (Lissitzky died 1941, so the work itself is likely public domain, but the specific file page must be checked).

## Sources
- https://en.wikipedia.org/wiki/Duotone
- https://www.oreilly.com/library/view/real-world-print/9780321637598/9780321637598_ch09lev2sec13.html
- https://en.wikipedia.org/wiki/Tritone
- https://en.wikipedia.org/wiki/Beat_the_Whites_with_the_Red_Wedge
- https://en.wikipedia.org/wiki/The_Man-Machine
- https://collections.vam.ac.uk/item/O1192920/the-man-machine-record-cover/
- https://www.johncoulthart.com/feuilleton/2014/08/18/el-lissitzky-record-covers
- https://www.fontsinuse.com/uses/17163/franz-ferdinand-franz-ferdinand-andamp-single
- https://en.wikipedia.org/wiki/Barbara_Kruger
- https://en.wikipedia.org/wiki/Sin_City_(film)
- https://www.nfb.ca/film/tower_bawher/
- https://en.wikipedia.org/wiki/Tower_Bawher
- https://resolume.com/support/de/lut
- https://cdm.link/tag/lut
