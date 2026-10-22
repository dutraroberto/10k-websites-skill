# ffmpeg Recipes

Exact commands for every processing step. These parameters are proven defaults; the keyframe interval in particular is the difference between smooth and stuttering scrub.

## Folder discipline

First, the project folder itself: create a named folder for the site (the brand's name works well) at a location agreed with the user, with `index.html` and `assets/` inside, and tell the user where it lives. Raw generations and review copies live OUTSIDE that deploy folder (for example in `review/` beside the project folder, or one level up). Only processed files go into `assets/`. Nothing in the deploy folder that should not ship, ever. Files the user brings back from ChatGPT and Flow land in `review/incoming/` (the handoff is in `prompt-handoff.md`).

## The 16:9 crop (every ChatGPT still that becomes a video frame)

ChatGPT usually returns landscape images at 3:2. Flow wants 16:9, and a frame Flow crops by itself is a frame you did not compose. Crop it yourself, centered, before inspection and before it goes on a video card:

```
ffmpeg -i review/incoming/start-frame.png -vf "crop=iw:trunc(iw*9/16/2)*2" review/incoming/start-frame-16x9.png
```

That keeps the full width and trims the top and bottom equally. If the subject sits high or low, shift the crop with a y offset (`crop=iw:trunc(iw*9/16/2)*2:0:Y`, where Y is the pixels to drop from the top). Check the result: subject intact, negative space where the captions will live. The `-16x9` file is the one that goes to Flow, never the raw download. Supporting stills for the lower sections skip this crop; they are sized for their sections instead.

## The scrub encode (the one that matters most)

Re-encode the approved raw video with a short keyframe interval, or scrubbing will stutter because the browser can only seek precisely to keyframes:

```
ffmpeg -i raw.mp4 -c:v libx264 -crf 18 -preset slow -g 8 -keyint_min 8 -pix_fmt yuv420p -movflags +faststart -an assets/hero-scrub.mp4
```

- `-g 8 -keyint_min 8`: a keyframe every 8 frames, so every scroll position seeks cleanly.
- `-crf 18`: visually clean quality.
- `-movflags +faststart`: the metadata moves to the front so playback and Blob use start immediately.
- `-an`: strip audio; a scrub video never needs it, and Veo always delivers some.
- Target roughly 4 to 8 MB for a 6-second 1080p clip (5 to 10 MB for an 8-second manual-path clip). If it lands far above that, raise `-crf` toward 20 to 22 and re-check quality.

### The compression fork by footage type

Footage splits into two families, and they compress in opposite ways:

**Busy detail masks artifacts.** Footage that fills the frame with busy detail (particles, full-frame fabric or texture) hides compression artifacts inside the detail, so crf can push hard. Busy full-frame detail tolerates crf 25 to 26 with a downscale to about 1700px wide (`-vf scale=1728:-2`); starting points, not targets. Full-frame texture is the densest footage of all and may need a narrower width or a higher crf still.

**Smooth gradients are the opposite.** They band instead of masking (banding is visible stair-steps across what should be a smooth color ramp). Push crf hard on a gradient-heavy clip and the calm frames break first, so check the calm gradient frames specifically.

**The method is always the same:** start from the starting points above, step ONE variable at a time (crf first, then width), and eyeball the worst frames by scrubbing, not by pausing. Artifacts that survive a freeze-frame vanish in motion on busy footage. Budget size roughly in proportion to duration: a 6-second clip wants roughly a third the size of an 18-second master. Add a downscale before ever sacrificing keyframe density, and keep every other scrub flag identical (`-preset slow -g 8 -keyint_min 8 -pix_fmt yuv420p -movflags +faststart -an`).

## The tail trim (when the gate feedback is "end it earlier")

The cheapest ending fix is not a re-roll. When a shot is strong until the subject drifts back into motion near the end, cut the raw at the last steady frame. The page maps scroll to progress, not to seconds, so a shorter clip costs nothing anywhere else. Extract candidate frames around the target second, pick the one that rests best (margins, face, composition), then trim and scrub-encode in one pass:

```
ffmpeg -i raw.mp4 -t 4.3 -c:v libx264 -crf 18 -preset slow -g 8 -keyint_min 8 -pix_fmt yuv420p -movflags +faststart -an assets/hero-scrub.mp4
```

Re-derive the poster and ending frame afterward, and re-verify the new ending with the header mocked over it, because the trimmed frame is now the page's resting composition. Only reach for a full-price re-roll when no steady frame exists at all.

## The ending-rest check (objective, one command)

Whether the ending truly rests does not have to be a judgment call. Measure the motion per frame: difference each frame against the previous one and read the average brightness of the difference, which is a motion curve:

```
ffmpeg -i raw.mp4 -vf "tblend=all_mode=difference,signalstats,metadata=print:key=lavfi.signalstats.YAVG" -f null -
```

Each frame prints a `YAVG` line; higher means more changed since the last frame. Read the tail of the curve: an arrival rises and falls back near its starting level; a drift stays high to the end. One command replaces the eyeball, and it decides whether the tail trim above is needed at all.

## Poster (first frame) and ending frame

```
ffmpeg -i assets/hero-scrub.mp4 -frames:v 1 -q:v 2 assets/hero-poster.jpg
ffmpeg -sseof -0.1 -i assets/hero-scrub.mp4 -update 1 -frames:v 1 -q:v 2 assets/hero-ending.jpg
```

The ending frame is a free, perfectly on-brand design asset. Reuse it in a lower section.

## Frame extraction for inspection (before the video gate)

Pull start, middle, and end frames from the RAW video and look at them yourself: anatomy, transitions, whether the ending truly rests. For a 6-second clip (read the real duration first with `ffprobe -v error -show_entries format=duration -of csv=p=0 raw.mp4` and take the middle; for an 8-second manual-path clip, `-ss 4`):

```
ffmpeg -ss 0 -i raw.mp4 -frames:v 1 -q:v 2 review/frame-start.jpg
ffmpeg -ss 3 -i raw.mp4 -frames:v 1 -q:v 2 review/frame-mid.jpg
ffmpeg -sseof -0.1 -i raw.mp4 -update 1 -frames:v 1 -q:v 2 review/frame-end.jpg
```

## The chaining frame grab (full quality, never review quality)

The frame that becomes the next segment's start frame must be a full-quality PNG. The `-q:v 2` review jpgs above are for your eyes only; chaining from one bakes compression into every later segment:

```
ffmpeg -sseof -0.1 -i seg1-raw.mp4 -update 1 -frames:v 1 review/incoming/seg2-start.png
```

PNG is lossless, so this frame carries no compression. It goes on the next segment's card as the Flow start frame upload (the chaining recipe is in `prompt-laws.md`). It is already 16:9, because it came out of a 16:9 video, so it needs no crop.

## Segment concat (Tier 2 chained journeys)

**Preferred: concat the raws and encode exactly once.** Feed the RAW segments into a single filter and apply the scrub encode one time to the joined result. One encode means the joins cannot mismatch, because there is nothing to match:

```
ffmpeg -i seg1-raw.mp4 -i seg2-raw.mp4 -i seg3-raw.mp4 -filter_complex "[0:v][1:v][2:v]concat=n=3:v=1:a=0[v]" -map "[v]" -c:v libx264 -crf 25 -preset slow -g 8 -keyint_min 8 -pix_fmt yuv420p -movflags +faststart -an assets/hero-scrub.mp4
```

Set `-crf` from the compression fork above: 18 as the normal starting point, harder values (plus a downscale) for busy particle or texture footage, gentler for gradient-heavy footage. The `-crf 25` in the example shows the busy-footage value.

**Fallback when the raws are unavailable:** encode every segment with IDENTICAL parameters (the scrub encode above: same codec, resolution, frame rate, pixel format, `-g 8`), then concatenate losslessly with the concat demuxer:

`concat-list.txt`:
```
file 'seg1-scrub.mp4'
file 'seg2-scrub.mp4'
file 'seg3-scrub.mp4'
```

```
ffmpeg -f concat -safe 0 -i concat-list.txt -c copy assets/hero-scrub.mp4
```

If a demuxer join glitches on playback, the segments' parameters were not truly identical: re-run the scrub encode on each segment from its raw source with the exact same command, then concat again. Do not paper over a glitchy join by re-encoding the concatenated file at lower quality. And if you still have the raws, use the single-encode path instead; it makes this whole failure mode impossible.

## The crossfade rescue (when a chained join shows)

When an already-generated chain shows a visible cut at a join even though the motion continues (usually texture re-imagined at the seam; the seam law is in `prompt-laws.md`), replace the hard concat with a short crossfade per join using `xfade`. A quarter second is the proven starting point. A scrolling visitor reads it as the surface shifting, not as a cut.

The offset is the first input's length minus the fade length. Measure the real segment lengths with ffprobe first; generated clips are not always exactly 6.0 or 8.0 seconds. Two 6-second segments with a 0.25-second fade:

```
ffmpeg -i seg1-raw.mp4 -i seg2-raw.mp4 -filter_complex "[0:v][1:v]xfade=transition=fade:duration=0.25:offset=5.75[v]" -map "[v]" -c:v libx264 -crf 18 -preset slow -g 8 -keyint_min 8 -pix_fmt yuv420p -movflags +faststart -an assets/hero-scrub.mp4
```

For three segments, chain the xfades. Each later offset is the joined length so far minus the fade (6 + 6 - 0.25 - 0.25 = 11.5):

```
ffmpeg -i seg1-raw.mp4 -i seg2-raw.mp4 -i seg3-raw.mp4 -filter_complex "[0:v][1:v]xfade=transition=fade:duration=0.25:offset=5.75[a];[a][2:v]xfade=transition=fade:duration=0.25:offset=11.5[v]" -map "[v]" -c:v libx264 -crf 18 -preset slow -g 8 -keyint_min 8 -pix_fmt yuv420p -movflags +faststart -an assets/hero-scrub.mp4
```

For 8-second manual-path segments, the offsets become 7.75 and 15.5 (8 + 8 - 0.25 - 0.25).

Set `-crf` and any downscale from the compression fork above. This is still a single encode, so the joins cannot mismatch. Each fade trims the total by its length; the page never notices, because everything runs on progress, not seconds. Verify by scrubbing back and forth across each join, not by watching at speed. A scrub site is read at scrub speed, and that is where a seam either shows or disappears.

## Web still sizing

Resize supporting stills to about 1920px wide with ONE clean compression pass. Many hosts recompress images server-side, so upload large and clean and let the host's pass be the only lossy step (videos pass through hosts untouched):

```
ffmpeg -i raw-still.png -vf scale=1920:-2 -q:v 2 assets/section-name.jpg
```

`-q:v 2` is near-transparent JPEG quality. `-2` keeps the height even, which some encoders require. Screenshots and other sharp-edged UI imagery stay PNG or lossless WebP; the JPEG pass is for photographic stills.

## Verify after every encode

Play or frame-extract the output before using it. A silent encode failure caught now is free; caught after deploy it is a re-deploy.
