# Tritone Red, White and Black — design

## Look

Any video that goes in comes out as a screen-printed poster: big flat fields of black, one red and white, no greys, no gradients, no texture inside a field. Faces turn into stencils, hands into silhouettes with a red mid-tone band, smoke and feedback into hard-edged blobs that change shape all the time. The borders between the three fields are the drawing. They crawl and flicker where film grain or sensor noise crosses a threshold. On the beat the picture snaps: the red band jumps from the mid-tones to the shadows or highlights, black and white swap, or the red disappears and the image becomes a stark black and white print with a red accent that comes back on the kick. The mood is Constructivist poster and record sleeve: El Lissitzky's *Beat the Whites with the Red Wedge* (three flat colours and one diagonal), Kraftwerk's *The Man-Machine*, Barbara Kruger's crushed photographs with white-on-red captions (see research.md). Black is the dominant colour, red is the accent, and white is the light.

Design rule: **exactly three output colours, decided last.** The quantiser is the final step before the output, so no blur, bloom, scaling or compression can mix new pinks and dark reds into the picture afterwards (the only exception is the deliberate `edge` softness).

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Sources | any clips: faces, hands, type, geometry, feedback, or a Spout/NDI feed from UE or TouchDesigner | Normal / whatever the mix needs | 100 % | high contrast and lots of motion work best; flat, low-contrast footage turns to mush |
| 2 | Wedge / circle mask (optional) | rotating Lissitzky wedge or circle, white on black, drawn in its own layer or generator | used as a matte, not visible | — | decides where red is allowed (variation 2) |
| 3 | Pre-filter | luma, blur 0–3 px, crush curve (gain around a pivot) | inside the tritone effect | — | steadies the edges and piles the histogram up at the ends |
| 4 | Quantiser | 3 bands (low / mid / high) mapped to black / red / white, with soft edges and hysteresis | inside the tritone effect, or the LUT effect | 100 % | **last** effect on the composition or the master group |
| 5 | Keyline (optional) | 1–2 px black outline on band borders | inside the tritone effect | — | screen-print trap line, the `outline` toggle |
| 6 | Kruger caption (optional) | white bold condensed type on a red bar (same red hex as the quantiser) | Normal, **above** the quantiser | 100 % | sits above so its red stays exact; text from a MIDI-keyed list |
| 7 | Output | composition at the wall's native pixel size, no scaling after layer 4 | — | — | see Performance tips |

Layers 3–5 are one effect: a single shader pass, plus an optional previous-frame buffer for hysteresis.

## Techniques

**1. Luma and crush**
1. Luma on the encoded (display) values: `Y = dot(rgb, (0.2126, 0.7152, 0.0722))` (Rec.709 weights). Linear-light accuracy does not matter here; what you see on screen is what gets thresholded.
2. Optional gamma: `Y = pow(Y, gamma)`, gamma 0.5–2.0, which moves the mid-tones without moving black or white.
3. Crush: `Y = clamp((Y - pivot) * crush + pivot, 0, 1)` with pivot 0.5 and crush 1–4. With hard edges crush hardly matters; it matters with soft edges and for how much of the frame clips to pure black and white.

**2. Three-band quantiser (the core)**
1. Two thresholds, `A < B`. Enforce `B >= A + 0.02` so the red band never vanishes by accident (use a separate `red off` control for that).
2. Band weights with edge softness `s`: `wMid = smoothstep(A - s, A + s, Y)`, `wHigh = smoothstep(B - s, B + s, Y)`.
3. Colour: `c = mix(cLow, cMid, wMid)`, then `c = mix(c, cHigh, wHigh)`. With `s = 0` (use `step`) you get exactly three colours. With `s > 0` you get thin blended seams (dark red between black and red, pink between red and white), which read as ink spread.
4. Why it works: three flat fields make the eye read shape instead of texture, and the threshold borders become contour lines that animate on their own.

