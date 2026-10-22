# The Hero-Video Laws and Prompt Construction

Design every shot by these laws BEFORE generating. They predict which concepts land first-try and which burn retries. Each one came from a real build that either succeeded because of it or failed without it.

## The twelve laws

1. **The motion agrees with the scroll.** Scrolling down must read as going down, opening up, or arriving: a pour, a descent, a teardown, an approach. Ask of every concept: "when the visitor scrolls down, does this motion feel like down?" A subject that flies UP while the visitor scrolls down fights the page and always loses.

2. **One subject, one continuous motion, no cuts.** A single subject travels through a single journey. Do not ask AI video to turn one thing into a different thing: transformations between two subjects are the hardest shot in AI video and burn money. If the user insists on one, hide the swap inside a flash or cloud moment, keep one unbroken trajectory (same heading, same position, same speed on both sides of the swap), and expect retries.

3. **Lock the path, free the body.** The trajectory stays rigid, but the subject on it must stay alive: natural movement, ripple, small adjustments. The scene needs life too: drifting steam, streaking cloud, shifting light. Never stabilize a shot by freezing the subject. A frozen subject on a clean path reads as dead footage.

4. **Plan the ending first.** The final frame is where the page comes to rest, so write it into the prompt explicitly as a composed, satisfying arrival: the cup settled on the counter, the product assembled, the destination reached. An awkward ending makes an awkward website. If the ending is a product showcase, compose it with generous margin above and below the whole product: the site's header sits over the top of the frame, and cover-cropping eats the edges on wider or shorter screens. A product with its top cut off reads as an accident; squeezed against the nav it reads as busy. The alternative that dodges the problem entirely: a full-bleed texture ending with nothing croppable is text-safe on every screen. Verify by viewing the ending frame with the header mocked over it, at a wide window and a short one, before approving.

5. **Choose forgiving subjects.** Fluids, mist, steam, light, fabric, and distant silhouettes render beautifully. Anything whose exact anatomy every viewer knows up close (controllers, keyboards, hands, familiar animals in detail) will show its errors instantly. Keep risky detail distant, simple, and moving.

6. **Prefer a vertical motion axis.** A straight up-and-down journey matches the scroll axis one to one, so the reveal moves with the page in both directions. Not mandatory, but when two concepts are equal, take the vertical one.

7. **Compose for the layout.** Decide during concept design where the action sits in frame, and leave intentional negative space for the page's captions and story to live in. The action lane stays clear; the words flank it.

8. **Sell the boundary crossings.** When the camera passes through a surface (into water, through mist, past glass), write the physical lens moment into the prompt: a splash, droplets on the lens, a beat of blur. A clean pass through a boundary reads as fake; the mess is the realism.

9. **If the hero features a product, brand it or frame it close.** A generic unbranded object at distance reads as a placeholder. Either apply the brand mark via image editing before animating, or write the ending to land close enough that the object's design carries it.

10. **Text over footage earns its legibility.** Live video behind type is a moving background you do not control frame to frame. So every text band gets a legibility system: a local scrim (a soft dark gradient behind the words) that deepens only while that band is active, a real text shadow, and a contrast check against the WORST frame of that band, never the average. Place each band in the calmest region of its frames. If a line cannot be read at a glance over the busiest moment of its band, it fails. The working system, with code and the audit, is in `scrub-pipeline.md`.

11. **Pace text in scroll distance, not seconds.** A scroll site is read in flicks, not played at 24 frames per second. Give every caption beat a long fully-visible plateau (most of its band, enough to survive several normal scroll flicks) with short eased ramps at the edges, so a reader never sees text pop in and vanish between two flicks and never has to stop dead to catch a line. Test the beat map by flick-scrolling like a real visitor, not by slow dragging. The exact numbers and the flick test are in `scrub-pipeline.md`.

12. **The standing guards:** write "no text, no logos, no lettering anywhere" into every image and video prompt (ChatGPT in particular loves to add captions, signage, and labels). Write every prompt as one finished shot description, never a question or a list of options, so neither tool improvises its own idea of the shot. And never build the site around footage the user has not approved: silent scaffolding during a render wait is fine, but nothing gets shown or finished until the video passes its gate.

