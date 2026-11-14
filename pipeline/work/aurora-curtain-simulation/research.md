# Aurora Curtain Simulation — research

## Origin
- The name goes back to Pierre Gassendi's 1649 description of a 1621 display. He used "aurora borealis", a term Galileo had used in 1619. It comes from the Roman goddess of dawn ([Wikipedia: Aurora](https://en.wikipedia.org/wiki/Aurora)).
- Kristian Birkeland aimed an electron beam at a magnetised sphere (a "terrella") in a vacuum chamber. The beam went to the magnetic poles and made rings of light, so he concluded the aurora forms the same way. In 1908 he proposed field-aligned currents (now Birkeland currents). Mainstream science dismissed this at first. Satellite magnetometer data confirmed it in 1967 ([Wikipedia: Kristian Birkeland](https://en.wikipedia.org/wiki/Kristian_Birkeland)).
- Carl Størmer triangulated more than 12,000 auroras with cameras from 1911. He found none below 70 km, a height maximum near 100 km, and only 6.5% above 150 km ([Wikipedia: Aurora](https://en.wikipedia.org/wiki/Aurora)).
- Real-time rendering of curtains in games and engines has a literature of its own. One example is the volume ray-caster in Gaia Sky, based on a 2011 paper by Lawlor and Genetti ([Toni Sagristà](https://tonisagrista.com/blog/2024/rendering-aurorae-nebulae/)).

## Key works and artists
- **Project Aurora**, Ginny Ruffner: a wall of 34,560 LEDs, 20 ft high and 10 ft wide, now at the National Nordic Museum. It shows sheets of coloured light drawn from aurora images. Its microprocessors refresh it 20 times per second ([Nordic Museum](https://nordicmuseum.org/exhibitions/project-aurora)).
- **Mytherrella**, Dorin Cucicov, shown at ISEA2023 in Paris: an interactive installation that mixes aurora science with mythology. A custom real-time StyleGAN trained on all-sky aurora images generates the video, shaped by visitors' live interaction ([ISEA archive](https://www.isea-symposium-archives.org/?p=75676)).
- **Night Curtains**: a free browser simulator. It renders procedural curtains on the GPU in WebGL, so they never loop. It has a storm level from KP 1 to 9, palette choice, sky motion and landscape options ([Night Curtains](https://arcade.pirillo.com/night-curtains.html)).
- **Gaia Sky aurora renderer**: a volume ray-caster. Hand-drawn footprints are stretched vertically with an electron-deposition function. Colours come from a JSON descriptor. It runs at 20–80 FPS ([Toni Sagristà](https://tonisagrista.com/blog/2024/rendering-aurorae-nebulae/)).

## How it works
- Electrons and protons from the solar wind are guided by Earth's magnetic field into the upper atmosphere. They excite or ionise atoms and molecules. These release the energy as light when they relax ([Wikipedia: Aurora](https://en.wikipedia.org/wiki/Aurora)).
- Colour depends on the gas and on altitude:
  - **Green, oxygen, 557.7 nm.** Lifetime of the excited state is about 0.7 s. Typical curtain colour.
  - **Red, oxygen, 630.0 nm.** Emitted at the highest altitudes. The transition is so slow (up to about two minutes) that it only finishes where collisions are rare. Mostly seen during strong solar activity.
  - **Blue and purple, molecular nitrogen, about 428 nm.** Emitted at the lowest altitudes and much faster than oxygen. It appears at the lower edge of curtains in high activity.
  - Source: [Wikipedia: Aurora](https://en.wikipedia.org/wiki/Aurora).
- Stack from top to bottom: red, then green, then blue or purple. Auroras form above about 80 km. The sharp lower edge of a curtain exists because atomic oxygen becomes scarce below about 100 km ([Wikipedia: Aurora](https://en.wikipedia.org/wiki/Aurora)). Other sources give different altitude numbers for the green layer. One says roughly 100–300 km for green, with higher oxygen red and nitrogen pink or purple at the lower edge ([Night Curtains](https://arcade.pirillo.com/night-curtains.html)).
- Curtain form comes from viewpoint and field geometry. Arcs can break into fast-changing rays ([Wikipedia: Aurora](https://en.wikipedia.org/wiki/Aurora)).
- The light does not travel. Atoms stay put. Shifting field conditions send particles to other places, so the pattern seems to flow ([Discover](https://www.discovermagazine.com/real-time-footage-of-aurora-shows-them-dancing-and-shimmering-13807)).

## Visual language
- Tall vertical curtains with a bright, sharp lower edge and a soft fade upward. Folds and rays run along the vertical.
- Gradient by height: purple or pink hem, green body, red crown.
- Almost black ground (silhouette horizon, stars) so the glow carries the image.
- Slow, wide motion in time-lapse. Real-time footage is shimmering and dancing, and stars hold still ([Discover](https://www.discovermagazine.com/real-time-footage-of-aurora-shows-them-dancing-and-shimmering-13807)).
- Pulsating auroras vary on roughly 2–20 s periods. Some changes happen within a second ([Wikipedia: Aurora](https://en.wikipedia.org/wiki/Aurora)).

## Live potential
- Mapping ideas (my suggestions, based on the timescales above):
  - Kick: a brightness pulse on the lower edge, or a ray burst.
  - Bass level: curtain height and ray length.
  - Hi-hats: fine shimmer, a high-frequency ray flicker.
  - BPM: pulsation period, sync to 2–20 s patterns (4 to 16 beats).
  - Faders: wave speed, fold count, altitude spread of colour bands, palette, storm level (borrowing the KP idea).
- Substorms are the natural "drop": a quiet arc turns into an active display in a few minutes ([Wikipedia: Aurora](https://en.wikipedia.org/wiki/Aurora)).
- Layers well as a backdrop beneath silhouettes, typography or line work. Additive blending works because the image is mostly black.
- Boring risks: a pure slow green wash with no structure, and no change of scale. Needs ray detail, a palette shift or a storm event.

## Variations
- Non-physical palettes: swap oxygen green for magenta and gold, keep the altitude order.
- Reverse the stack, or let the three colour bands drift out of sync.
- Curtains over a city or architecture silhouette via a mask from another layer.
- Terrella view: a glowing ring around a magnetised sphere, after Birkeland ([Wikipedia: Kristian Birkeland](https://en.wikipedia.org/wiki/Kristian_Birkeland)).
- Pixel-LED wall version: sheets drawn from aurora images on a low-res grid, as in Project Aurora ([Nordic Museum](https://nordicmuseum.org/exhibitions/project-aurora)).

## Facts
- The red 630 nm transition takes up to about two minutes, the green one about 0.7 s ([Wikipedia: Aurora](https://en.wikipedia.org/wiki/Aurora)).
- Størmer's survey found none below 70 km ([Wikipedia: Aurora](https://en.wikipedia.org/wiki/Aurora)).
- Birkeland's theory was dismissed in his lifetime and confirmed by satellite data in 1967 ([Wikipedia: Kristian Birkeland](https://en.wikipedia.org/wiki/Kristian_Birkeland)).
- Most aurora videos are time-lapse, because real movement is usually slow. Real-time clips look very different ([Discover](https://www.discovermagazine.com/real-time-footage-of-aurora-shows-them-dancing-and-shimmering-13807)).
- Project Aurora has 34,560 LEDs and refreshes 20 times per second ([Nordic Museum](https://nordicmuseum.org/exhibitions/project-aurora)).

## Visual references
- [Dance of the Spirits: Incredible Real Time Northern Lights from Jan 24 Solar Storm](https://www.youtube.com/watch?v=SkiNUMKKsgM) — real-time footage of a solar-storm display, shimmering curtains in motion (channel: ingenioustv by Alister Chapman).
- [Friday the 13th Aurora](https://vimeo.com/238450470) — aurora film by Alan Dyer (title and author confirmed via Vimeo oEmbed).
- [Project Aurora, National Nordic Museum](https://nordicmuseum.org/exhibitions/project-aurora) — LED wall of moving aurora sheets.
- [Night Curtains](https://arcade.pirillo.com/night-curtains.html) — live procedural aurora simulator with storm level and palette controls.
- [Mytherrella, ISEA2023](https://www.isea-symposium-archives.org/?p=75676) — interactive generative aurora installation.

## Lead image
- file_page: https://commons.wikimedia.org/wiki/File:Northern_lights_curtain_over_Nordmo_in_Bjerkvik,_Narvik,_Nordland,_Norway,_2023_September.jpg
- image_url: https://upload.wikimedia.org/wikipedia/commons/thumb/f/fb/Northern_lights_curtain_over_Nordmo_in_Bjerkvik%2C_Narvik%2C_Nordland%2C_Norway%2C_2023_September.jpg/1280px-Northern_lights_curtain_over_Nordmo_in_Bjerkvik%2C_Narvik%2C_Nordland%2C_Norway%2C_2023_September.jpg
- shows: green aurora curtain and beams over Nordmo, Bjerkvik, Norway (portrait photo)
- author: Ximonic (Simo Räsänen)
- license: CC BY-SA 4.0
- license_url: https://creativecommons.org/licenses/by-sa/4.0

## Sources
- [Wikipedia: Aurora](https://en.wikipedia.org/wiki/Aurora)
- [Wikipedia: Kristian Birkeland](https://en.wikipedia.org/wiki/Kristian_Birkeland)
- [Toni Sagristà: Rendering aurorae and nebulae](https://tonisagrista.com/blog/2024/rendering-aurorae-nebulae/)
- [National Nordic Museum: Project Aurora](https://nordicmuseum.org/exhibitions/project-aurora)
- [ISEA archive: Mytherrella](https://www.isea-symposium-archives.org/?p=75676)
- [Night Curtains](https://arcade.pirillo.com/night-curtains.html)
- [Discover Magazine](https://www.discovermagazine.com/real-time-footage-of-aurora-shows-them-dancing-and-shimmering-13807)
- [Commons file page](https://commons.wikimedia.org/wiki/File:Northern_lights_curtain_over_Nordmo_in_Bjerkvik,_Narvik,_Nordland,_Norway,_2023_September.jpg)