**3. Colour assignment (the beat flips)**
1. Colours live in three slots, `K = black`, `R = red`, `W = white`. Two controls decide which band gets which slot.
2. `red_pos` (3 steps): 0 = red in the shadows (low = R, mid = K, high = W), 1 = red in the mid-tones (K, R, W, the classic poster), 2 = red in the highlights (K, W, R, the Sin City spot-colour reading).
3. `invert` (toggle) swaps K and W in whatever order `red_pos` gives. 3 × 2 = all 6 permutations, but each control is simple enough to hit on a pad.
4. `red off` (toggle or beat gate) replaces R with whichever of K or W is next to it in luma: `red_pos` 0 → K, 2 → W, 1 → split the mid band at `(A + B) / 2`. The result is pure black and white, so "red only on the kick" (variation 1) is just this gate driven by the kick.

**4. Hysteresis against chatter (optional)**
1. Noise near a threshold makes pixels flip every frame. Sometimes you want that (it is the strobing contour), sometimes it looks like broken video.
2. Keep last frame's band index per pixel (0, 1, 2). A pixel only changes band when `Y` passes the threshold by more than `hold` (0–0.05) in the direction of the change: thresholds move up by `hold` for pixels that are currently below them and down by `hold` for pixels that are currently above.
3. `hold = 0` gives raw flicker; 0.02 calms 8-bit camera noise; 0.05 makes the image "sticky" and posterised in time.

**5. Wedge mask (optional, variation 2)**
1. A second input `mask` (0..1, luminance).
2. Red is allowed only where `mask > 0.5`. Outside, the red band falls back as in `red off`. Rotate the wedge 1 turn per 4 or 8 bars, or snap it 45° per beat.

**6. Keyline (optional)**
1. Band index `b = (Y > A) + (Y > B)`.
2. If the band index changes between neighbouring pixels (compare with the pixel to the right and the pixel below, or use `fwidth(b) > 0`), draw `cLow`, normally black, with a width of 1–2 px. This gives a silkscreen trap line that holds the shapes together on very noisy sources.

## Shaders

Render order: P0 → P1. P1 reads its own previous output for hysteresis (ping-pong). Every pass runs at the output (wall) resolution, and P0 may run at half resolution.

**P0 — pre-filter (optional)**
- Purpose: get rid of noise before thresholding, so edges move with the picture, not with the sensor.
- Inputs: source texture; `blur` (0–3 px).
- Per pixel: 5-tap or 9-tap Gaussian (or one Kawase step) of radius `blur`, then store luma only (one channel is enough).
- Output: R8 or R16F luma texture, at full or half resolution. Half resolution is fine because P1 samples it bilinearly, and that is also cheap smoothing.

**P1 — tritone quantiser**
- Purpose: the whole look.
- Inputs: P0 output (or the source directly when P0 is off), previous P1 band buffer (if `hold > 0`), optional `mask` texture; uniforms `low_cut` (A), `high_cut` (B), `crush`, `gamma`, `edge` (s), `hold`, `red_pos` (int 0–2), `invert` (bool), `red_off` (bool or 0..1), `red_hue` (degrees), `outline` (bool), and the three slot colours.
- Per pixel:
  1. `Y` = luma (from P0, or computed here), then gamma and crush as in Technique 1.
  2. Read the previous band index; shift A and B by ±`hold` as in Technique 4.
  3. Band weights `wMid`, `wHigh` with softness `edge`.
  4. Build the band colours `cLow`, `cMid`, `cHigh` from the slots using `red_pos` and `invert`; apply `red_off` and the mask fallback.
  5. Mix the colours. If `outline` is on and the band changes in the neighbourhood, output `cLow`.
  6. Write the colour (alpha 1) and, in a second target or persistent buffer, the band index for the next frame.
- Output: RGBA8 at the wall resolution. Nothing is drawn after it except the caption layer.

