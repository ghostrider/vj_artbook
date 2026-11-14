# Point Cloud Scan Worlds — research

## Origin
A point cloud is a set of points in 3D space, each with X, Y, Z and often RGB colour, normals or timestamps. It is produced by 3D scanners (lidar, structured light) or by photogrammetry software that reconstructs points from overlapping photos ([Wikipedia: Point cloud](https://en.wikipedia.org/wiki/Point_cloud)). Clouds are normally an intermediate step towards meshes, but rendering the raw points is an aesthetic of its own.

The pop-culture moment is Radiohead's "House of Cards" (released on YouTube 19 July 2008). Director James Frost first pitched his lidar/GeoVideo work in November 2007. Tests followed on 24 March 2008, and the shoot began on 3 May 2008 in West Palm Beach, Florida ([James Frost case study](https://www.jamesfrost.tv/case-studies/radiohead)). Exteriors and the party scene were captured by a Velodyne scanner with 64 rotating lasers, run by Rick Yodar of Velodyne. Thom Yorke's close-ups used Geometric Informatics' structured-light "GeoVideo" system, which had been developed for medical use ([same source](https://www.jamesfrost.tv/case-studies/radiohead)). Part of the data was released as open source so the public could make their own versions ([same source](https://www.jamesfrost.tv/case-studies/radiohead)).

The newest chapter is 3D Gaussian Splatting (Kerbl, Kopanas, Leimkühler, Drettakis; SIGGRAPH 2023, arXiv 8 Aug 2023). It starts from the sparse points of camera calibration and optimises anisotropic 3D Gaussians, rendering at 30 fps or more at 1080p ([arXiv 2308.04079](https://arxiv.org/abs/2308.04079)).

## Key works and artists
- **Radiohead, "House of Cards" (2008)**, directed by James Frost, with no conventional cameras. Lidar and structured-light point clouds, rendered as glowing dots. See the [James Frost case study](https://www.jamesfrost.tv/case-studies/radiohead).
- **Andy Shauf, "Clove Cigarette" (2020)**, directed by Colin Medley and Jared Raab, with Luca Tarantini as technical director. Leica lidar plus iPhone 4K video were turned into photogrammetry point clouds in Metashape, then assembled in Unreal Engine as one continuous camera move through a world of about half a billion points. Unreal handled about 20 million points on screen ([No Film School](https://nofilmschool.com/music-video-lidar-point-clouds-unreal-engine)). This is the closest precedent to the Unreal workflow Uli already uses.
- **A$AP Rocky, "Helicopter" (2026)**, directed by Dan Strait. Nearly all performances were captured volumetrically with a 56-camera RGB-D array and rendered as dynamic Gaussian splats, relit and simulated in Houdini, with CG Nomads GSOPs in post. About 30 minutes of footage was exported as roughly 1 TB of PLY sequences ([Radiancefields.com](https://radiancefields.com/a-ap-rocky-releases-helicopter-music-video-featuring-gaussian-splatting)).
- **Synth Splat** by Lorie Chen, Steph Ng and Nicholas Pudjarminta. It is a browser visualiser where Gaussian splat worlds crossfade between tracks, and real-time audio analysis (Meyda.js) drives shaders and lighting. Rendering uses Spark ([World Labs showcase](https://www.worldlabs.ai/labs/showcase/synth-splat)).

## How it works
- **Lidar** fires lasers and records distance per return, giving XYZ points. The Velodyne used for "House of Cards" spun 64 lasers ([James Frost](https://www.jamesfrost.tv/case-studies/radiohead)).
- **Photogrammetry** matches features across many photos, solves camera positions, and triangulates a sparse, then dense, set of coloured points ([Wikipedia](https://en.wikipedia.org/wiki/Point_cloud)).
- **Gaussian splats** replace points with soft, oriented, semi-transparent ellipsoids that are optimised so the render matches the source photos. Starting from the sparse calibration points, training interleaves optimisation with density control ([arXiv](https://arxiv.org/abs/2308.04079)). A splat file is still a list of points with extra attributes, so it can be treated as a particle set.
- **Rendering raw points** is cheap: one GPU instance, sprite or point per entry. Point size, colour and position are free parameters, which is what makes it a good live-visual primitive.

## Visual language
- Glittering dust in the shape of a real place: dense where surfaces face the scanner, empty where it could not see. The gaps (scan shadows, holes behind objects) are part of the look.
- Colour either from photos (RGB) or from false-colour lidar intensity or range. The Ouster lead image colours points by intensity times range, from dark blue to orange ([Commons file](https://commons.wikimedia.org/wiki/File:Ouster_OS1-64_lidar_point_cloud_of_intersection_of_Folsom_and_Dore_St,_San_Francisco.png)).
- Motion: slow fly-through (parallax is the whole effect), orbit around a scanned person or object, slicing planes sweeping through the volume, and scan-line rings on lidar.
- Rhythm comes from dissolving and reassembling: GSOPs describes scenes that dissolve into a point cloud and reassemble via particle conversion ([Radiancefields.com on GSOPs](https://radiancefields.com/gsops-brings-native-gaussian-splatting-to-touchdesigner)).

## Live potential
- **Kick:** point size pulse, or a radial shockwave that displaces points outward from a centre and springs back.
- **Bass:** low-frequency noise displacement amplitude; global scale breathing.
- **Hi-hats:** sparkle by randomly toggling or brightening a small fraction of points; flicker in alpha; thin or thicken the cloud.
- **BPM / MIDI:** tempo-synced slicing plane sweeping through the scene; a fader for "dissolve amount" (from solid scan to dust); a fader for point size; a fader to blend between two scans (GSOPs supports blending multiple splats, globally or by texture, in TouchDesigner, [source](https://radiancefields.com/gsops-brings-native-gaussian-splatting-to-touchdesigner)). Camera path keyed to bar counts.
- **Layering:** output via Spout/NDI into Resolume. Point clouds on black layer well with additive blend and give strong depth for feedback, scan-line or colour-grade effects on top.
- **Tools that fit the setup:** Unreal Niagara handled about 20 million visible points in the Clove Cigarette production ([No Film School](https://nofilmschool.com/music-video-lidar-point-clouds-unreal-engine)). In TouchDesigner, Derivative's palette component gsLoader loads .spz and .ply splats as points in a POP ([Derivative gsLoader](https://derivative.ca/UserGuide/Palette:gsLoader)). GSOPs (beta, Lake Heckaman) needs TouchDesigner 2025.32820 or later and has a free non-commercial Lite tier ([Radiancefields.com](https://radiancefields.com/gsops-brings-native-gaussian-splatting-to-touchdesigner)). A TouchDesigner tutorial covers training in PostShot and showing the result as point clouds and splats ([Derivative tutorial 05, ETCH-01](https://derivative.ca/community-post/tutorial/touchdesigner-tutorial-05-3d-gaussian-splatting-trainingrefiningmanipulating)).
- **Boring on a big screen when:** the camera drifts at constant speed through an untouched scan, points are all one size, or one scan is held for minutes. Heavy point counts also cost frame rate; thin the cloud for stage use (GSOPs has a Thin Splat operator for this).

## Variations
1. **Dust-to-solid:** a dissolve fader moves every point from its scan position to a noise cloud and back, so the room appears only on the drop.
2. **Slicing planes:** a thin plane sweeps through the scan on each bar and only points within the slab are lit, giving a medical-scan or lidar-sweep look.
3. **Two-place morph:** blend two scans (a club and a forest) point by point, driven by a fader.
4. **Scan the venue:** capture the Schaffhausen venue the afternoon before and play it back as dust during the set, with the crowd inside the scan.
5. **False-colour lidar mode:** recolour any RGB cloud by depth or intensity (the Radiohead and Ouster look) and switch colour modes on hi-hats.

## Facts
- "House of Cards" used no cameras, and the crew drove the lidar bus at about 10 mph for clean scans ([James Frost](https://www.jamesfrost.tv/case-studies/radiohead)).
- "House of Cards" won a D&AD Pencil and was included in the V&A's "Decode" exhibition ([James Frost](https://www.jamesfrost.tv/case-studies/radiohead)).
- The Clove Cigarette team chose Unreal over Cinema 4D partly because the X-Particles plugin would have cost about $1,000, while Unreal was free; the final world held about half a billion points ([No Film School](https://nofilmschool.com/music-video-lidar-point-clouds-unreal-engine)).
- The "Helicopter" splat sequences came to about 1 TB of PLY files ([Radiancefields.com](https://radiancefields.com/a-ap-rocky-releases-helicopter-music-video-featuring-gaussian-splatting)).
- Gaussian-splat files can be exported as SPZ (described as about 90% smaller) or PLY from Niantic's capture tools, which include Scaniverse data ([Niantic Spatial](https://www.nianticspatial.com/products/capture)).
- Kyle McDonald's home structured-light scanning guide, which needs only a camera and a projector, was published as a DIY route to a similar look ([Gizmodo](https://gizmodo.com/5437326/recreate-an-extraordinary-radiohead-video-with-ordinary-tech-and-this-guide)).

## How to source scans legally
- **Your own scans:** photograph or film a place and run photogrammetry or splat training (the TouchDesigner tutorial above uses PostShot from video). Niantic's Scaniverse/Capture pipeline exports splats as SPZ or PLY ([Niantic Spatial](https://www.nianticspatial.com/products/capture)). Mind private property, people's faces, and venue rules before scanning.
- **CC0 libraries:** Poly Haven assets are CC0: any purpose including commercial, no credit required ([Poly Haven licence](https://polyhaven.com/license)). Poly Haven's scanned models are meshes, which can be sampled into point clouds.
- **Radiohead data:** released as open source for remixing per the [James Frost case study](https://www.jamesfrost.tv/case-studies/radiohead); check its current licence before any commercial use.
- Always check the licence on each asset; do not assume a scan is free to use because it is public.

## Visual references
- [Radiohead, "House of Cards" (official video)](https://www.youtube.com/watch?v=8nTFjVm9sTQ) — the original lidar and structured-light point-cloud look, official Radiohead channel (title confirmed via oEmbed).
- [Synth Splat, World Labs showcase](https://www.worldlabs.ai/labs/showcase/synth-splat) — audio-reactive Gaussian splat worlds crossfading with a DJ set.
- [GSOPs article](https://radiancefields.com/gsops-brings-native-gaussian-splatting-to-touchdesigner) — splats dissolving into physics particles and back in TouchDesigner.
- [No Film School: Clove Cigarette breakdown](https://nofilmschool.com/music-video-lidar-point-clouds-unreal-engine) — behind the scenes of a lidar and photogrammetry point-cloud world in Unreal.
- [A$AP Rocky "Helicopter" write-up](https://radiancefields.com/a-ap-rocky-releases-helicopter-music-video-featuring-gaussian-splatting) — volumetric performance rendered as Gaussian splats.

## Lead image
- file_page: https://commons.wikimedia.org/wiki/File:Ouster_OS1-64_lidar_point_cloud_of_intersection_of_Folsom_and_Dore_St,_San_Francisco.png
- image_url: https://upload.wikimedia.org/wikipedia/commons/thumb/b/bd/Ouster_OS1-64_lidar_point_cloud_of_intersection_of_Folsom_and_Dore_St%2C_San_Francisco.png/1280px-Ouster_OS1-64_lidar_point_cloud_of_intersection_of_Folsom_and_Dore_St%2C_San_Francisco.png
- shows: orthographic view of a registered lidar point cloud (about 23 million points over 18 s) of a San Francisco intersection, coloured by intensity times range from dark blue to orange
- author: Daniel L. Lu (Commons user Dllu)
- license: CC BY 4.0
- license_url: https://creativecommons.org/licenses/by/4.0

## Sources
- [Wikipedia: Point cloud](https://en.wikipedia.org/wiki/Point_cloud)
- [James Frost: Radiohead case study](https://www.jamesfrost.tv/case-studies/radiohead)
- [arXiv 2308.04079, 3D Gaussian Splatting](https://arxiv.org/abs/2308.04079)
- [No Film School: Clove Cigarette](https://nofilmschool.com/music-video-lidar-point-clouds-unreal-engine)
- [Radiancefields.com: Helicopter](https://radiancefields.com/a-ap-rocky-releases-helicopter-music-video-featuring-gaussian-splatting)
- [Radiancefields.com: GSOPs](https://radiancefields.com/gsops-brings-native-gaussian-splatting-to-touchdesigner)
- [World Labs: Synth Splat](https://www.worldlabs.ai/labs/showcase/synth-splat)
- [Derivative: gsLoader](https://derivative.ca/UserGuide/Palette:gsLoader)
- [Derivative tutorial 05](https://derivative.ca/community-post/tutorial/touchdesigner-tutorial-05-3d-gaussian-splatting-trainingrefiningmanipulating)
- [Niantic Spatial: Capture](https://www.nianticspatial.com/products/capture)
- [Poly Haven licence](https://polyhaven.com/license)
- [Gizmodo: recreate the Radiohead video](https://gizmodo.com/5437326/recreate-an-extraordinary-radiohead-video-with-ordinary-tech-and-this-guide)
- [Commons file page (lead image)](https://commons.wikimedia.org/wiki/File:Ouster_OS1-64_lidar_point_cloud_of_intersection_of_Folsom_and_Dore_St,_San_Francisco.png)
