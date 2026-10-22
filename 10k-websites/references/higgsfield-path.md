# The Higgsfield Path: You Generate Directly

On this path you run every generation yourself through the Higgsfield CLI (`higgsfield`, also `hf`), from the terminal, spending the user's Higgsfield credits. The user never leaves the chat: they approve prices and judge results, you do the rest. The laws in `prompt-laws.md` and every gate in Phase 6 apply exactly as on the manual path; only the hands that press generate change.

Never route the build through Higgsfield's own website builder (the higgsfield-websites skill or `higgsfield website …`). This path borrows Higgsfield's image and video models only.

## Setup and the health check

- **Installed?** `higgsfield --version`. If missing: `npm i -g @higgsfield/cli` (needs Node.js, which Phase 1 installs anyway), then check the version again.
- **Signed in?** `higgsfield account status` shows the email, the plan, and the credits. If it says not authenticated or session expired, the user runs the sign-in: start `higgsfield auth login` yourself in the background; it opens their browser, where they sign in once and approve. If the browser does not open, the command prints a local sign-in file path to open by hand; pass that path to the user. Then wait with a clickable check-in ("Signed in, check it") and re-run `higgsfield account status`. Real credits on screen is the verification.
- **"No workspace selected"** is normal right after the first sign-in: every command, even `account status`, fails until a workspace is active. Run `higgsfield workspace list`. With one workspace (usually "Private", the personal account), select it yourself with `higgsfield workspace set <id>`. With several, ask which one pays (clickable, one option per workspace, the personal one first). Then re-run `higgsfield account status`.

## The commands this build uses

```
higgsfield model list --image --json          # current image models and their ids
higgsfield model list --video --json          # current video models and their ids
higgsfield model get <model_id> --json        # the params one model accepts
higgsfield generate cost <model_id> [same flags as create]   # exact price, free
higgsfield generate create <model_id> --prompt "..." [flags] --wait --json
higgsfield account status                     # credits left
```

- **Always `--wait`**: the command blocks until the job finishes and prints the result. Video can take minutes; pass `--wait-timeout 20m` for video so a slow render is not cut off.
- **Media flags take local paths**: `--start-image ./review/incoming/start-frame-16x9.png` uploads the file by itself. No manual upload step, no ids to juggle. `--end-image` works the same on models that accept it. `--image` passes a reference image (a product photo, a logo, the owner's face).
- **Download every result** from the URL in the output into `review/incoming/` with the same file names the manual path uses (`start-frame.png`, `seg1-raw.mp4`, and so on), so Phase 7 and every recipe stay identical: `curl -L -o review/incoming/seg1-raw.mp4 "<result-url>"`.
- **Never paste raw JSON, job ids, or URLs dumps into the chat.** Show the user the image itself, or the saved video's path.

## Models

The model lineup changes. Before the first generation of a build, list the current models and confirm every id below with `model get`, and never pass a param the schema does not list.

- **Starting frame and stills:** GPT Image 2.5 (`gpt_image_2_5`) is the default: `--aspect_ratio 16:9 --resolution 2k --quality high`. Its schema defaults to `quality low`, so always pass the quality explicitly. For a moodier cinematic still, Soul Cinema (`soul_cinematic`) is the alternative. A frame that arrives at exactly 16:9 needs no crop; check the real size and crop only if it is not 16:9 (recipe in `ffmpeg-recipes.md`).
- **Video:** Seedance 2.5 (`seedance_2_5`) is the default: `--mode omni_reference --start-image <frame> --duration 6 --resolution 1080p --aspect_ratio 16:9 --generate_audio false`, plus `--end-image <frame>` when the end frame option is used (see `prompt-handoff.md`; it works the same here). Its schema defaults to audio on and 720p, so always pass both explicitly; audio is never used (the scrub encode strips it) and only adds cost. Alternatives worth pricing side by side: Veo 3.1 (`veo3_1`), Kling 3.0 (`kling3_0`, start and end frames), and the cheap pair Kling 3.0 Turbo (`kling3_0_turbo`) and Veo 3.1 Lite (`veo3_1_lite`). On every model, turn audio off when the schema has a switch for it.
- **6 seconds** is the proven default of the original pipeline, and it costs about a quarter less than 8 (price is roughly proportional to duration). Every recipe in `ffmpeg-recipes.md` and `scrub-pipeline.md` gives the 6-second values first. If a model only offers other lengths (some cheap models fix theirs), pick the closest to 6, read the real duration with ffprobe, and adjust.

## Cost preflighting (free, always first)

`higgsfield generate cost` with the exact flags you plan returns the real price without spending anything. Use it before every generation and tell the user the price in plain words before it moves: the starting frame's price before the frame ("The starting image costs about N credits, making it now"), then the video model menu with real prices once the frame is approved.

**The video model is a real choice, and the user makes it.** Price the SAME planned shot (same frame, prompt, duration, resolution) on the default and on two alternatives, and present each price, one line on what it buys, and the credits left from `account status`. The honest tradeoff: the default buys the highest ceiling; a cheaper model turns a small balance from one shot into several (the spread has measured about ten to one between the default at 1080p and the cheap pair), which makes the video gate a normal creative decision instead of a frightening one. Fold the supporting stills into the same total so one yes covers the path from here to the build. For a chained journey, price the whole chain per segment and present the total up front.

## Chaining on this path

The chaining recipe in `prompt-laws.md` holds unchanged; only step 3 differs. Instead of handing the extracted PNG to the user, pass it yourself: `--start-image review/incoming/seg2-start.png` on the next segment's `generate create`. Each segment still gets its own inspection and its own ⛔ VIDEO GATE before the next one is generated.

## When credits run out

Say it plainly with the real number from `account status`. Two honest options, as a clickable question: top up the Higgsfield account (they buy credits on higgsfield.ai and you re-check the balance), or switch the remaining generations to the manual path (ChatGPT and Google Flow, `prompt-handoff.md`). Approved assets stay; nothing is redone.