The red slot is computed once on the CPU (or per frame in the shader, it is cheap): start from the base red in HSV and rotate the hue by `red_hue`, keeping S and V fixed (see Color scheme).

## Color scheme

| role | hex | notes |
|------|-----|-------|
| shadow / base (K) | `#000000` | true black; LED walls do it perfectly, so don't lift it |
| accent (R), default | `#E4160C` | poster red, warm and slightly toward vermilion; reads as "Lissitzky red" on LED |
| highlight (W) | `#F2EFE8` | warm paper white at about 94 % instead of `#FFFFFF`, so white fields don't blind people and the red stays the hero |
| alt red: vermilion | `#FF3B12` | `red_hue` about +8° |
| alt red: crimson | `#B5001F` | `red_hue` about −12°, also darker; use it in breaks |
| alt red: hot pink | `#FF1F6B` | `red_hue` about −25°, for a Kruger/pop moment, not for a whole set |
| caption bar | same as R | always linked to the live red so caption and image match |

- Mapping: hard three-stop gradient map on luma (see Techniques). No cosine palette, no ramps.
- Over time: black and white are locked; only the red moves. Palette drift (variation 3) is a hue walk of ±10° over 32 bars. Larger hue jumps are cuts on a phrase change, never fades.
- With the music: the red's *position* (`red_pos`) and its *presence* (`red off` gate) are the musical controls; its hue is a mood control.
- LED contrast: keep red ≤ 35 % of the frame on average (raise `high_cut` or lower `low_cut` if it is heavier). Full-field white flashes at 94 % are already very bright on a big wall. If the processor has a global brightness setting, run the wall at 60–80 % and keep the content at full range.

## Blending and compositing

- **Position in the chain:** the tritone effect goes last on the composition (or on a master group that contains every source layer), so every clip beneath shares one three-colour world. This is the "unify unrelated clips" trick from research.md.
- **Between sources:** mix sources freely beneath (Add, Screen, Multiply, Difference all work). The quantiser only sees the luma of the mix. Difference and Multiply are especially good because they give hard dark shapes that the threshold turns into stencils.
- **Mask:** the wedge layer is a matte that goes into the effect's mask input (ISF/Wire/UE). In native Resolume, use the two-group method (see Resolume recipe).
- **Feedback:** if you use feedback, put it *beneath* the quantiser, never after it. Feedback of a three-colour image through a blur creates new in-between colours. Feedback beneath, quantised afterwards, gives great hard-edged trails.
- **Post chain after the quantiser:** none. No bloom, glow, chromatic aberration, grain or sharpening, since all of them create colours outside the palette. If you want grain, add noise *before* P1 (it turns into contour flicker, which is the on-brand version of grain).
- **Caption layer:** Normal blend, 100 %, above the quantiser, red bar in exactly the live R hex, type in W.

## Motion and timing

The effect itself does not move. All motion comes from the source and from discrete switches. Treat it like a drum machine for colour.

- **Per beat (1/4):** optional `red off` gate: red shows for 1/8 note on every kick, otherwise black and white only (variation 1).
- **Per bar:** `red_pos` steps 1 → 0 → 1 → 2 (mids, shadows, mids, highlights), or a BeatStep Pro drum-track pattern that fires the three positions on chosen steps.
- **Per phrase (8 or 16 bars):** `invert` flips once; hue cut to the next red; wedge mask on/off.
- **Thresholds:** hand moves on faders at 0.5–4 s, or a kick pulse that pushes both thresholds up by +0.06 to +0.10 (more black, the image "ducks"), attack 5 ms, release 120–200 ms.
- **Build-up:** narrow the red band slowly over 8–16 bars (`high_cut` → `low_cut` + 0.03), `hold` → 0 for maximum flicker.
- **Drop:** on beat 1, snap to wide red (A = 0.18, B = 0.82) for one beat, then back to the poster default, and start the per-bar `red_pos` sequence.
- **Break:** `red off` latched (pure black and white), `edge` up to 0.04, `hold` 0.04, slow hue drift toward crimson; the red returns on the first kick after the break.

