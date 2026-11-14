# Boids Murmuration Flocks — research

## Origin
Craig Reynolds built the boids model in 1986 and published it at SIGGRAPH in 1987 as "Flocks, herds and schools: A distributed behavioral model". The name is short for "bird-oid object" ([Wikipedia](https://en.wikipedia.org/wiki/Boids), [Reynolds](https://www.red3d.com/cwr/boids/)). He developed it with colleagues at Symbolics Graphics Division and Whitney/Demos Productions. The first boid animation was the short *Stanley and Stella in: Breaking the Ice* (SIGGRAPH '87 Electronic Theater). *Batman Returns* (1992) was the first feature film to use it, for bat swarms and penguin armies ([Reynolds](https://www.red3d.com/cwr/boids/)). Ichiro Aoki published closely related work in 1982 ([Wikipedia](https://en.wikipedia.org/wiki/Boids)).

The natural counterpart is the starling murmuration. The STARFLAG project filmed real flocks in 3D and found that each bird reacts to a fixed number of neighbours, about seven, regardless of distance ([The Contrarian](https://contrarian.ca/2012/06/20/the-seventh-starling/)).

## Key works and artists
- **Craig Reynolds, boids and steering behaviors**: the original model and the wider steering-behavior family, with the open-source C++ OpenSteer ([boids page](https://www.red3d.com/cwr/boids/), [steering behaviors](https://www.red3d.com/cwr/steer/)).
- **Robert Hodgin, *Murmuration* (2015 page)**: a flocking system aimed at the look of starlings, reworked for about two decades. It went from about 1,000 CPU boids to 40,000 on the GPU, and to 200,000–500,000 in Houdini. A friend's compute-shader port ran 30+ fps with over a million flockers ([Hodgin](https://roberthodgin.com/project/murmuration)).
- **Sebastian Lague, "Coding Adventure: Boids"** (2019): a popular tutorial in Unity ([YouTube](https://www.youtube.com/watch?v=bqtqltqcQhw)).
- **Batman Returns (1992)**: boids in a feature film ([Reynolds](https://www.red3d.com/cwr/boids/)).

## How it works
Every boid looks only at flockmates in a local neighbourhood and applies three steering rules ([Reynolds](https://www.red3d.com/cwr/boids/)):
1. Separation: steer away from crowding neighbours.
2. Alignment: match their average heading.
3. Cohesion: move toward their average position.

Complex global motion emerges from these simple local rules. Early versions also had obstacle avoidance and goal seeking. Real starlings use a topological rule (fixed neighbour count) rather than a fixed radius. As a result the interaction range shrinks in dense flocks, and the flock can change shape, fluctuate and even split while staying cohesive ([The Contrarian](https://contrarian.ca/2012/06/20/the-seventh-starling/)). In a boids implementation this means a k-nearest-neighbours query instead of a radius query.

## Visual language
Thousands of small agents form one fluid body: dense dark ribbons, sudden thinning, waves of turning that pass through the cloud, splits and merges. Single-colour points or tiny oriented wedges against a light or dark field work best. The Commons photo below shows the dense, smoky look of a real murmuration. Hodgin's work aims at the same starling density, with very large counts.

## Live potential
- **Kick**: impulse on the cohesion weight (flock contracts into a tight ball) or a short burst of speed.
- **Bass or sustained level**: map to separation or noise, so the flock loosens and "breathes".
- **Hi-hats**: small random turn kicks, or a jitter in heading for shimmer.
- **BPM**: orbit a goal or "predator" point on a beat-synced path. A predator that flees or attacks on bar boundaries drives the flock to split and regroup.
- **MIDI faders**: flock size, speed, neighbour count (k), rule weights, trail length, point size.
- **Layering**: use the flock density or positions as a mask or displacement source for other layers. It also works as a particle layer in Unreal Niagara or Wire, and as a Spout/NDI source.
- **Boring on a big screen**: static weights make a flock settle into a uniform swirl. Without a changing predator or goal, it only circles. Use contrast, scale changes and long trails sparingly.

## Variations
1. Predator-driven: the predator is a MIDI-note-controlled or beat-orbiting point that triggers splits.
2. Topological k-neighbour flocks, with the fader changing k live from a loose swarm to a tight one.
3. Density-as-image: render the flock into a density buffer and use it as a luminance mask for feedback or video.
4. Multi-species: two or three flocks with different colours that avoid or chase each other on different beats.
5. Letters or shapes: add a goal field so the flock briefly forms a logo, then dissolves on the drop.

## Facts
- Reynolds's boids debuted in a feature film in 1992 with Batman Returns ([Reynolds](https://www.red3d.com/cwr/boids/)).
- The boid model has been applied to swarm robotics, data visualisation and optimisation ([Wikipedia](https://en.wikipedia.org/wiki/Boids)).
- Real starlings are modelled with about seven fixed neighbours rather than a distance ([The Contrarian](https://contrarian.ca/2012/06/20/the-seventh-starling/)).
- Hodgin's flock simulation has scaled from about 1,000 CPU agents to over a million on a GPU compute-shader port ([Hodgin](https://roberthodgin.com/project/murmuration)).
- The three flocking rules are one of many steering behaviors Reynolds documents, including seek, flee, wander and flow-field following ([Reynolds steering](https://www.red3d.com/cwr/steer/)).

## Visual references
- [Craig Reynolds, Boids](https://www.red3d.com/cwr/boids/) — the original background page on the model, with links to demos.
- [Robert Hodgin, Murmuration](https://roberthodgin.com/project/murmuration) — project page of a large starling-style flock simulation, with an image.
- [Coding Adventure: Boids](https://www.youtube.com/watch?v=bqtqltqcQhw) — Sebastian Lague's tutorial video (channel Sebastian Lague) showing the rules in motion.
- [I Didn't know Birds use Math in Murmurations! - Smarter Every Day 234](https://www.youtube.com/watch?v=4LWmRuB-uNU) — channel SmarterEveryDay, on murmuration behaviour.

## Lead image
- file_page: https://commons.wikimedia.org/wiki/File:Murmuration_11-2025.jpg
- image_url: https://upload.wikimedia.org/wikipedia/commons/thumb/7/7d/Murmuration_11-2025.jpg/1280px-Murmuration_11-2025.jpg
- shows: a murmuration of starlings, several hundred birds forming a dense cloud
- author: Skander zarrad
- license: CC BY-SA 4.0
- license_url: https://creativecommons.org/licenses/by-sa/4.0

## Sources
- [Boids, Wikipedia](https://en.wikipedia.org/wiki/Boids)
- [Craig Reynolds, Boids](https://www.red3d.com/cwr/boids/)
- [Craig Reynolds, Steering Behaviors](https://www.red3d.com/cwr/steer/)
- [Robert Hodgin, Murmuration](https://roberthodgin.com/project/murmuration)
- [The seventh starling, The Contrarian](https://contrarian.ca/2012/06/20/the-seventh-starling/)
- [Coding Adventure: Boids (YouTube)](https://www.youtube.com/watch?v=bqtqltqcQhw)
- [Smarter Every Day 234 (YouTube)](https://www.youtube.com/watch?v=4LWmRuB-uNU)
- [Commons file page](https://commons.wikimedia.org/wiki/File:Murmuration_11-2025.jpg)
