# Pulsar Ridgeline Waveforms — research

## Origin
- **Discovery.** Jocelyn Bell (now Bell Burnell), then a 24-year-old PhD student at the Mullard Radio Astronomy Observatory near Cambridge, found a faint recurring signal in her chart recordings. Her supervisor was Antony Hewish. The signal was first recorded on 6 August 1967, and regular pulses were first observed on 28 November 1967. Pulses arrive about 1.337 s apart. The object was first labelled LGM ("Little Green Men", as a joke) and later named CP 1919, for Cambridge Pulsar at 19h 19m right ascension ([University of Cambridge](https://www.cam.ac.uk/stories/journeysofdiscovery-pulsars)). It is now called PSR B1919+21 ([Big Think](https://bigthink.com/strange-maps/known-pleasures-from-pulsar-signal-to-post-punk-icon/)).
- **Early follow-up.** R. Wielebinski observed CP 1919 at the Parkes telescope in March 1968 at five radio frequencies and analysed its pulse statistics ([Nature, 14 Sept 1968](https://www.nature.com/articles/2191135a0)).
- **The stacked plot.** Harold D. Craft Jr., a doctoral student working at Arecibo, made the figure for his 1970 Cornell PhD thesis, "Radio observations of the pulse profiles and dispersion measures of twelve pulsars". It is figure 5-37 on page 215 ([Timmes, cococubed](https://cococubed.com/pix_pages/joy_division_unknown_pleasures.shtml)). It stacks 80 successive pulses of CP 1919 recorded at 318 MHz. Each pulse lasts about 0.04 s and they repeat every 1.337 s ([Scientific American](https://www.scientificamerican.com/article/the-pulsar-chart-that-became-a-pop-icon-turns-50-joy-division-rsquo-s-unknown-pleasures/)).
- **Craft's method.** He wrote the program that stacked the pulses and tilted the stack slightly, so that it looked like "looking up a hillside". A Cornell draftsperson traced it in India ink for print ([Big Think](https://bigthink.com/strange-maps/known-pleasures-from-pulsar-signal-to-post-punk-icon/)).
- **Reprints.** Scientific American, Jan 1971 (Ostriker), used white lines on cyan. Graphis Diagrams (1974) printed it white on black. The Cambridge Encyclopaedia of Astronomy (1977) printed it black on white ([Big Think](https://bigthink.com/strange-maps/known-pleasures-from-pulsar-signal-to-post-punk-icon/), [cococubed](https://cococubed.com/pix_pages/joy_division_unknown_pleasures.shtml)). Designer Peter Saville used an inverted version for Joy Division's 1979 album cover ([Scientific American](https://www.scientificamerican.com/article/the-pulsar-chart-that-became-a-pop-icon-turns-50-joy-division-rsquo-s-unknown-pleasures/)).
- **Ridgeline plots as a technique.** The name "joy plot" was proposed by Jenny Bryan in April 2017 and spread through Claus Wilke's ggjoy R package (July 2017). Wilke renamed the style "ridgeline plot" in September 2017, because the band name has a dark origin that most data people did not know ([Wilke](https://clauswilke.com/blog/2017/09/15/goodbye-joyplots/)). Use the term "ridgeline" in the chapter.

## Key works and artists
- **Harold D. Craft Jr.**: the original 1970 figure 5-37 (see Origin). He later directed Arecibo Observatory, 1973 to 1982 ([Big Think](https://bigthink.com/strange-maps/known-pleasures-from-pulsar-signal-to-post-punk-icon/)).
- **Jen Christiansen** (Scientific American) traced the plot back to Craft in a 2015 article, and wrote the 50th-anniversary piece ([cococubed](https://cococubed.com/pix_pages/joy_division_unknown_pleasures.shtml), [Scientific American](https://www.scientificamerican.com/article/the-pulsar-chart-that-became-a-pop-icon-turns-50-joy-division-rsquo-s-unknown-pleasures/)).
- **Frank Timmes** made vector recreations of Craft's figure, as an SVG and a PDF ([cococubed](https://cococubed.com/pix_pages/joy_division_unknown_pleasures.shtml)).
- **Joy Division, *Unknown Pleasures* (1979)**: the album cover uses an inverted version of this plot. It is copyrighted. Mention and link it only: [cover context at Scientific American](https://www.scientificamerican.com/article/the-pulsar-chart-that-became-a-pop-icon-turns-50-joy-division-rsquo-s-unknown-pleasures/). The chapter is about the general ridgeline technique.
- **ggridges / ggjoy (Claus Wilke)**: the R tooling that made ridgeline plots a standard chart type ([Wilke](https://clauswilke.com/blog/2017/09/15/goodbye-joyplots/)).

## How it works
- **Physics.** A pulsar is a dense, rapidly rotating star that sends a beam of radio waves across the sky like a lighthouse. Each rotation gives one pulse ([University of Cambridge](https://www.cam.ac.uk/stories/journeysofdiscovery-pulsars)).
- **Why stack.** A single pulse is noisy and varies in shape. Plotting many consecutive pulses one under the other shows both the steady period (a stable column of peaks) and the pulse-to-pulse variation. Craft's stack uses 80 pulses ([Scientific American](https://www.scientificamerican.com/article/the-pulsar-chart-that-became-a-pop-icon-turns-50-joy-division-rsquo-s-unknown-pleasures/)).
- **Ridgeline technique.** Each row is a 1D curve (time within one period on x, intensity as height). Rows are offset in y and overlap. Each row is filled with the background colour so lower-in-front rows hide the lines behind them. This is the hidden-line effect. In ggridges this is a height value per x for each group, with a scale factor that controls overlap ([R Graph Gallery](https://r-graph-gallery.com/ridgeline-plot), [R-bloggers CP 1919 example](https://www.r-bloggers.com/2024/06/cp-1919-psr-b191921-dataset-2/)).
- **Draw order.** The example code reverses the y axis so the first trace is at the top and later traces are drawn in front ([R-bloggers](https://www.r-bloggers.com/2024/06/cp-1919-psr-b191921-dataset-2/)).
- **Data.** A CP 1919 dataset of 24,000 rows (x, y, z) is available as an R dataset. Its creator is unknown and its licence is not stated on the page ([R-bloggers](https://www.r-bloggers.com/2024/06/cp-1919-psr-b191921-dataset-2/)). Do not rely on it. A live version should generate its own data.

## Visual language
- Many thin parallel lines in perspective, with peaks that cluster in a narrow band, giving a mountain-range look.
- Hidden-line occlusion: fill of each row equals the background, so a peak cuts off the lines behind it.
- Two colour treatments appear in the historical record: dark fill with white lines, and white fill with black lines. A third, light-blue fill with white lines, ran in Scientific American ([cococubed](https://cococubed.com/pix_pages/joy_division_unknown_pleasures.shtml)).
- Rhythm comes from the stable column of peaks. The tension comes from the noise on every line.

## Live potential
- **Scrolling history.** Each new frame, or each beat, pushes a new row in at the back or front and shifts old rows. Row n holds the waveform from n steps ago, so rhythm travels visibly through the mountain.
- **Kick.** Trigger one tall pulse in the newest row, or a pulse of the whole amplitude scale.
- **Bass.** Row amplitude scale and overall tilt.
- **Hi-hats.** High-frequency jitter on the line, or a brightness flick on the front rows.
- **BPM.** Set the row spawn to a beat division, so one row equals one beat or sixteenth. The pulse column then locks to the tempo, as the 1.337 s period does in the original.
- **MIDI faders.** Row count, spacing, tilt, noise amount, line width, pulse position and width.
- **Layering.** Black fill makes it work as a mask or matte over other material. With a transparent fill, other layers show through between lines. Use it as a luma key over video.
- **What gets boring.** A static, symmetrical stack with no variation, a fixed tilt, or white-on-black used with no other change for a whole set. Needs noise, rhythm changes and a way to break the grid.

## Variations
1. **Spectrum ridgeline.** Each row is an FFT snapshot instead of a time waveform, so frequency becomes x.
2. **Polar ridgeline.** Rows as concentric rings, peaks pushing outward.
3. **Colour by age.** Rows fade through a palette as they age, driven by BPM.
4. **3D rotation.** Rotate the stack in Unreal as a mesh or Niagara ribbons, with the camera flying along the ridges.
5. **Row feedback.** Row n feeds from row n-1 through a delay, so a kick echoes down the stack.
(These are design ideas, not claims from sources.)

## Facts
- The pulsar was nicknamed LGM, for Little Green Men ([University of Cambridge](https://www.cam.ac.uk/stories/journeysofdiscovery-pulsars)).
- The 1974 Nobel Prize in Physics went to Hewish and Martin Ryle, not to Bell ([University of Cambridge](https://www.cam.ac.uk/stories/journeysofdiscovery-pulsars)).
- Craft only learned of the album cover when a Cornell professor pointed it out ([Big Think](https://bigthink.com/strange-maps/known-pleasures-from-pulsar-signal-to-post-punk-icon/)).
- The plot was black on white in the Cambridge Encyclopaedia of Astronomy (1977); the band saw that version, and Saville inverted it ([Big Think](https://bigthink.com/strange-maps/known-pleasures-from-pulsar-signal-to-post-punk-icon/)).
- Each pulse lasts about 0.04 s, so the pulse fills only about 3 percent of the 1.337 s period ([Scientific American](https://www.scientificamerican.com/article/the-pulsar-chart-that-became-a-pop-icon-turns-50-joy-division-rsquo-s-unknown-pleasures/); the percentage is a simple calculation).
- "Joyplot" was renamed "ridgeline plot" in 2017 ([Wilke](https://clauswilke.com/blog/2017/09/15/goodbye-joyplots/)).

## Visual references
- [Timmes: Unknown Pleasures / CP 1919 page](https://cococubed.com/pix_pages/joy_division_unknown_pleasures.shtml) — history of the plot with vector recreations of Craft's figure 5-37 and its reprints in several colour schemes.
- [Scientific American: the pulsar chart turns 50](https://www.scientificamerican.com/article/the-pulsar-chart-that-became-a-pop-icon-turns-50-joy-division-rsquo-s-unknown-pleasures/) — the plot's history, including the white-on-cyan Scientific American version.
- [R-bloggers: CP 1919 dataset in ggridges](https://www.r-bloggers.com/2024/06/cp-1919-psr-b191921-dataset-2/) — the CP 1919 stack redrawn as a ridgeline plot in three colour schemes.
- [R Graph Gallery: ridgeline chart](https://r-graph-gallery.com/ridgeline-plot) — examples of the general ridgeline technique with colour gradients.

## Lead image
- kind: similar
- image_url: https://images.unsplash.com/photo-1768280511074-3b3effe7a139?w=1400
- page_url: https://unsplash.com/photos/abstract-black-and-white-wavy-lines-pattern-xP6ngsgxck0
- photographer: Resource Database (@resourcedatabase)
- license: Unsplash License (free, not Unsplash+), https://unsplash.com/license
- shows: "Abstract black and white wavy lines pattern" (tags include line art, data visualization, topographic map)

## Quotation images
- kind: quotation
- image_url: https://framemark.vam.ac.uk/collections/2011EW6763/full/!600,/0/default.jpg
- page_url: https://collections.vam.ac.uk/item/O191218/unknown-pleasures-record-sleeve-peter-saville
- work: Unknown Pleasures record sleeve, Joy Division, design Peter Saville, published by Factory Records, 1979
- rights: © Peter Saville and Factory Records (photo © Victoria and Albert Museum, London); object given by Brett Wickens (Peter Saville Associates)
- site: Victoria and Albert Museum collections (official museum record; image URL is the page's og:image)
- note: no free-licence image found (Commons not reachable; Commons pulsar charts are likely copies of the cover; cococubed recreations have no stated licence).

## Sources
- [University of Cambridge: journeys of discovery, pulsars](https://www.cam.ac.uk/stories/journeysofdiscovery-pulsars)
- [Scientific American: the pulsar chart turns 50](https://www.scientificamerican.com/article/the-pulsar-chart-that-became-a-pop-icon-turns-50-joy-division-rsquo-s-unknown-pleasures/)
- [Big Think: from pulsar signal to post-punk icon](https://bigthink.com/strange-maps/known-pleasures-from-pulsar-signal-to-post-punk-icon/)
- [Timmes, cococubed](https://cococubed.com/pix_pages/joy_division_unknown_pleasures.shtml)
- [Wilke: Goodbye joyplots](https://clauswilke.com/blog/2017/09/15/goodbye-joyplots/)
- [Wielebinski, Nature 1968](https://www.nature.com/articles/2191135a0)
- [R-bloggers CP 1919 dataset](https://www.r-bloggers.com/2024/06/cp-1919-psr-b191921-dataset-2/)
- [R Graph Gallery ridgeline](https://r-graph-gallery.com/ridgeline-plot)