## Parameters

| name | range | default | what it does on stage | suggested MIDI / BPM mapping |
|------|-------|---------|-----------------------|------------------------------|
| `low_cut` (A) | 0.00–0.95 | 0.30 | where black ends; up = more black, a silhouette look | KeyStep 37 knob or Resolume fader; kick pulse +0.08 |
| `high_cut` (B) | A+0.02–1.00 | 0.62 | where white starts; together with A it sets how much red there is | second knob; hi-hat jitter ±0.02 |
| `red_hue` | −40° to +40° | 0° (`#E4160C`) | shifts the accent between crimson, red, vermilion and orange; pink at −25° | knob, or slow LFO 1 cycle / 32 bars, ±10° |
| `edge` | 0.00–0.10 (luma) | 0.01 | hard poster cut versus soft ink-spread seams | knob; 0 on the drop, 0.03–0.05 in breaks |
| `red_pos` + `invert` | 0 / 1 / 2, toggle | 1, off | the beat flip: which band is red, and black/white swap | BeatStep Pro drum track: three pads → positions, a fourth pad → invert; clocked to the set's BPM |
| `crush` | 1.0–4.0 | 1.8 | pre-contrast; with soft edges it decides how much clips to the ends | knob, rarely touched |

Set-and-forget: `hold` 0.02, `blur` 1 px, `gamma` 1.0, `outline` off, `red off` on a momentary pad (hold = black and white only).

## Build recipes

### Resolume Arena

