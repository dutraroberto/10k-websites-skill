# The Manual Path: ChatGPT for Images, Google for Video

On this path you do not generate media yourself (the other path, where you do, is `higgsfield-path.md`). You write every prompt; the user pastes it into ChatGPT (images) or Google Flow (video), downloads the result, and hands the file back. You inspect it, gate it, and build with it. The user's only job is copy, paste, click generate, download, drop the file back. Everything else stays yours: the shot design, the wording, the settings, the inspection, the processing.

## The two tools

- **Images: ChatGPT** (chatgpt.com or the app). Its built-in image generation makes every start frame, ending frame, and supporting still, and it edits images too (applying a logo, fixing a sneaked-in trademark, removing an object). It accepts uploaded images as references, which is how a real product photo, a logo, or the owner's face rides in.
- **Video: Google Flow** (labs.google/flow), Google's filmmaking tool running the Veo video models. Its Frames to Video mode takes a start frame (and optionally an end frame) plus a prompt, which is exactly what this pipeline needs. The Gemini app (gemini.google.com) can also turn a photo into a video and is the fallback when Flow is unavailable to the user, with less control. Video generation needs a Google plan that includes Veo (the Google AI plans); check what the user's account offers instead of trusting a plan name written here.

Never offer any other image or video generator, even when one is connected on the machine. The user picked this path; the only switch is to the Higgsfield path, and only when they ask for it.

## The prompt card (the format of every handoff)

Every generation request reaches the user as one prompt card, in a single message, nothing else mixed in. The words around the prompt are in the user's language; the prompt itself is written in English, because both tools follow English prompts most faithfully.

```
### [Card name, e.g. "Starting frame" or "Segment 2 video"]
**Where:** ChatGPT, new chat   |   Google Flow, Frames to Video
**Settings:** [only what they must set: aspect ratio 16:9, model, mode, start frame file to upload, end frame file if used]
**Attach:** [files to upload with the prompt, with exact paths, or "nothing"]

<the prompt in its own code block, so one click copies it>

**Save as:** [exact file name] → drag it into this chat, or save it to [exact folder path]
```

Rules for the card:

- **One card per generation, one message per card**, unless several stills are independent (supporting stills can go out as a numbered set in one message, each its own card).
- **The prompt sits alone in its code block.** No comments, no brackets left unfilled, no "optional" lines. What is in the block is exactly what gets pasted.
- **Name the file you want back.** A fixed name (`start-frame.png`, `seg1-raw.mp4`) means you always know which file is which.
- **The incoming folder** is `review/incoming/` beside the project folder, outside the deploy folder (folder discipline in `ffmpeg-recipes.md`). Create it before the first card and give the user its full path. Dragging the file into the chat works too; when they do, copy it into that folder yourself.
- **Close every card with a clickable check-in:** "Done, it's in the folder" (Recommended), plus "It came out wrong" for when the tool refused or produced something broken. Then wait. A "done" is the signal to look for the file and inspect it, never the inspection itself.

## ChatGPT, step by step (what to tell the user the first time)

1. Open a NEW chat for each image job, so earlier images do not bleed into the style. Supporting stills for the same site can share one chat on purpose, since that chat already holds the world.
2. Attach any file the card lists (product photo, logo, reference face) with the paperclip, then paste the prompt and send.
3. Download the image at full size (open it, then the download button) and save it with the card's file name.

ChatGPT tends to make landscape images at 3:2, not 16:9. That is fine: ask for 16:9 in the prompt anyway, and when the file arrives, crop it to 16:9 yourself with the recipe in `ffmpeg-recipes.md`. Because a crop trims top and bottom, every start-frame prompt keeps the subject and the action well inside the vertical middle of the frame.

ChatGPT occasionally refuses or softens a prompt. Rewrite it in plain commercial product-photography words (name the subject early, describe objects rather than sensations) and hand over a new card.

## Google Flow, step by step (what to tell the user the first time)

1. Open labs.google/flow, start a new project, and pick Frames to Video.
2. Set the aspect ratio to 16:9 (landscape) and choose the model the card names.
3. Upload the start frame the card names (always the processed 16:9 file you prepared, never the raw ChatGPT download). If the card lists an end frame too, add it in the end slot.
4. Paste the prompt and generate. Flow often makes more than one take per prompt: have the user download every take and save them as `seg1-raw-a.mp4`, `seg1-raw-b.mp4`, and so on. You inspect them all and recommend the best; extra takes are free choice, not extra cost.
5. Download at the highest resolution Flow offers (1080p when the option is there).

**Modes and models.** Flow usually offers a faster, cheaper Veo mode and a top-quality mode, and each generation spends part of the account's monthly allowance. Before the first video, have the user tell you what Flow shows them (or send a screenshot) so you present real options, not remembered ones.

**Audio.** Veo makes sound with the video. The site never uses it, and the scrub encode strips it, so the prompt only says "no music, no dialogue" to keep the model's attention on the picture.

**Clip length.** A Veo clip is about 8 seconds, longer than the Higgsfield path's 6. The recipes in `ffmpeg-recipes.md` and `scrub-pipeline.md` give the 8-second values beside the 6-second ones; always read the real duration with ffprobe and use the matching values.

## The end frame option (law 4 made nearly free)

Frames to Video can take an end frame as well as a start frame, and so can the Higgsfield default video model (`--end-image`), so this option works on both paths. When the composed resting frame matters most (a product showcase, a settled cup, an exact arrival), generate the ending frame as a second ChatGPT still in the same chat as the start frame, crop both to 16:9, and hand both to Flow. The model then travels between two frames you already approved, which makes law 4 nearly free. Keep it optional: an end frame can make the motion feel steered, so use it when the ending's composition matters more than the journey's looseness, and inspect for a mushy middle.

## Receiving files

- **Images:** read the file to look at it, then run the inspections from Phase 6 (trademarks, anatomy, brand coherence, composition for the layout). Crop to 16:9 and show the user the processed version in chat.
- **Videos:** extract start, middle, and end frames with ffmpeg and look at them before saying anything. Then the ⛔ VIDEO GATE as written in Phase 6.
- **Wrong file or missing file:** list the incoming folder, say plainly what you see, and ask for the right one. Never build on a guess about which file is which.