## Prompt construction

### Start frame template (ChatGPT image, asked as 16:9, cropped to 16:9 on arrival)

Compose the image as frame one of the motion: the subject positioned so the journey can begin. ChatGPT usually returns 3:2, and the crop to 16:9 trims the top and bottom, so keep the subject and the action inside the vertical middle of the frame.

```
[SUBJECT] at [POSITION IN FRAME], composed as the first moment of a motion
that will [ONE-SENTENCE JOURNEY]. [LIGHTING: source, direction, mood].
[PALETTE: the three to five brand colors described as materials and light,
not hex codes]. [ATMOSPHERE: the ambient life the scene carries]. Intentional
negative space at [WHERE THE CAPTIONS WILL LIVE]. Cinematic, photorealistic,
wide landscape 16:9 frame, subject kept in the vertical middle. No text,
no logos, no lettering anywhere.
```

**The negative-space phrasing trap: never name empty space as darkness or emptiness.** When you reserve room for captions, describe the scene as one continuous world filling the frame edge to edge, with the calm region as part of that world: soft shadow, receding depth, a plain surface. Ask for "generous empty darkness left and right" and the model paints literal black side panels, which costs a re-roll. Edge-to-edge phrasing lands first try.

The same trap has a symmetry case. When the composition needs a centered subject, "centered" alone is not enough. Say the subject bisects the frame, dead center, the same distance from the left edge as from the right. Describe both halves as one identical treatment, and explicitly ban objects, machine parts, and bright highlights on either side. A real build took three attempts before this phrasing landed the shot.

If the user supplied a real product photo, that photo can be the start frame instead. Inspect it for resolution and composition first, and confirm the negative space works for the layout. When it needs the brand's world around it, write an edit card: the user attaches the photo in ChatGPT and the prompt asks to keep the product exactly as it is and rebuild only the scene, light, and framing around it.

When the subject is a real person (the owner, the chef, the maker), their photo rides in as a reference image instead of a start frame: the card has the user attach the photo in ChatGPT and asks for a new scene with that same person, restating their recognizable details in the prompt (hair, glasses, clothing), and inspect the result for likeness the same way you inspect for trademarks. Likeness is a brand-coherence detail; a near-miss face fails the whole site. And one hard rule before any face is generated: only use a photo of the user themselves or of a person who has agreed to appear on the site. If the photo is of anyone else, stop and ask before generating.

### Video template (Google Flow, Frames to Video, Veo, 16:9, about 8 seconds)

```
One continuous shot, no cuts. [SUBJECT] [VERB OF THE JOURNEY: pours, descends,
approaches, assembles] from [START STATE] to [END STATE] along [THE EXPLICIT
TRAJECTORY: straight down the center of frame, a slow forward push, etc].
The [SUBJECT] stays alive throughout: [SMALL NATURAL MOTION: ripple, sway,
micro-adjustments]. The scene stays alive: [AMBIENT LIFE: drifting steam,
shifting light, streaking cloud]. [IF A BOUNDARY IS CROSSED: the physical
lens moment, e.g. a splash and droplets on the lens with a beat of blur].
The shot ends at rest: [THE COMPOSED FINAL FRAME, fully described: what sits
where, what the light does, why it feels arrived]. No text or lettering
anywhere. No music, no dialogue.
```

Download at 1080p when Flow offers it, never anything larger. The web version gets re-encoded and compressed anyway.

Veo sometimes adds a cut or a second camera angle when a prompt is vague about continuity. "One continuous shot, no cuts" stays first in every video prompt for that reason, and a camera instruction (a slow push in, a locked overhead descent) beats leaving the camera to the model.

## The chaining recipe (Tier 2: a 15 to 20 second scroll journey)

One long journey built from short segments that join invisibly (6 seconds each on the Higgsfield path, about 8 on the manual path):

