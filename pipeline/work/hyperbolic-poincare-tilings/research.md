# Hyperbolic Poincare Tilings — research

## Origin
The Poincare disk is a model of the hyperbolic plane in which the whole infinite plane is drawn inside a circle. Eugenio Beltrami found the disk and half-plane models in the late 1860s ([Dunham, Escher's use of the Poincare models](https://www.math-art.eu/Documents/pdfs/Dunham.pdf)). The art link comes from M. C. Escher. In 1957 he received a paper by H. S. M. Coxeter with a {6,4} hyperbolic triangle pattern, which gave him "quite a shock" ([Dunham](https://www.math-art.eu/Documents/pdfs/Dunham.pdf)). Grant Sanderson's Bridges 2024 talk retells how Escher's contact with mathematicians (Polya's 17 wallpaper symmetries, the 1954 Congress of Mathematicians, then Coxeter) led to the Circle Limit series ([Bridges 2024](https://archive.bridgesmathart.org/2024/bridges2024-4.html)). Escher made Circle Limit I in 1958 and II to IV over the next two years, drawing them by hand ([Dunham](https://www.math-art.eu/Documents/pdfs/Dunham.pdf)). The computer algorithm to fill the disk with a motif was first described by Douglas Dunham and first implemented by his PhD student Ajit Datar ([EscherSketch README](https://www.github.com/looeee/hyperbolic-tiling)).

## Key works and artists
- **Circle Limit III (1959)**, M. C. Escher: woodcut printed from five blocks; interlocking fish whose mouths meet in three-pointed stars, shrinking toward the rim ([National Gallery of Art](https://www.nga.gov/artworks/61280-circle-limit-iii)). Copyrighted: link only.
- **Hyperbolic Poincare tiling of triangles**, Matthew Cook (Caltech): Mathematica notebook that renders pictures and animations of a triangle tiling of the disk ([Wolfram Library](https://library.wolfram.com/infocenter/Demos/148)).
- **EscherSketch**, looeee: open-source WebGL (Three.js) prototype that maps two fish triangle images (half white, half black fish) onto hyperbolic triangles to build a two-coloured {p,q} tiling in the disk ([GitHub](https://www.github.com/looeee/hyperbolic-tiling)).
- **Poincare meets Mandelbrot**: Mandelbrot and Julia sets mapped into a {7,3}-based hyperbolic tiling ([westy31.nl](https://westy31.nl/PoincareMeetsMandelbrot/PoincareMeetsMandelbrot.html)). Shows how other fractal material can be carried by the tiling.
- **hypertiling**, University of Wuerzburg: Python library for regular hyperbolic lattices with millions of vertices, plotting and animation ([hyperweb](https://gitpages.physik.uni-wuerzburg.de/hypertiling/hyperweb/)).

## How it works
- Tilings are named {p,q}: regular p-gons, q meeting at each vertex. The plane is hyperbolic when (p-2)(q-2) > 4. Euclidean space fits only six triangles around a vertex; hyperbolic space allows seven, eight, 50 or infinitely many ([Scientific American](https://www.scientificamerican.com/blog/roots-of-unity/everything-looks-better-in-the-hyperbolic-plane/)).
- The disk is conformal: angles are preserved, so a motif keeps roughly its shape as it shrinks toward the rim. Equal hyperbolic distances become ever smaller on screen near the boundary, so all Circle Limit III fish are the same hyperbolic size ([Dunham](https://www.math-art.eu/Documents/pdfs/Dunham.pdf)).
- Hyperbolic "straight lines" are circle arcs meeting the boundary at right angles (diameters included) ([Dunham](https://www.math-art.eu/Documents/pdfs/Dunham.pdf)).
- Construction: reflect a triangle (angles pi/p, pi/q, pi/2) across its edges; each reflection is a circle inversion ([westy31.nl](https://westy31.nl/PoincareMeetsMandelbrot/PoincareMeetsMandelbrot.html)). A (2,3,7) triangle gives the {7,3} tiling ([AMS Visual Insight](https://blogs.ams.org/visualinsight/wp-json/wp/v2/posts/946)).
- Shader idea (design note, standard technique): per pixel, fold the point repeatedly by the inversions/reflections until it lands in the fundamental triangle, then colour by fold count and position. Moving the camera is a Moebius transform applied to the pixel before folding.
- Fast CPU construction: a combinatorial layer-by-layer method builds cells at nanoseconds per cell ([arXiv 2508.04765](https://arxiv.org/html/2508.04765v1)).

## Visual language
Endless repeating cells that shrink toward a hard circular rim; curved edges (arcs, not straight lines); strict radial symmetry around the centre with a sense of falling inward. Two-colour checker (fish style) or colour by fold depth gives rhythm. Dark outside the disk makes it read as an object or a portal. Escher's version adds figurative motifs and tile-sized stars at vertices.

## Live potential
- Kick: pulse of disk translation (a hyperbolic step toward the next vertex) or brightness flash on edges.
- Bass: slow rotation speed or edge thickness; hi-hats: fine line shimmer, colour cycling per fold depth.
- BPM: one tile step or one rotation of 2pi/p per bar, so the fly-through loops cleanly.
- Faders: p and q (tiling type, snap to valid hyperbolic pairs), translation direction, zoom, line width, palette.
- Layering: use the disk as a mask or texture source in Resolume (Spout); feed a video or camera into the tiles as the motif (each tile is a copy of the fundamental region); use the rim as a vignette.
- Boring risk: static centred disk; fix with continuous translation, tiling changes on drops and texture inside tiles.

## Variations
1. Video-in tiles: fold a live feed through the tiling instead of flat colours.
2. Fractal tiles: Mandelbrot or Julia content inside the cells, as in the westy31 images.
3. Tiling morph: step through {p,q} pairs on a MIDI fader with short crossfades.
4. Off-disk projection: map onto a dome or use the disk as a mask for a second layer.
5. Poincare half-plane or Klein variants for a flatter, brick-like fly-through.

## Facts
- {7,3} heptagon tiling has symmetry group (2,3,7); Klein's quartic can be tiled by 24 regular heptagons ([AMS Visual Insight](https://blogs.ams.org/visualinsight/wp-json/wp/v2/posts/946)).
- Escher drew the hyperbolic patterns by hand, because motif sizes vary continuously ([Dunham](https://www.math-art.eu/Documents/pdfs/Dunham.pdf)).
- Heptagon counts in the westy31 construction follow Fibonacci numbers by generation: 1 + 7(F(2N+1) - 1) ([westy31.nl](https://westy31.nl/PoincareMeetsMandelbrot/PoincareMeetsMandelbrot.html)).
- Polygons in EscherSketch get heavily stretched above about 8 sides or vertex count ([GitHub](https://www.github.com/looeee/hyperbolic-tiling)).
- Circle Limit III uses five blocks and about five colours in the National Gallery description ([NGA](https://www.nga.gov/artworks/61280-circle-limit-iii)).

## Visual references
- [Circle Limit III, NGA](https://www.nga.gov/artworks/61280-circle-limit-iii) — Escher's woodcut (copyrighted, link only).
- [EscherSketch](https://www.github.com/looeee/hyperbolic-tiling) — WebGL fish tiling in the Poincare disk.
- [Poincare meets Mandelbrot](https://westy31.nl/PoincareMeetsMandelbrot/PoincareMeetsMandelbrot.html) — fractal content in a {7,3}-based tiling.
- [Cook, Poincare tiling](https://library.wolfram.com/infocenter/Demos/148) — triangle tiling pictures and animations (notebook download).
- [hypertiling](https://gitpages.physik.uni-wuerzburg.de/hypertiling/hyperweb/) — lattice plots and animation in the disk.

## Lead image
- kind: free
- image_url: https://blogs.ams.org/visualinsight/files/2014/06/73_tiling.png
- page_url: https://blogs.ams.org/visualinsight/2014/07/15/73-tiling/
- author: Anton Sherwood
- license: Public domain (the AMS Visual Insight page states he put it in the public domain)
- shows: {7,3} tiling of the hyperbolic plane by equal regular heptagons, three meeting at each vertex

## Quotation images
- kind: quotation
- image_url: https://mcescher.com/wp-content/uploads/2019/05/LW-434.jpg
- page_url: https://mcescher.com/gallery/mathematical/
- work: Circle Limit III (LW 434), M.C. Escher, December 1959, woodcut
- rights holder: The M.C. Escher Company B.V. (site footer: all rights reserved, no reproduction without written permission)
- note: no free-licence image could be verified (Commons not reachable). Free candidate to check later: Anton Sherwood's {7,3} image, which Visual Insight says is public domain on Wikimedia Commons ([AMS Visual Insight](https://blogs.ams.org/visualinsight/wp-json/wp/v2/posts/946)).

## Sources
- [Bridges 2024, Sanderson](https://archive.bridgesmathart.org/2024/bridges2024-4.html)
- [Dunham PDF](https://www.math-art.eu/Documents/pdfs/Dunham.pdf)
- [EscherSketch](https://www.github.com/looeee/hyperbolic-tiling)
- [NGA Circle Limit III](https://www.nga.gov/artworks/61280-circle-limit-iii)
- [AMS Visual Insight](https://blogs.ams.org/visualinsight/wp-json/wp/v2/posts/946)
- [Scientific American](https://www.scientificamerican.com/blog/roots-of-unity/everything-looks-better-in-the-hyperbolic-plane/)
- [westy31.nl](https://westy31.nl/PoincareMeetsMandelbrot/PoincareMeetsMandelbrot.html)
- [hypertiling](https://gitpages.physik.uni-wuerzburg.de/hypertiling/hyperweb/)
- [arXiv 2508.04765](https://arxiv.org/html/2508.04765v1)
- [Wolfram Library](https://library.wolfram.com/infocenter/Demos/148)