**Path A — native, zero custom code: a 3-colour LUT.** The LUT effect loads `.cube` files only and can sit on a clip, layer, group or the composition, with opacity and blend mode ([Resolume: LUT](https://resolume.com/support/de/lut)).

LUT recipe (any script or tool that writes a `.cube` text file can do this; the format is a header plus one `R G B` line per grid node, red index changing fastest and blue slowest ([colour-science: Iridas/Adobe .cube module](https://colour.readthedocs.io/en/develop/_modules/colour/io/luts/iridas_cube.html))):

1. It must be a **3D LUT** (`LUT_3D_SIZE`). A 1D LUT maps R, G and B independently, so it cannot turn one luma value into a coloured output. For example, a mid-grey pixel cannot come out red because its green and blue would have to go to zero while its red goes up, and a 1D curve per channel cannot tell grey from green.
2. Grid size: 33 (35,937 lines) is standard; use 65 (274,625 lines) for crisper edges. Trilinear interpolation between nodes always softens the edge by about one grid step (1/32 ≈ 0.03 luma at size 33, ≈ 0.016 at 65). That softness is the LUT's built-in `edge`.
3. For each node (r, g, b) with values `i / (N - 1)`: compute `Y` with Rec.709 weights, apply crush (pivot 0.5, gain 1.8), then output by band:

| band (on crushed Y) | output R G B (0–1) | colour |
|---------------------|--------------------|--------|
| Y < A (0.30) | 0.000 0.000 0.000 | black `#000000` |
| A ≤ Y < B (0.62) | 0.894 0.086 0.047 | red `#E4160C` |
| Y ≥ B | 0.949 0.937 0.910 | white `#F2EFE8` |

4. Keep `B − A` at least 3 grid steps (≥ 0.1 at size 33), otherwise interpolation across a black→white cell creates grey.
5. Build a small **LUT bank** with the same thresholds: `tri_mid_red` (K R W), `tri_low_red` (R K W), `tri_high_red` (K W R), `tri_mid_red_inv` (W R K), `bw_only` (threshold at (A+B)/2, black/white), plus hue variants of `tri_mid_red` in crimson, vermilion and pink. That makes 8 files.

Chain on the composition (or on a master group), top to bottom in the effect list:
1. **Blur**, 1–2 px (optional; steadies the edges).
2. **Brightness** and **Contrast** (names as in the Arena 7 effect browser; check your version). These are your two live threshold controls, because the LUT's thresholds are fixed: Brightness shifts both thresholds together (up = more white), and Contrast spreads or narrows the red band around mid-grey (up = less red in source terms, because the source range is stretched across fixed thresholds).
3. **LUT** = `tri_mid_red.cube`, opacity 100 %, blend Normal. Below 100 % opacity the original video shows through and the three-colour promise is broken. Use that only on purpose, as a transition.
4. Beat flips: stack three more **LUT** effects (low, high, inverted) below the first and leave them bypassed. Map each bypass to a BeatStep Pro pad. Only one is active at a time; the last active one wins because it receives a three-colour image and remaps it again, so keep exactly one on. Alternatively, put the bank in a Wire patch (Path B), which is cleaner.

Wedge mask natively: duplicate the source group. Group 1 ends with the LUT `bw_only`. Group 2 (above) ends with `tri_mid_red` and has the wedge as its mask (for example a wedge clip in a layer with the Mask effect, channel Luminance). Red then shows only inside the wedge.

Audio: Resolume can drive Brightness from the audio FFT (kick band, see Audio reactivity). Add the Kruger caption as a Text Block or text clip in a layer above the master group.

**Path B — custom effect (where an ISF/Wire effect fits):** replace steps 2–4 with one Wire effect (below) placed in the same slot, last on the composition. This gives true independent A/B thresholds, edge, hue and beat flips on five dashboard faders.

### Resolume Wire

Build an effect patch (exported to Arena as an effect). Two variants, both built around nodes from the Resolume docs:

**B1 — LUT-bank patch** (no shader code):
- Effect texture input → **LUT** node (applies a LUT to a texture) → effect output.
- **LUT 3D Resource** node loaded with the LUT bank as a collection ([Resolume: LUT](https://resolume.com/support/de/lut)) → an index selects the LUT → LUT node's LUT input.
- Expose: `lut index` (int 0–7, mapped to BeatStep pads or stepped by a BPM-synced counter: beat counter → modulo 3 → index), plus pre-LUT brightness/contrast as two floats.

**B2 — ISF patch** (full control): Wire runs ISF shaders through the **ISF** node, with inputs declared in the shader's JSON block (float with MIN/MAX, bool, color, event) and shown on the node ([Resolume: ISF](https://resolume.com/support/en/isf)).
- Effect texture input → ISF node (P1 from Shaders; fold P0 into it as a 5-tap blur) → effect output.
- Optional second image input on the ISF node ← the wedge mask (a Wire shape generator or a second texture input).
- ISF inputs: `low_cut`, `high_cut`, `edge`, `crush`, `gamma`, `hold` (floats); `red_hue` (float −40..40); `red_pos` (float 0–2, rounded in the shader, or long dropdown); `invert`, `red_off`, `outline` (bool); `redColor`, `whiteColor` (color).
- Hysteresis needs P1's previous frame (an ISF persistent buffer pass). The Resolume ISF article doesn't say whether multipass or persistent buffers run in Wire. Test it; if they don't work, set `hold = 0` and raise the blur to 2 px instead.
- Beat logic in Wire: BPM/beat node → counter → modulo 3 → `red_pos`; a bar-phase compare drives `red_off` as a gate (on for the first 1/8 of each beat).
- Expose to the Arena dashboard: `low_cut`, `high_cut`, `red_hue`, `edge`, `red_pos`, `invert`, `red off`.

### Unreal Engine 5.7

A **Post Process Material** on a Post Process Volume (Infinite Extent = on), or on the CineCamera's post-process settings ([Epic: Post Process Materials](https://dev.epicgames.com/documentation/unreal-engine/post-process-materials-in-unreal-engine)).

Material setup:
- Material Domain = Post Process; Blendable Location = **After Tonemapping** (thresholds then act on display values, the same as in Resolume). Output only Emissive Color.
- Node chain: `SceneTexture: PostProcessInput0` (Color) → `Dot` with constant3 (0.2126, 0.7152, 0.0722) → `Power` (gamma) → `Subtract 0.5 → Multiply crush → Add 0.5 → Saturate` → two `SmoothStep` nodes (min = A − edge, max = A + edge; and the same with B) → two `Lerp` nodes (cLow → cMid by wMid, then → cHigh by wHigh) → Emissive.
- Band colours: three `Lerp`/`Switch`-style selects driven by scalar `RedPos` (0/1/2) and `Invert` (0/1); build them with `If` nodes or by lerping between the slot colours with `RedPos` rounded. Red slot = Vector Parameter `RedColor` (computed hue on the Blueprint side) or a `HueShift` node on a base red driven by `RedHue / 360`.
- Wedge mask: a texture parameter or a second `SceneTexture`-independent UV mask (a rotated half-plane from `ScreenPosition`, angle = scalar `WedgeAngle`) multiplied into "red allowed".
- Skip hysteresis in UE (it would need a render target ping-pong). Instead use 1–2 px blur: sample PostProcessInput0 at 4 offsets of `ViewSize`-scaled texel size and average them before the Dot.

Scene settings, so the thresholds are stable: Exposure → Metering Mode Manual (no auto exposure), Bloom intensity 0, Lens Flares off, Film Grain 0, Vignette 0, Motion Blur 0 or low. All of these would otherwise move the luma under your thresholds.

Remote Control (OSC/MIDI):
- Put the parameters in a **Material Parameter Collection** `MPC_Tritone` (LowCut, HighCut, Edge, Crush, RedHue, RedPos, Invert, RedOff, WedgeAngle) and read them in the material with `CollectionParameter` nodes.
- A small Blueprint actor `BP_TritoneControl` with float/bool variables marked as exposed. On change (or on Tick) it writes them to the MPC. Expose those variables in a Remote Control Preset, then bind OSC addresses (e.g. `/tritone/low`, `/tritone/high`, `/tritone/hue`, `/tritone/edge`, `/tritone/redpos`) and MIDI CCs/notes from the KeyStep/BeatStep in the Remote Control protocol panel.
- Output: send the UE render to Resolume over Spout (e.g. the Off World Live Spout plugin) or NDI (the NDI SDK for Unreal). Decide where the quantiser lives: either in UE (send finished three-colour frames) **or** in Resolume (send the raw render), not both. If you go through NDI, quantise *after* receiving (see Performance tips).

### Godot

Godot 4, 2D:
- Source: `VideoStreamPlayer`, a `SubViewport` with the 3D scene, or a camera feed, shown in a `TextureRect` that fills the screen.
- Effect: a `ColorRect` on top (full rect) with a `canvas_item` shader that reads the screen through `uniform sampler2D screen_tex : hint_screen_texture, filter_linear` and runs P1.
- Hysteresis: render the tritone pass into a `SubViewport` (Clear Mode = Never, Update Mode = Always), and pass that viewport's `ViewportTexture` from the previous frame back in as `prev_band`. Store the band index in the red channel of a separate small viewport if you want the visible output to stay clean. Skip this if not needed.
- Uniforms with hints: `low_cut` and `high_cut` (`hint_range(0.0, 1.0)`), `edge` (`hint_range(0.0, 0.1)`), `crush` (1–4), `red_hue` (−40..40), `red_pos` (int 0–2), `invert`, `red_off`, `outline` (bool), `red_color` and `white_color` (`source_color`), `mask_tex` (sampler2D).
- Control: set uniforms from GDScript via `material.set_shader_parameter()`, driven by `InputEventMIDI` (Godot has built-in MIDI input via `OS.open_midi_inputs()`) and an OSC addon if needed.

## Audio reactivity

| band | drives | amount | smoothing | threshold / gate |
|------|--------|--------|-----------|------------------|
| kick, 40–120 Hz | `low_cut` + `high_cut` offset (both up = image ducks toward black) | +0.06 to +0.10 | attack 5 ms, release 150 ms | trigger above −18 dBFS band level, 80 ms re-arm |
| kick (alternative) | `red off` gate opens (red appears) | on/off | none (hard gate), hold 1/8 note | same trigger |
| snare / clap, 1–4 kHz transient | `invert` flip | toggle | — | onset detector, max one flip per beat |
| hi-hats, 6–12 kHz | `high_cut` jitter | ±0.02 | attack 2 ms, release 60 ms | — keep it small, or the white fields shimmer |
| long-term RMS (2–4 s window) | `gamma` | 0.8 (loud) … 1.3 (quiet) | attack 1 s, release 3 s | — slowly darkens quiet passages |

Discrete flips (`red_pos`, `invert`) should be tempo-locked (BeatStep Pro sequencer on MIDI clock) rather than audio-triggered. Transients can double-trigger, and a wrong colour flip is very visible.

## Performance tips

- **Cost:** negligible. P1 is a handful of math per pixel plus 1–5 texture reads. Even at 7680 × 2160 it is well under 1 ms on a mid-range GPU. The LUT effect is one 3D texture lookup. Most of the cost is in your sources.
- **Quantise at the final resolution.** Any scaling after the quantiser (Advanced Output slice scaling, LED processor scaling, a downscaling NDI sender) blends the three colours into pinks and dark reds along every edge. Make the composition match the wall's pixel map 1:1, or apply the effect per output slice.
- **NDI and codecs:** NDI and most video codecs subsample chroma, which smears red/white and red/black edges into fringes. Send the raw (un-quantised) image over NDI and quantise on the receiving machine. Spout on the same machine is lossless and fine.
- **Flicker on big walls:** at audience distance, 1 px chatter on a 2.5 mm pitch wall looks like sparkle, and on a coarse 6–10 mm pitch wall it looks like broken pixels. On coarse walls use `hold` 0.02–0.04 and blur 2 px, or turn on `outline`.
- **Heavy red:** large red fields on LED walls bleed into neighbouring pixels in camera (filming the show) and can moiré. Keep red ≤ 35 % of the area and avoid 1 px red lines on coarse walls.
- **Odd aspect ratios:** no problem for the quantiser itself. Wedge mask angles should be computed in pixel space (aspect-corrected), otherwise the wedge squashes on 3:1 or 6:1 walls.
- **Low-contrast sources:** a fixed threshold on flat footage gives mush. Keep a "contrast-safe" set of sources (faces with side light, hands on black, type, geometry, feedback), and use `crush` plus Contrast before the LUT for everything else.

## Further references

- [Resolume: LUT](https://resolume.com/support/de/lut) — the LUT effect in Arena/Avenue (only `.cube`, opacity, blend mode, clip to composition) and the Wire LUT nodes (LUT 3D In, LUT 3D Resource with collections, LUT).
- [Resolume: ISF in Wire](https://resolume.com/support/en/isf) — how the ISF and ISF Resource nodes load shaders and expose inputs from the JSON header.
- [ISF Spec (GitHub)](https://github.com/mrRay/ISF_Spec) — the Interactive Shader Format spec: input types, multi-pass and persistent buffers, which you need for the hysteresis pass.
- [Epic: Post Process Materials in Unreal Engine](https://dev.epicgames.com/documentation/unreal-engine/post-process-materials-in-unreal-engine) — material domain, blendable location, `SceneTexture: PostProcessInput0`.
- [colour-science: .cube reader/writer source](https://colour.readthedocs.io/en/develop/_modules/colour/io/luts/iridas_cube.html) — readable reference for the `.cube` header keywords (`TITLE`, `LUT_3D_SIZE`, `DOMAIN_MIN/MAX`) and data order (red fastest), based on Adobe's Cube LUT Specification.
