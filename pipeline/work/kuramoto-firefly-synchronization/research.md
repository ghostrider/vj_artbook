# Kuramoto Firefly Synchronization — research

## Origin
The Kuramoto model is named after Yoshiki Kuramoto, whose 1975 paper in *Lecture Notes in Physics* is the earliest reference given for it. It was motivated by chemical and biological oscillators ([Wikipedia](https://en.wikipedia.org/wiki/Kuramoto_model)). Veritasium's page on the topic links the same 1975 paper and draws on Steven Strogatz's book *Sync* ([Veritasium](https://veritasium.com/videos/2021/3/31/the-secret-of-synchronization)).

Fireflies are the natural showcase. About 1 percent of roughly 2,000 firefly species synchronise their flashes over large areas. One example is *Photinus carolinus* in the Great Smoky Mountains ([UConn Today](https://today.uconn.edu/?p=39950)). Strictly, firefly flash coupling is a pulse-coupled rule (see How it works), and Kuramoto is the smooth-phase cousin.

## Key works and artists
- **Nicky Case, "Fireflies"** (explorable). Dots in darkness start random, form clusters, then waves, then near-total unison. Dragging scatters the swarm and it re-syncs ([ncase.me/fireflies](https://ncase.me/fireflies/)).
- **Kuramoto Web Installation** (2021). David Pirrò, Hanns Holger Rutz, Luc Döbereiner and Daniele Pozzi made it as part of the ALMAT research project. Coupled 1D Kuramoto oscillators are shown as grey-scale phase with time on the vertical axis ([Research Catalogue](https://www.researchcatalogue.net/view/552571/621219)).
- **Contingency and Synchronization.** Luc Döbereiner and David Pirrò made this sound installation, shown at the Orpheus Institute in Ghent, March 2019. It has six loudspeaker-microphone pairs. Each speaker's oscillator tries to lock onto what its microphone hears, based on the Kuramoto model ([pirro.mur.at](https://pirro.mur.at/contsync.html)).
- **Interactive Kuramoto animation** by hdietert: phases as points on a unit circle, with a histogram, order-parameter plot and controls for coupling, noise and frequency distribution ([page](https://hdietert.github.io/static/kuramoto-animation/kuramoto.html)).

## How it works
The Kuramoto model has N oscillators, each with its own natural frequency. Each is coupled to all the others, and the pull depends sinusoidally on the phase difference. Below a critical coupling the population stays incoherent. Above it, synchrony emerges, and with strong coupling all oscillators share one frequency. The order parameter r measures coherence: r = 0 means phases spread uniformly, and r near 1 means unison. ψ is the mean phase ([Wikipedia](https://en.wikipedia.org/wiki/Kuramoto_model)).

The firefly version in Nicky Case's explorable is simpler. Each firefly has a clock and flashes at "twelve". Seeing a neighbour flash nudges its clock forward. That one local rule is all the coupling, with no leader ([ncase.me](https://ncase.me/fireflies/), [Bram.us](https://bram.us/2021/04/03/synchronizing-fireflies-visualizer)).

## Visual language
- Phase maps naturally to colour (hue wheel), to brightness (flash on phase wrap) or to ring angle.
- The incoherent state looks like twinkling noise. Near the threshold, clusters and travelling waves form. At high coupling the field pulses as one.
- Phase on a unit circle shows the swarm as a dot cloud that collapses into a single clump ([hdietert](https://hdietert.github.io/static/kuramoto-animation/kuramoto.html)).
- Time on one axis gives stripe or raster patterns ([Research Catalogue](https://www.researchcatalogue.net/view/552571/621219)).

## Live potential
- Coupling strength K on a fader: the swarm goes from chaos to unison.
- Frequency spread on a fader: wide spread resists sync.
- A kick resets or kicks all phases to a target angle. The hdietert demo has an "Add perturbation" control with target angle and strength, which suggests the same idea.
- Hi-hats could add a small noise term. The same demo has an adjustable noise strength.
- BPM sets the mean natural frequency. Lock the synced pulse to the beat and let the unison flash land on the downbeat.
- The scalar r can drive other layers: bloom, camera shake, or a bass swell. It layers well as additive points over feedback, or as a mask.
- Boring risk: if it sits fully synced or fully random for long. Hold the system near the critical coupling and use beats to scatter it.

## Variations
- Spatial coupling only with neighbours, so waves sweep across a grid.
- Rings of oscillators, each ring a different frequency band, with phase as ring rotation.
- Two populations with different frequency distributions. The hdietert demo offers a two-population (bi-Cauchy) option.
- Audio feedback: oscillator phases modulate shader parameters, with the audio's onset pulses as the coupling input, like the sound installation above.
- Drag or MIDI-key "scatter" gestures, as in Nicky Case's explorable.

## Facts
- Strogatz's Millennium Bridge paper (Nature, 3 Nov 2005) attributes the 2000 opening-day wobble to pedestrians unintentionally falling into step. The critical crowd was as low as 160, with up to 2,000 on the bridge ([ScienceDaily](https://www.sciencedaily.com/releases/2005/11/051103080801.htm)).
- About 1 percent of firefly species synchronise over large areas ([UConn Today](https://today.uconn.edu/?p=39950)).
- Female *P. carolinus* in a lab test responded over 80 percent of the time to synchronised flashes and 10 percent or less to out-of-sync ones ([UConn Today](https://today.uconn.edu/?p=39950)).
- Kuramoto was surprised when arrays of Josephson junctions followed his model ([Wikipedia](https://en.wikipedia.org/wiki/Kuramoto_model)).
- Applications listed include neuroscience, power grids and flocking ([Wikipedia](https://en.wikipedia.org/wiki/Kuramoto_model)).

## Visual references
- [Nicky Case: Fireflies](https://ncase.me/fireflies/) — glowing dots flashing, clustering into waves, then unison; drag to scatter.
- [Kuramoto animation (hdietert)](https://hdietert.github.io/static/kuramoto-animation/kuramoto.html) — phases on a unit circle, order-parameter plot, perturbation button.
- [Kuramoto Web Installation](https://www.researchcatalogue.net/view/552571/621219) — grey-scale phase raster of coupled oscillators over time.
- [Contingency and Synchronization](https://pirro.mur.at/contsync.html) — sound installation documentation; page links audio and video recordings.
- [Veritasium: The Secret of Synchronization](https://veritasium.com/videos/2021/3/31/the-secret-of-synchronization) — page for the video on sync, with the Kuramoto and firefly interactives in its references.

## Lead image
NONE. The Wikimedia Commons file pages found in search (Kuramoto model animations) could not be opened to confirm licence and author, so no image is claimed.

## Sources
- [Wikipedia: Kuramoto model](https://en.wikipedia.org/wiki/Kuramoto_model)
- [Veritasium page](https://veritasium.com/videos/2021/3/31/the-secret-of-synchronization)
- [UConn Today](https://today.uconn.edu/?p=39950)
- [Nicky Case, Fireflies](https://ncase.me/fireflies/)
- [Bram.us](https://bram.us/2021/04/03/synchronizing-fireflies-visualizer)
- [Research Catalogue](https://www.researchcatalogue.net/view/552571/621219)
- [pirro.mur.at](https://pirro.mur.at/contsync.html)
- [hdietert animation](https://hdietert.github.io/static/kuramoto-animation/kuramoto.html)
- [ScienceDaily](https://www.sciencedaily.com/releases/2005/11/051103080801.htm)