1. Generate segment 1 from the start frame. Run the full inspection and ⛔ VIDEO GATE on it alone.
2. Extract the final frame of the approved segment as a full-quality PNG with ffmpeg (exact command in `ffmpeg-recipes.md`; review-grade jpgs are not good enough to chain from).
3. Use that PNG as the next segment's start frame. On the Higgsfield path, pass it yourself with `--start-image` (see `higgsfield-path.md`). On the manual path, hand it to the user as the start frame of the next segment's card. Save it as `review/incoming/seg2-start.png` (and so on), give its full path on the card, and have them upload it in Flow's Frames to Video start slot. Never let them screenshot or re-save it; the PNG goes in exactly as extracted. Flow's own Extend feature also continues a clip, but stay with the extracted frame: it keeps every segment on its own gate and its own retry.
4. Write the next segment's prompt so the motion CONTINUES: same heading, same speed, same lighting, picking up exactly where the previous segment rested. The join is invisible only if the motion vector never breaks. Special case that comes up often: when a segment ends in a near-empty or near-black frame (say, a single point of light), the next segment's prompt must explicitly describe what grows out of that frame. Do that and the join disappears.
5. Gate each segment separately. A rejected segment is a single re-roll, not a redo of the whole journey.
6. Join the approved segments into one file with the single-encode concat in `ffmpeg-recipes.md`: feed the RAW segments into one filter and encode exactly once with the scrub settings. One encode means the joins cannot mismatch. The fallback, when raws are unavailable, is to encode every segment with IDENTICAL settings and join with the concat demuxer; identical parameters are what keep that path invisible at the joins, and mismatched ones glitch at every join.
7. The joined file is the one scrub video the page uses. The page never knows it was segments.

Only the final segment needs the composed resting ending (law 4). Middle segments should end mid-motion so the next one can continue it.

**The seam law: texture identity does not carry over.** Each generation re-imagines fine texture from its start frame. Position carries over; the exact weave, grain, or skin does not. So a rest-to-rest join on hyper-specific texture shows as a visible cut even when the motion vector is perfect. Storyboard every seam to land inside motion, or inside a moment that motivates a texture refresh: a sweep across the lens, a blur beat, a moment of darkness, a shift of light. Never butt two rest states together on specific texture. If a chain is already generated and a seam shows anyway, the rescue is the crossfade join in `ffmpeg-recipes.md`.

**Which worlds chain reliably:** abstract worlds (pure light, particles, atmosphere) are the chaining reliability champions. With no anatomy for a continuation prompt to get wrong, a three-segment chain can land first-try on every segment. When a Tier 2 concept is on the fence, this is a strong reason to go abstract.

## When a tool improvises

ChatGPT sometimes "improves" a prompt into its own idea: a different angle, added props, a caption across the image. Flow sometimes adds a cut or drifts from the start frame. Do not accept the improvisation by default; your designed shot obeys the laws and composes for your layout. Hand over a corrected card that names the exact deviation to avoid ("the camera stays locked overhead, no cut, no second angle"). The exception is a gift, which inspection in Phase 6 keeps.

## The video model choice

On the Higgsfield path, the choice runs on real preflighted prices; it lives in `higgsfield-path.md`. The rest of this section is the manual path.

Flow usually offers two kinds of Veo generation: a top-quality mode and a faster, cheaper one. Each generation spends part of the user's monthly Google allowance, and the cheaper mode spends noticeably less. Both are genuinely good. Present the honest tradeoff before the first video card: the top-quality mode buys the highest ceiling for the final hero; the cheaper mode turns a tight allowance from one shot into several, which makes the video gate a normal creative decision instead of a frightening one. A common, sensible pattern: explore in the cheaper mode, then generate the approved prompt once more in top quality. Their allowance, their choice, made before anything is spent.

Never quote a price or an allowance from memory. Ask the user what Flow shows on their account, or have them send a screenshot, and work from that. For a chained journey, say up front how many video generations the whole chain needs, plus one expected retry, so they can check it against what they have.
