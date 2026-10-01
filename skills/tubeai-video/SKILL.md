---
name: tubeai-video
description: Assisted-animation and editing workflow for YouTube channel videos, built on Remotion. Sets up or resumes the project (Remotion, yt-dlp, headless Chromium, GPU renders, local voice and transcription), keeps each channel's branding and reusable animations consistent, finds video ideas and drafts scripts with the user (with YouTube data from the optional TubeAI connector), matches existing animation styles from a YouTube link or video file, researches media (news articles, X/Reddit posts, YouTube clips), generates voiceovers with Qwen3-TTS, edits a creator's raw recording (transcribes it, cuts it against the script and times the inserts to the words), and delivers the edit as a timeline for Premiere Pro (XML), Final Cut Pro (FCPXML) and DaVinci Resolve (OTIO). Assumes a non-technical user and does all the technical work itself. Use when the user wants to set up the video project, add a channel, find video ideas, share a style reference, edit a recording, or plan, research, animate, voice or render a scene.
argument-hint: "[setup | new channel <name> | video ideas | style reference <link or file> | edit <recording> | scene idea]"
---

# TubeAI Video: assisted animations and edits

The user edits videos for YouTube channels. We build animated scenes in Remotion that they drop into their edit, and we can edit a creator's raw recording for them: cut it, time the inserts to the words, move the frame on the speaker, master the voice, and hand it over as a timeline they finish in Premiere Pro, Final Cut Pro or DaVinci Resolve. Each channel's branding stays consistent: its styling and animations are set once, then reused or used as the base for new ones. The user brings ideas, some of the media and sometimes a raw recording, and can share a YouTube link or a video file to show the style of the current animations. Through the optional TubeAI connector, Claude also helps find video ideas and draft scripts. Claude researches the rest, builds the animations, and cuts and renders.

## The user isn't technical

Assume the user isn't technical, and make all of this as seamless as possible for them.

- Claude does the technical work: installs, commands, scripts, config, renders, conversions and fixes. Never ask the user to run a command, edit a file or read code.
- When something truly needs the user's hands (approving an install, signing in, connecting the Chrome extension, updating a graphics driver), give short numbered steps at the level of what to click, and check afterwards that it worked.
- Talk in plain language. The first time a technical word can't be avoided, explain it in a few words. Report outcomes, not internals: what's ready, where it is, what's next.
- Pick sensible defaults and carry on. Ask only when it's genuinely the user's call (creative direction, brand, script wording, permissions, anything that costs money): one question at a time, with a recommendation.
- Never make the user wait on tooling. Scripts, agent files, installs and project upgrades are yours: add what a task needs while doing it, and mention it in a line.
- Show instead of describing: send stills, renders and voice samples, so the user approves by looking and listening.
- When something breaks, tell the user in a line or two what happened while you fix it or work around it (see Keep it fast and smooth). Bring them a problem only when it needs their decision, and then explain it plainly, with the options.
- Deliver files ready to use: give the exact path to each one, and say which file goes into their editor.
- Sub-agents report to the main chat, which turns their reports into plain language for the user.

## Keep it fast and smooth

The user wants a quick setup with no fuss, and then videos made without friction. Every stall is yours to solve.

- **Report it and fix it at the same time, so nothing waits on it.** When something fails:
  - Tell the user in a line what happened and what you're doing about it, for example "The voice model download dropped, so I'm restarting it while I set up the templates."
  - Fix it right away, in the background where you can: retry once if it looks temporary (a dropped download, a locked file), apply the known fix (from Troubleshooting or the error message itself), or reach the same result another way (another download source, the CPU instead of the GPU, a smaller voice model, another install method).
  - Keep going with everything that doesn't depend on it.
  - Say in a line when it's fixed. If the fix needs the user's decision or hands, ask with one recommended option, and keep the rest moving while you wait.
- **One OK for all the installs.** List in plain words everything setup will install, and ask once. Warn the user that their computer will ask for permission a few times: on Windows they click **Yes**, and on a Mac they type their password.
- **Ask early.** Ask what only the user can answer (which channel, its logo, colors and fonts, a style reference) at the start of setup, still one question at a time, so the answers come in while things install instead of holding up the end.
- **Skip what's already there.** Check what's installed and working first, and never reinstall something that works.
- **Slow parts first, side by side.** Start the big downloads (PyTorch, the transcription and voice models) in the background right away, and do the other steps while they run. Later, renders, voiceovers and transcriptions also run in the background while the chat carries on with the user.
- **The usual snags:**
  - On Windows:
    - A tool installed a moment ago "isn't found": the shell still has the old PATH. Reload PATH from the registry in the command, or call the tool by its full path.
    - winget waits on a prompt: add `--accept-source-agreements --accept-package-agreements`.
    - winget is missing (older Windows 10): download the official installers directly.
    - `python` opens the Microsoft Store: use `py -3.12`.
    - PowerShell won't run `npm` or `npx` ("running scripts is disabled"): use `npm.cmd` and `npx.cmd`.
  - On a Mac:
    - `brew` "isn't found" right after Homebrew installs: it's in `/opt/homebrew/bin` on Apple Silicon (`/usr/local/bin` on Intel), which the current shell doesn't know yet. Call it by that path, or load it with `eval "$(/opt/homebrew/bin/brew shellenv)"`.
    - A build stops and asks for developer tools: Apple's command line tools are missing. `xcode-select --install` opens Apple's installer, and the user clicks **Install**.
    - `python3` is macOS's own older copy: use Homebrew's `python3.12`.
  - The GPU runs out of memory: step down (the 0.6B voice model instead of 1.7B, a smaller transcription model, a lower render concurrency, or the CPU), note it under "This machine" in `GUIDELINES.md`, and say so in a line.
- **Check it before calling it ready.** Probe every file before handing it over: a render plays for the right length and passes QA, a voiceover has sound, a timeline passes every check in the Timeline recipe. The user should never be the one who finds the problem.
- **Short updates.** One line at each milestone, with a rough time for anything long ("the tools are in; the voice model needs about 5 more minutes"), never a stream of logs.

## Start of every session

1. **TubeAI is optional.** Follow *Connect first* in the `tubeai-mcp` skill: if TubeAI isn't connected, offer it once, in a line, and carry on either way. If the user says no, note it under "This machine" in `GUIDELINES.md`, and don't offer it again unless they ask.
2. The project is the folder that has `GUIDELINES.md` and `remotion.config.ts`. If there's none and the user wants to start, run **Setup**. If the current folder is another project or isn't empty, ask where the video project should live before installing anything.
3. Otherwise read `GUIDELINES.md`, `README.md` and `PROJECTS.md`, and use `PROJECTS.md` to find the channel and video to work on:
   - The user names one: match what they say against the channels and videos listed (titles, other names, folders). If more than one matches, ask which.
   - They don't: list the work in progress and suggest what was updated most recently.
   - There's no `PROJECTS.md` yet (the project was set up before it existed): build it first from the `channels/` folders and their `BRIEF.md` files, and add `@PROJECTS.md` to `CLAUDE.md`.

   Then read that channel's `CHANNEL.md` and, for a video, its `BRIEF.md`. Sum up where things stand in 2–3 lines, ending with the next step, and carry on.
4. If the project is missing something this skill describes (a script, an agent file, a folder, a tool), it was set up with an older version of the skill. Add what the current task needs while doing it, and mention it in a line. Never hold a task back to ask about it.

## Workflow: main chat plans, sub-agents build

- The main chat is for discussing ideas and drafting scripts with the user, and for exploring YouTube data through TubeAI when it's connected (the `tubeai-mcp` skill). Turn each agreed idea into a brief, send it to sub-agents, check what they return, show the user (SendUserFile) and iterate. As a rule of thumb, research, editing and animation happen in sub-agents, not in the main chat.
- A scene needs 1–2 sub-agents, all running Opus 5.5 at xhigh effort. `video-researcher` runs first, and only when the scene needs media we don't have yet. Then `video-animator` builds and renders the scene with that media.
- A raw recording goes to `video-editor` (see Editing a recording). The inserts planned on top of it are then scenes like any other. When there are many, group them by type (overlays, data boards, article cards, charts): one `video-animator` per group, each in its own folder, all sharing the one timing table.
- Run at most two heavy agents at a time: more hits the user's usage limit. For revisions, continue the same agent (SendMessage) so it keeps its context.
- Mind the machine too: one transcription at a time, at most two still renders at once, and a recording's final renders in one batch at the end, under the render lock (see Rendering). A verbatim transcriber (about 6.5 GB of RAM), Whisper, Chrome renders and several agents at once have crashed a laptop.
- Don't overengineer: no verification rounds between agents, demo compositions, catalog write-ups, mock sets or renders nobody asked for.
- Record every decision in the video's `BRIEF.md`, and turn every correction the user makes into a written rule, so it never has to be made twice: in `CHANNEL.md` when it's about this channel's look, voice or pacing, in `GUIDELINES.md` when it's about how we work.
- Spawn the agents by `subagent_type` (their files are in `.claude/agents/`) without a `model` override. If those types aren't available in this session (for example, they were just created), use `general-purpose` with `model: opus` and paste the agent file's body above the brief. Tell the user that effort follows the session setting until a new chat loads the agents.
- The agent files already hold the default instructions (read the MDs and the channel in full). The brief you send adds only the specifics:

  ```text
  Channel: <slug> · Video: <yyyy-mm-dd-slug> · Scene: <CODE>-<video>-s<nn>
  Goal: <what the viewer should take away, 1–2 lines>
  Specs: <resolution and fps; mp4 | full-frame card (split render) | overlay (alpha); the length comes from the window>
  On screen: <exact copy, numbers, what gets highlighted and when>
  Window: <its row in cut/insert-windows.json (a recording) or its beats in timings.json (a voiceover): start word, end word, the word that brings in each element, and how it enters and exits>
  Media: <user-provided paths, links, researcher output>
  Reference: <YouTube link or video file (+ timestamps) whose style to match, if any>
  Reuse: <templates and channel animations to use; what's new>
  ```

- When an agent returns, check that the files exist and passed QA (see Rendering), show the user the stills or render, and update the scene's status in `BRIEF.md` and the video's row in `PROJECTS.md`.

## Layout

```text
<project>/
├─ CLAUDE.md              # "@GUIDELINES.md" and "@PROJECTS.md", so every new chat loads the rules and the current work
├─ GUIDELINES.md          # workflow rules, the source of truth for chats and agents
├─ PROJECTS.md            # every channel and video: where each stands and what's next
├─ README.md              # for the user: preview and render by hand
├─ remotion.config.ts     # public dir = media/, GPU settings (from core/lib/render-settings.ts)
├─ .claude/agents/        # video-researcher.md, video-animator.md, video-editor.md
├─ .claude/skills/        # remotion-best-practices (Remotion's official agent skill), and tubeai-mcp if it didn't come with the plugin
├─ .venv/                 # Python: Qwen3-TTS voices, Whisper and CrisperWhisper transcripts
├─ core/                  # logic shared by all channels (no branding)
│  ├─ index.ts, Root.tsx  # entry; mounts each channel's <Folder> and the template demos
│  ├─ templates/          # XPostCard, RedditPostCard, ArticleHighlight, DocumentCard, Montage, CTA, transitions
│  ├─ components/, lib/   # shared building blocks; the timing-table helpers, render settings
│  └─ scripts/            # capture, clip, voice, transcribe, cut, master, assemble, timeline, render, qa; lib/ for the code they share
├─ channels/<slug>/       # one per channel; <slug> is its name in kebab-case
│  ├─ CHANNEL.md          # brand guide, voice, glossary, style reference, animation catalog
│  ├─ theme.ts            # brand tokens
│  ├─ animations/         # reusable branded animations
│  ├─ index.tsx           # the channel's compositions
│  └─ videos/<yyyy-mm-dd-slug>/  # BRIEF.md, scenes/, and cut/ for a recording (cuts, timing table, versions)
├─ media/<slug>/
│  ├─ user-provided/      # the user's files, read-only for Claude
│  └─ automated-research/ # everything Claude finds or makes: <video>/, style-refs/ and voice/, each with SOURCES.md
├─ recordings/<slug>/     # raw recordings, outside media/ so renders never copy them
├─ archive/<slug>/        # research media of shipped videos, kept out of media/
└─ out/<slug>/<video>/    # renders, the voice master, and timeline/ with the .xml, .fcpxml and .otio exports, REVIEW.md and any LUT
```

## PROJECTS.md

`PROJECTS.md`, at the project root, maps the current work: every channel and video, where each one stands, and what's next. `CLAUDE.md` loads it into every chat, so a new chat knows where to start without searching.

- The top line gives the totals. Then each channel gets a section with its name, its folder, and how many videos are in progress and shipped.
- Work on the channel itself that isn't part of a video, such as building its starter animations, gets one line under the channel's heading, with its next step.
- Each video in progress gets a row: its working title (plus any other name the user calls it), its folder, its status in a few words, the next step, and the date it last changed. Make the next step specific enough to start on, naming the scene and the action ("animate s05"), never just "continue".
- A shipped video moves to its channel's one-line "Shipped" list.
- It only points the way: the detail stays in `CHANNEL.md` and `BRIEF.md`.
- Update it whenever a channel or video is added, a video's status or next step changes, or a video ships.

```markdown
# Projects

<n> channels · <n> videos in progress · <n> shipped. Update this file whenever a channel or video is added, a video's status or next step changes, or a video ships.

## <Channel name> · `channels/<slug>/` · <n> in progress, <n> shipped

Channel work: <e.g. starter animations, 4 of 6 built> · next: <e.g. build the lower third>

| Video | Folder | Status | Next step | Updated |
|---|---|---|---|---|
| <working title> (<other names>) | `videos/<yyyy-mm-dd-slug>/` | <e.g. 4 of 6 scenes rendered> | <e.g. animate s05> | <yyyy-mm-dd> |

Shipped: `<yyyy-mm-dd-slug>` (<title>), …
```

## Channels

- Each channel gets one folder in `channels/` and one in `media/`, both named by its slug (the channel's name in kebab-case). Channels come from the user: the first one is whichever they name at setup.
- `CHANNEL.md` is the brand guide, and every agent reads it in full. It covers:
  - audience and tone
  - canvas (resolution, fps)
  - palette (hex values)
  - fonts
  - logo usage
  - motion style (easings, durations, how things enter and exit)
  - transitions: the in and out transitions, their lengths and sounds, and the end dip
  - voice: the Qwen3-TTS model size, the speaker (or designed voice), the speed and, on 1.7B, the tone instruction; or the creator's own voice
  - glossary: names and terms, spelled correctly (transcripts are corrected against it, and captions use it)
  - pacing: the pause policy for each host: every natural pause kept (the default), pauses capped at a length (such as 0.5 s), or cut tight (about 0.06 s, for a screen-share format); measured from the host's first recording or the creator's published edits (share of silence, median and 95th-percentile pause), and confirmed with the user
  - do's and don'ts
  - where the channel departs from the defaults for moves on the speaker, sound and photo credits
  - the composition-ID code: a few capital letters from the channel's name
  - the style reference: source and findings (see Style references)
  - a catalog of the reusable animations, with when to use each
- `theme.ts` holds the same values as tokens. Scenes never hard-code brand values.
- Reuse first, then derive, then create. Compose existing animations first. Make variants through props or wrappers, never by copying and editing. Build something new only when needed. A pattern used twice moves to `animations/` with a one-line catalog entry.
- New channel: ask the user for brand inputs (logo, colors, fonts) and, if they have one, a style reference: a YouTube link or a video file that shows the channel's current animations. Fill the remaining gaps by researching the channel's look (banner, thumbnails, recent videos). Propose the theme, the motion style and a starter set of animations, for example:
  - title/section card, lower third, stat callout
  - headline/quote card, transition, end card
  - channel-specific pieces, such as ticker or chart reveals for a markets channel

  Build once the user approves.
- The user can share a new style reference at any time, to refresh a channel's look or to match one specific animation.

## Style references

The user can provide a YouTube link or a video file to show the style of the current animations, for a whole channel or for one specific animation. When there's a reference, match it instead of guessing.

- Getting it: download a YouTube link with `npm run clip` into `media/<slug>/automated-research/style-refs/`. If the user gives timestamps, clip only those parts; otherwise take the whole video. Read a video file from the user where they put it, and never move it.
- Looking at it: pull frames with FFmpeg.
  - Overview: one frame per scene change, tiled into contact sheets, e.g. `ffmpeg -i ref.mp4 -vf "select='gt(scene,0.3)',scale=640:-1,tile=4x4" -fps_mode vfr sheet-%02d.png`.
  - Motion: every frame around each animation, to measure durations in frames, easing, direction and stagger. Delete these frame runs once measured and keep the sheets.
- What to extract:
  - palette (hex values sampled from the frames; the user's official values win if they have them)
  - typography (the closest match; ask the user for the actual font files)
  - layout and margins
  - recurring elements (lower thirds, callouts, cards, transitions, logo sting)
  - motion style
- Where it goes: the findings, with the source and the key sheets, go into `CHANNEL.md` under "Style reference", and from there into `theme.ts` and the animations. When an animation is rebuilt from a reference, show the user side-by-side stills (the reference frame next to ours) before building on it.
- Who does it: `video-researcher` gets the video, pulls the frames and writes the findings. `video-animator` builds from them.

## Media

- `media/` is Remotion's public dir, so files load with `staticFile('<slug>/user-provided/…')`. Every render copies all of it, so raw recordings never go there: they live in `recordings/<slug>/`, or wherever the user keeps them.
- `user-provided/` belongs to the user. Never rename, move, re-encode or delete anything there. Converted or cropped copies go to `automated-research/`.
- `automated-research/` holds everything Claude downloads, captures or generates, including from links the user supplied: `<video>/` for a video's media, `style-refs/` for style references, `voice/` for a designed channel voice.
  - Use kebab-case names prefixed with the scene (`s03-article-screenshot.png`).
  - Every file gets a line in its folder's `SOURCES.md`: file · source URL · date captured · what it shows · scene.
- Articles: capture the text (headline, outlet, author, date, paragraphs → JSON) and a screenshot, then rebuild the article in `ArticleHighlight` in the outlet's own look: its logo (for identification only), page colours and the closest free fonts. Compare the rebuild side by side with the screenshot before using it: a stand-in font that looks off gets noticed.
- Documents (regulator pages, filings, PDFs, papers): rebuild them as clean text or table cards (`DocumentCard`). Never use raw screenshots of them: they come out small, cluttered and cropped.
- X and Reddit posts: capture the real post data (text, author, handle, avatar, date, metrics, media) plus a reference screenshot, then rebuild the post in the card templates.
- YouTube: find the video through TubeAI when it's connected (the `tubeai-mcp` skill), otherwise on YouTube itself, then use yt-dlp and download only the section needed (`npm run clip`). If downloads fail with HTTP 403, see Troubleshooting. Videos on other sites (X, Reddit, Vimeo, news sites and most others) download the same way: `npm run clip` takes their links too, since yt-dlp supports them.
- Voiceover: a recorded voiceover from the user or creator goes in `user-provided/`. Synthetic voiceovers from `npm run voice` go in `automated-research/<video>/vo/` and are logged in `SOURCES.md`. Never reuse the narration or music of a reference video or another channel.
- Accuracy: never invent or alter quotes, posts, headlines or figures. Rebuilt cards must match the source as captured. Make mock content only when the user asks for it, and label it as mock in `BRIEF.md`.
- Figures: check every figure in the script against a primary source before animating it. When one is imprecise or unsupported, show the user the discrepancy with options and a recommendation, and let them decide: a chart must never contradict the narration, so either the script changes or the chart uses the defensible measure.
- In a recording, the words are already said, so labels follow the data. When the host misstates a figure (31% rounded up to "about 32%", or "banks" for a group that includes fund managers), the screen shows the correct one and `REVIEW.md` lists it. The host's joke names for companies never go on screen.
- A relative date ("in 27 days") is true on one upload day only. List every such line in `REVIEW.md`, with the upload date it assumes.
- Photos: credit them in the video description (`REVIEW.md` lists the credits), and on screen too only when the channel wants it. Logos are for identification only.
- Paywalls and bot walls: never work around them (no archive sites, no CAPTCHA solving). Read through the user's own signed-in browser (Claude in Chrome, signed in to the same account as the app); many sites block headless tools, the app's built-in browser pane and WebFetch, even when the page is free. When a publisher asks that large parts not be reproduced, show a brief excerpt. Automated downloads follow each site's access rules, such as declaring a user agent.
- Browsing and capture:
  - Claude in Chrome (the app's browser tools): finding pages, logged-in, paywalled or bot-blocked sites, and visual checks
  - `npm run capture` (headless Chromium): the files that go into the video
  - WebFetch: quick text

## Voice and transcripts

- **Voiceover:** `npm run voice` runs Qwen3-TTS locally, in the project's Python environment. The hardware check picks the model size, 1.7B or 0.6B, and `CHANNEL.md` records the channel's voice. The models are on Hugging Face under `Qwen/`: `Qwen3-TTS-12Hz-1.7B-CustomVoice`, `Qwen3-TTS-12Hz-1.7B-VoiceDesign` and `Qwen3-TTS-12Hz-1.7B-Base`; `Qwen3-TTS-12Hz-0.6B-CustomVoice` and `Qwen3-TTS-12Hz-0.6B-Base`.
  - **CustomVoice** has 9 preset voices and is the default for narration. For a new channel, render one sample line per speaker (`--samples`) and let the user pick by ear. For English, Ryan and Aiden are the native speakers.
  - **Base** clones a voice from about 3 seconds of reference audio plus its transcript. Use it only for the user's own voice or one they have permission to use, never a creator's or public figure's voice without it. It's handy for patching a flubbed line in a creator's recording, with the creator's OK.
  - **On 1.7B**, CustomVoice also takes a tone instruction in plain words, for the whole voiceover (`--instruct`) or for one beat. The channel's default instruction comes from its audience and tone in `CHANNEL.md`: draft it with the user and settle it by ear with a few samples, the same way as the speaker. A beat gets its own instruction only when the script calls for a different delivery. **VoiceDesign** makes a new voice from a description the user agrees on (gender, age, pitch, pace, character). A designed voice can drift from line to line, so to make one the channel's voice, render a reference clip with it once, save it in `media/<slug>/automated-research/voice/`, and voice every line with Base cloning that clip.
  - **On 0.6B**, the models don't take instructions: tone comes from the speaker, punctuation and line breaks.
  - Input is `beats.json` (`{ "beats": [{ "id", "text", "pauseAfter"?, "instruct"? }] }`). Output is `vo.wav` (48 kHz, loudness-normalized to -16 LUFS), one WAV per beat, and `timings.json`.
- **Transcripts:** `npm run transcribe` runs two models locally, in the project's Python environment, on a 16 kHz mono WAV with word timestamps on. The hardware check picks their size.
  - **Whisper** (OpenAI's open model, through `transformers`) gives the words. It takes a prompt, so the channel glossary (and the script's names, when there is one) goes in, and names and terms come out spelled right. But it hides stumbles: it folds a restart and its retake into one clean sentence, and drops repeats and fillers, so a re-check with the same Whisper pass looks clean too.
  - **CrisperWhisper** (the `crisperwhisper` package: `large`, `medium` or `turbo`) is the verbatim pass that finds them: fillers, stutters, repeated words and false starts stay in, with word timings accurate to about 30–40 ms, so scenes sync to them directly. It takes no prompt, so it misspells names: the transcript keeps its words and timings, with Whisper's spelling of names and terms. `mode="intended"` gives a clean, formatted version when one is wanted, such as for captions. `forced_align(audio, text)` times a known text against the audio, such as a script line or a join being checked. `verbatimize(audio, text)` adds the real disfluencies to a clean transcript.
  - Word-by-word captions get a check against the audio. Flag any word you're unsure of for review.
- **`timings.json`** is the one timing format both scripts write, in seconds: `{ duration, beats: [{ id, text, start, end }], words: [{ text, start, end }] }`. `voice` gets its word timings by aligning its own text to its output (CrisperWhisper's `forced_align`). A voiceover's scenes time off this file; a recording's scenes time off the timing table built from it (see Editing a recording). Neither ever hard-codes a time.

## Editing a recording

When the user hands in a creator's raw recording (a file in `recordings/<slug>/`, or a path), edit it into a timeline they finish in their own editor. The recording itself is never changed, so there's nothing to approve before cutting. `video-editor` does the transcript, the cut, the voice master, the timing table, the moves on the speaker and the final assembly; `video-animator` builds the inserts.

**The deliverable is a timeline, not a video.** Editors polish and export in Premiere themselves. Deliver the Final Cut Pro 7 `.xml` (Premiere imports it as a ready sequence: File → Import), an `.fcpxml` for today's Final Cut Pro (File → Import → XML) and an `.otio` (DaVinci Resolve 18.5+: File → Import → Timeline; Premiere Pro 25.6+). Render an MP4 only when the user asks for one. With the timeline go `REVIEW.md` (see The review list) and, when the editor has graded a frame, a LUT (see Colour).

Give the user a rough time for the whole edit up front, then work in this order:

1. **Transcribe** the recording (`npm run transcribe`, see Voice and transcripts).
2. **Read the script**, if there is one, and research what it claims: check every figure against a primary source (see Media).
3. **Brief the inserts** in `BRIEF.md`: every proposed insert with the sentence it goes on (see Inserts). The user approves the brief before any cutting, and usually adds more inserts, articles especially.
4. **Cut** it (below), and deliver the cut-only timeline, so the user can review the cut while the inserts are built. For a new host, confirm the pause policy first.
5. **Master the voice** (`npm run master`, see Sound).
6. **Build the timing table** (below). With a host who improvises, re-plan the inserts from the real cut first (see Inserts), and show the user what's new.
7. **Build the inserts**, each timed from its window in the table, and **plan the moves** on the speaker (see Moves on the speaker).
8. **Assemble** the timeline. Test the assembly first with tiny stand-in files at the planned frame counts. Then render every insert in one batch, and run `npm run assemble`: it builds the timeline spec from the windows, renders, moves and voice master, writes the `.xml`, `.fcpxml` and `.otio` (`npm run timeline`), and runs every check in the Timeline recipe.
9. **QA** everything: the renders (see Rendering), the timeline's checks and the sound checks.
10. **Deliver** the full timeline with `REVIEW.md`, giving the exact path to each file.

**Cutting**
- With a script, diff the transcript against it word by word. Script text said twice is a retake: keep the last complete take, unless an earlier one is clearly cleaner. A run that matches the script, breaks off and starts again is a false start. Runs that match nothing need judgement: improvisation, or a mistake.
- Find every stumble, not only the ones the transcript shows (a Whisper-driven cut once left in 20: 9 repetitions, 6 false starts, 3 stutters and 2 fillers):
  - the verbatim pass against Whisper's words: any word only the verbatim pass has is a candidate;
  - each sentence re-transcribed as its own short clip by Whisper with a verbatim-leaning prompt ("Um, uh, I- I mean, the- the, so so…"), then diffed against the transcript: any extra word is a candidate;
  - the audio where the text looks clean: speech energy with no word under it, a word that lasts too long for its syllables, and sentence starts, where restarts cluster.
- Cut only clear mistakes: false starts, flubbed words, retakes.
  - Pauses follow the host's pause policy (pacing in `CHANNEL.md`). By default, every natural pause stays. For a new host, measure their pauses and confirm the policy with the user before cutting.
  - Keep rants, tangents and ad-libs. The script is a guide.
  - An aside the speaker takes back ("you know what, I'm not going to say this") is an outtake: cut all of it, even when it's unique. So is a note to the editor said on camera ("cut that"): cut the passage and the instruction.
  - Content beats a clean join. If the only clean cut would remove a sentence of real content just to lose a stutter, keep the sentence, stutter and all, or ask.
- Make clean joins: place every cut by the silence in the audio, not by the word times, because a short word at a pause ("so", "but") is often put on the wrong side of it. Cut in real silence at a phrase boundary, never leave a repeated word across a join ("so… so"), and don't cut where nothing is wrong. When a join is rough, move it to the sentence break before or after, or pick a different take boundary.
- Verify every join by re-transcribing about 2 s either side of it, with the verbatim pass or the verbatim-leaning prompt, looking for repeated or partial words.
- `cut/cuts.json` holds only this video's decisions: the removed ranges, each with its reason, and the word fixes. The shared cut library (`core/scripts/lib/`) turns them into the frame-accurate kept segments at any frame rate, the cut's word timings and a reference WAV of the cut, with a 15 ms fade at each cut, for the checks. Write `cut/CUTS.md` too: the transcript with every removal struck through and its reason, a short "your call" list of the genuine judgement calls, and a "listen to these" list of joins between words.
- In the timeline, cuts are plain edits, and a 1–2 frame crossfade in Premiere fixes any click. A rendered cut gets a 15 ms audio fade at each cut instead.

**The timing table** (in `cut/`, rebuilt on every re-cut)
- `words-cut.json`: every kept word with its start and end in cut time, with the word fixes from `cuts.json`.
- `insert-windows.json`: for each insert, its start phrase, end phrase, the spoken text, its start and end in cut seconds, its beats (the word that brings in each element), its role (full-frame card or overlay), and how it enters and exits (its own transition, a hold or a dissolve; see Inserts).
- The core helpers map source time to cut time from the kept segments, and find phrases loosely (`findPhrase(phrase, { after, before })`, `findAll`, `wordsBetween`), ignoring case, punctuation, number words against digits, currency signs and thousands separators.
- Scenes import the table when they're built and never hard-code a time, so every re-cut re-syncs every insert.

**Feedback** comes as timecodes from the version the user reviewed. Keep each delivered version's `cuts.json` in `cut/versions/`, and map feedback through that version, not the current one. If the timecodes look offset, match by content.

To fix an insert after the editor has imported the timeline, re-render it under the same file name and frame count, and the timeline stays valid. On Windows, check first that Premiere isn't holding the file: an exclusive open succeeds.

Keep the recording's frame rate throughout, including NTSC rates like 29.97 and 23.976, as the exact fraction: 30000/1001 at 29.97 (1,601.6 audio samples per frame at 48 kHz), never rounded through 30.

**The review list.** `REVIEW.md` goes with every delivered timeline, in `timeline/`, with times on the cut:
- the cut's judgement calls, each with its default already in place;
- the joins worth a listen, and the moment where the limiter works hardest;
- a table of every place the screen corrects the host: what they say, and what's on screen;
- every line with a relative date, and the upload date it assumes;
- builds worth a glance, such as a long hold or overlapping builds;
- the moves on the speaker, the track layout, the sound and the colour;
- the photo credits, ready for the video description.

## Inserts

An insert is a scene laid over a recording: an article or document card, a chart, a graphic, a montage, a clip, a CTA.

**Planning**
- One insert per statement. Two statements worth showing get two inserts: one card stretched over both is a planning and design mistake, not only a sync mistake.
- When the speaker rambles in the middle of a point, split the insert around the ramble.
- Step-by-step explanations and analogies get a step build that lasts as long as the explanation (one step per phrase), or several cards. A short card can't hold them.
- Things named in a row (companies, people, places) get a quick montage: one photo per item, each on its name. A company shows as its headquarters, with the logo visible.
- Plan richly. Editors want more articles, charts and graphics than the script asks for: about one insert every 30 seconds works, as long as each one is well timed.
- With a host who improvises, re-plan the inserts from the real cut: ad-libs bring new lists, people, numbers and analogies. The script's cue phrases often aren't said word for word, so anchor every insert on the words actually said, and drop any hint that depends on the script's order.

**Sync** (most corrections land here)
1. An insert starts on the first word of the sentence it illustrates, and ends right after that sentence or thought. Its length comes from the narration: a fixed 5–7 seconds is wrong almost every time.
2. Every element inside an insert (list items, logos, events on a timeline, bars, numbers) appears on the word that names it, never ahead of the speech, and each figure gets time to be explained before the next one lands.
3. A data insert goes where the claim is said, not where the topic is introduced.
4. Nothing stays up after the thought is over.
5. A marker or highlight on a quoted sentence starts when the speaker starts saying it.

A scene's length is its narration window plus its exit (or its hold, see Joins): frame 0 is the start word, and the exit starts right after the end word.

**Joins**
- Back-to-back inserts (the next one starts within about 1 s of the last one's end): the first holds, and the next cross-dissolves in over it in 6 frames, on the track right above it. Only the first insert of a run enters with its transition, and only the last exits.
- An overlay followed straight away by a card holds under it, so the card's entrance covers it. Without the hold, the overlay plays its exit and the card its entrance back to back.
- A full-frame overlay that plays like a card (a split screen with a window for the speaker, say) joins a card that ends within about 1 s before it with the same cross-dissolve, as two cards would.

**Design**
- A card format editors approved: the related photo, sharp for a beat, then blurred and dimmed (a crossfade to a pre-blurred copy, see Rendering) while a rounded card springs up with the content; then the insert pushes out. Use it for a channel that has no card style of its own yet.
- Fill the card: no big empty areas, and stat cards sized to their content. Nothing cropped at the card's edges, except a deliberate page continuation.
- No box waits empty: a card's box opens with its first content, not before it.
- Numbered lists run vertically, one item under the other, never across a strip. They're big: about a third of the frame.
- A highlight behind dark text is a light tint, with black text on it at a contrast of 12:1 or more, never a saturated fill.
- Only real numbers roll. A rolling-number (odometer) effect once showed the index "WIG20" as "WIG10" mid-roll, and "4 MONTHS" as "2 MONTHS": words that contain digits land static.
- Clip only the axis an animation needs. Big type is often nudged a few px left so its stems line up with a small label, and inside a box with `overflow: hidden` that cuts off the left curve of a first C, G, O or S and the ends of T and W. For a vertical reveal, clip with `clip-path: inset(0 -<pad> 0 -<pad>)` instead.
- Charts highlight the claim and drop anything that muddies it. Every figure comes from the data at build time, and the build fails if one doesn't match.
- Transitions are baked into each insert, so they reveal the recording underneath in any editor: the channel's in and out transitions, and a small transparent-to-black overlay for the end dip. A full-frame card renders in three parts, with alpha only in its head and tail, and an overlay as one ProRes 4444 file (see Rendering). A baked transition can't be retimed in the editor. If the user would rather adjust transitions there, test FCP7 `<transitionitem>`s first: how they map to Premiere's own or third-party transitions is uncertain.
- Sounds, such as a short UI click (about 30–40 ms) at each card start and montage transition, are the channel's choice (`CHANNEL.md`). Make them with FFmpeg. Card renders are silent: each sound goes on the timeline as a clip of its own (see Sound). An overlay carries its own audio.
- A CTA is built from scratch with the channel's real name and avatar, never a template's placeholder.

## Moves on the speaker

In a recording's timeline, the frame also moves on the speaker: cuts to a new framing, and eased zooms and pans. They're written into the timeline (see Timeline recipe), so the editor can still adjust them. These are the defaults; `CHANNEL.md` wins where it sets its own.

**When to move**
- To make room for an overlay: away from a side panel, or scaled into a framed window.
- For emphasis, a punch-in: a cut to a tighter framing (108%, centred on the face). Use it on a jump cut while the speaker is on screen (it reads as a second camera), when they start an aside or a story, or on a short, blunt answer. It holds until the next full-frame insert covers them, and they reset under that insert, unseen. Never zoom in and back out in view on a short segment.
- A slow eased push (100% to 105% over about 3 s) on the intro and on the verdict, held the same way.
- **Not** while an overlay holds the speaker in place, under a lower third, or in 2–3 s gaps between cards.
- **How often:** about one move per 45 s of the speaker on screen.
- **How far:** with a 1080p recording in a 1080p sequence, never past 110%, or the picture goes soft. A 4K recording has more room.

**Moving for an overlay**
- The speaker and the overlay move as one motion: the speaker pans as the panel opens, and back as it closes. A panel must never cover them before they move.
- An overlay with a window for the speaker enters with the window closed. The speaker cuts to their window framing on a frame where the overlay covers them completely, then the window opens on them, already in place. The way out mirrors it. To check it, the overlay's lowest alpha anywhere in the frame must be fully opaque on the cut frame.
- To keep one point fixed while scaling (the face): centre = P − (P − C₀) × s, where P is that point at 100%, C₀ is the frame centre and s is the scale as a fraction.

Every move gets a sequence marker and a line in `REVIEW.md`. For scale, one 24-minute edit had 54 moves: 1 intro push, 13 punch-ins, 17 resets under cards, 19 panel pans and 4 window cuts.

## Sound

**The voice master.** A raw recording usually plays well under YouTube's -14 LUFS reference (one played at -20.5 LUFS). Process the whole recording's audio into a voice master, a new 48 kHz, 24-bit WAV; the recording itself is never touched. The FFmpeg chain:

```text
highpass=f=70,
acompressor=threshold=-20dB:ratio=2.5:attack=8:release=150,
volume=<make-up>dB,
aresample=192000,
alimiter=limit=-1.7dB:attack=5:release=50:level=0:latency=1,
aresample=48000
```

- Measure the make-up gain on the cut, not on the whole recording, in two passes: measure without it, then set it so the cut lands on -14 LUFS integrated (one cut measured -24.2 LUFS and took +10.4 dB). The limiter runs at 4× oversampling, which keeps the true peak under -1.5 dBTP; that cut ended with a loudness range of 3 LU.
- In the timeline, the voice master is double-system sound: its own file on A1/A2, cut exactly like V1, linked to each V1 clip, at offset 0.
- Find the moment where the limiter works hardest, and list it in `REVIEW.md` for a listen.

**Effects**
- Card renders are silent, and each click or sound is a clip of its own on an SFX track, at its frame. A sound can then be swapped or re-levelled in one place, and MP4 audio priming (about 43 ms late) can't shift it. An overlay carries its own audio in its ProRes file.
- Once the voice comes up, bring the effects and the overlays' audio up with it as clip gain (+4 dB by default), keeping each one under -1 dBTP after the gain.

**Checks**
- Rebuild the voice from the timeline's own clip values, the way the cut library builds its reference cut of the master, and compare the two: the difference must be zero, at a cross-correlation lag of 0 ms.
- Every join sits in silence on the processed voice. Raise the silence threshold by the make-up gain (-29.6 dBFS in one edit).
- Transcribe the opening of the mastered cut and compare it with the cut's words (one edit matched 99%).

## Colour

Premiere's FCP7 XML import doesn't recreate adjustment layers or Lumetri, as far as we know (a round-trip test is still to do). So in a recording's timeline, V2 stays empty for the editor's own adjustment layer, with every other video track above it.

**A LUT that matches the editor's grade.** When the editor grades one frame the way they like it and shares it, build a LUT from it:
1. Take the same frame from the recording.
2. Align the two, since the graded frame may be zoomed or shifted: search at 320×180 over scales 1.00–1.10 and ±12 px, scoring by the correlation of edge gradients (the grade doesn't change them), then refine.
3. Pair the pixels, skipping edges, where a small misalignment hurts.
4. Fit a 3rd-order polynomial per output channel, with a small ridge. Its 13 terms are 1, r, g, b, r², g², b², rg, rb, gb, r³, g³ and b³.
5. Bake it into a 33³ `.cube` in `timeline/`. Check the mean error (the first one came to 4.6/255) and show the user a before/after.

Simpler matching fails: matching the mean and standard deviation washed the image out, and per-channel histogram matching turned whites pink, because the two frames' content differed.

The editor applies it once in Premiere: New Item → Adjustment Layer, laid on V2 across the sequence → Lumetri Color → Creative → Look → the `.cube`. Nothing adds it after import yet, so give them these steps.

## Core templates

They live in `core/templates/`, carry no channel branding and are driven by props. Each one has a zod schema with sample defaults (editable in Studio), `dark` and `light` variants, and a demo composition. Channel scenes wrap them with theme tokens for framing, fonts and motion.

- `XPostCard` copies the real X layout: avatar, name, verified badge, handle, time, text (with optional word highlights), media and metrics.
- `RedditPostCard` shows the subreddit, user, post age, flair, title, body or media, votes and comments.
- `ArticleHighlight` shows the outlet, headline, byline/date and paragraphs.
  - An opaque marker, a light tint of the theme's highlight colour (see Inserts → Design), sweeps behind each highlighted phrase from left to right, one line at a time when the phrase wraps, starting when the speaker starts the sentence.
  - An optional slow push-in moves toward the line being highlighted.
  - Two modes: rebuilt text (the default), or a screenshot with highlight boxes in normalized coordinates, only for an article that can't be rebuilt.
- `DocumentCard` rebuilds a document (a regulator page, filing, PDF or paper) as clean text or a table, sized to its content.
- `Montage` shows one photo per item named in a row, each landing on its name, with a credit line when the channel shows credits on screen.
- `CTA` is an original like-and-subscribe with the channel's real name and avatar.
- The transition kit: the in and out transitions, the end dip to black and the dissolve between back-to-back inserts, with lengths and styles from `theme.ts`, rendered into the insert. Their sounds are separate files for the SFX track, except in an overlay, which carries its own audio.

## Rendering: always on the GPU

- Chromium renders frames on the GPU through ANGLE, on Windows and Mac alike (Direct3D 11 on Windows): `--gl=angle` with Chrome for Testing (`chromeMode: 'chrome-for-testing'`). Remotion's default, Chrome Headless Shell, couldn't even show its GPU status in testing. `npm run gpu` passes `--chrome-mode=chrome-for-testing --gl=angle` itself, because `remotion gpu` ignored the config's GL setting. It should report "Hardware accelerated" for Compositing, Rasterization and WebGL.
- Encoding runs on the GPU with `--hardware-acceleration=required`:
  - On Windows, that's NVENC, which Remotion supports only for NVIDIA cards and only for H.264/H.265. Remotion's FFmpeg needs NVIDIA driver 551.76 or newer (NVENC API 12.2). The render script checks the driver first and stops with a clear message if it's older. Then ask the user to either update the driver (walk them through it click by click) or OK a CPU encode (`--hw disable`), and say which one ran. Without NVENC (AMD or Intel graphics, or an NVIDIA chip without an encoder), the hardware check has already set CPU encoding, so there's nothing to ask.
  - On a Mac, that's VideoToolbox, Apple's built-in encoder, for H.264, H.265 and ProRes. There's no driver to check.

  Set quality with `--video-bitrate`, because hardware encoders don't accept `--crf`.
- Alpha overlays for the editing software use ProRes 4444 (`--image-format=png --pixel-format=yuva444p10le --codec=prores --prores-profile=4444`). Frames still render on the GPU. On Windows the encode runs on the CPU, because NVENC can't encode ProRes; it's the one exception there, so say so whenever you use it. On a Mac, VideoToolbox encodes ProRes: use `--hardware-acceleration=if-possible`, so it falls back to the CPU if it won't take the alpha channel. These files are large, so budget time and disk.
- Split renders save most of that for full-frame cards (`npm run render:split`). A card renders in three parts, placed back to back on one track: a head in ProRes 4444 with alpha, holding the entrance until the card covers the shot; an opaque H.264 body, encoded on the GPU; and a ProRes 4444 tail, holding the exit. An overlay stays a single ProRes 4444 file. ffprobe checks every part's frame count.
- Any video can also go out as a timeline, the professional output: the Premiere `.xml`, the Final Cut `.fcpxml` and the `.otio` (see Editing a recording), with the scenes at their start times over the voiceover when there is one, and a marker on each. It's the default for a recording. For other videos, offer it when the user finishes in Premiere, Final Cut or Resolve, and make it whenever they ask.
- `angle` can leak memory on long renders. Keep each composition to one scene, and split with `--frames` if one runs long.
- Never put a live CSS `filter: blur()` on a large photo: rendered on the GPU across parallel tabs, it flashes single white frames and black flicker. Pre-blur each background once (cached copies at 2560 px or less), crossfade from sharp to blurred by opacity, and hold each frame until its images are decoded.
- The hardware check sets the concurrency (how many frames render at once) for this machine. If a render runs out of memory or the computer slows to a crawl, halve it, save the new value in `render-settings.ts` and under "This machine" in `GUIDELINES.md`, and render again.
- Every final render takes a machine-wide render lock (a lock file in the system's temp folder) in `render` and `render:split`, so no two overlap, whichever agent starts them. A recording's inserts render in one batch at the end. Stills run at most two at a time.
- Every render run bundles the project, and bundling copies all of `media/`. Three things keep that cheap:
  - The `render` script bundles once per run and accepts several IDs.
  - Raw recordings live in `recordings/`, never in `media/`: a 4 GB recording there is copied on every render.
  - When a video ships, move its `automated-research/<video>/` to `archive/<slug>/<video>/`. Suggest the same for the user's large files, but never move them yourself.
- Render flags live in one module, `core/lib/render-settings.ts`, read by both `remotion.config.ts` and the render script, because Remotion's Node APIs don't read the config file.
- Resolution and fps come from `CHANNEL.md` (or, for a cut recording, from the recording), so renders match the user's timeline.
- Everything runs through npm scripts, so nobody has to remember flags: `studio`, `render`, `render:alpha`, `render:split`, `still`, `qa`, `capture`, `clip`, `voice`, `transcribe`, `cut`, `master`, `assemble`, `timeline`, `gpu`. Renders go to `out/<slug>/<video>/<composition-id>.<ext>`, outside `media/`, so they never get bundled.
- **QA before anyone sees a render:**
  - `npm run qa` scans every frame's brightness (FFmpeg `signalstats`, YAVG). It flags any single-frame jump or drop of more than about 40 (8-bit) against both neighbours, and any all-white or all-black frame that isn't at an edge. A structure pass, comparing each frame's detail with its neighbours', catches the blank frames a brightness scan misses on white pages. Alpha files are scanned over mid-grey.
  - Look at every insert at its start, middle and end: cropping, empty space, text size, marker placement.
  - Look for clipped letters: count the ink pixels in each column at a text box's edge. A jump from 0 to 20 or more in a single column is a letter cut off (see Inserts → Design).
  - Check its sync against the timing table.
- An MP4 with sound lands its audio about 43 ms late (AAC priming with no edit list): trim or flag the priming samples when muxing.
- After a reboot or a usage limit, probe every render (ffprobe duration) before trusting it, and redo only what's missing or broken.

## Hardware check

The first step of setup. It takes seconds, needs nothing from the user, and fits everything below to this machine.

**What to check** (built-in commands, nothing to install): the CPU (model, cores, threads), RAM (total and free), every GPU (model, memory, driver), free space on the system drive and on the project's drive, and whether it's a laptop.
- Windows, in PowerShell: `Win32_Processor`, `Win32_ComputerSystem`, `Win32_OperatingSystem`, `Win32_VideoController` and `Win32_Battery` (a laptop has one). For NVIDIA cards, read VRAM and the driver version from `nvidia-smi --query-gpu=name,memory.total,driver_version --format=csv`, because `Win32_VideoController` reports 4 GB at most.
- Mac: `sysctl` (`machdep.cpu.brand_string`, `hw.physicalcpu`, `hw.logicalcpu`, `hw.memsize`), `uname -m` (`arm64` is Apple Silicon), `system_profiler SPDisplaysDataType` for the GPU, `pmset -g batt` (a laptop shows a battery) and `df -h`. Apple Silicon shares its memory between the CPU and the GPU, so the RAM total is the GPU's budget too.

**What it decides:**
- **Video encoding**
  - Windows with NVIDIA, driver 551.76 or newer: NVENC (`--hardware-acceleration=required`). A few entry-level chips, like the GT 1030, have no encoder; the test render shows it, and they encode on the CPU.
  - Windows with NVIDIA and an older driver: walk the user through the update right away, so it happens while the rest installs. Encode on the CPU until then.
  - Windows with only AMD or Intel graphics: frames still render on the GPU, but encoding runs on the CPU (`--hardware-acceleration=disable`).
  - Mac: VideoToolbox (`--hardware-acceleration=required`, and `if-possible` for ProRes 4444 overlays).
- **Voice model** (see Voice and transcripts)
  - NVIDIA RTX 30-series or newer with 8 GB of VRAM or more: the 1.7B models in bf16, with tone instructions and voice design.
  - Other NVIDIA cards: the 0.6B models on the GPU, in bf16 on an RTX 30-series or newer and fp32 on older cards.
  - Any Apple Silicon Mac: the 1.7B models on the GPU (PyTorch's `mps` device). Test one line first; if `mps` misbehaves, use the 0.6B models on the CPU.
  - Anything else (AMD or Intel graphics, an Intel Mac): the 0.6B models on the CPU. It works, but slowly.
- **Transcription models** (see Voice and transcripts), the same size for Whisper and CrisperWhisper:
  - NVIDIA with 4 GB of VRAM or more, or any Apple Silicon Mac: `large` (Whisper `large-v3`).
  - NVIDIA cards with less: `medium`.
  - Everything else, and whenever it has to run on the CPU: `turbo` (Whisper `large-v3-turbo`).
- **Render concurrency** (how many frames Remotion renders at once)
  - The ceiling is the lower of the CPU's threads and one per GB of RAM beyond 4 GB kept for the system (16 GB allows 12).
  - Once the template demos exist, benchmark one at half, three quarters and all of that ceiling (`npx remotion benchmark <id> --concurrencies=<a>,<b>,<c> --frames=0-89`), and save the fastest in `core/lib/render-settings.ts`.
- **Memory:** under 16 GB of RAM, run one heavy job at a time (a render, a voiceover, a transcription).
- **CPU:** with 4 cores or fewer, anything on the CPU is slow (encodes without a hardware encoder, alpha overlays on Windows, transcriptions, voice without a GPU). Give the user a rough time before long jobs.
- **Laptop:** ask the user to keep it plugged in for renders and voiceovers. Most Windows laptops slow the CPU and GPU down on battery, and long renders drain any laptop fast.
- **Disk:** the tools and models take roughly 15 GB, and up to about 25 GB once all the 1.7B voice models are in. Part of that is on the system drive, where pip and Hugging Face keep their caches. Renders need more, and ProRes 4444 overlays are large. If space is tight, suggest another drive, or move the caches there, before installing.

**Then:**
- Tell the user in two or three plain lines what their machine is good at and what will be slower. For example: "Your <graphics card or chip> will do the heavy lifting, so renders and voiceovers will be quick. Memory is on the small side, so I'll run one big job at a time." No spec lists unless they ask.
- Write the findings and choices into `GUIDELINES.md` under "This machine", so later chats don't check again.
- Check again when the hardware or driver changes, or when something runs out of memory. A project set up before this check existed gets it before its first render, voiceover or transcription.

## Setup (one-time, Windows or Mac)

For the user, setup is a single request. Claude does every step below and only stops for install approvals and the user's decisions (which channel, its brand). Steps that don't depend on each other run side by side, with the big downloads started first (see Keep it fast and smooth).

1. Run the **Hardware check** and tell the user in two or three plain lines what it means for them.
2. Install the tools, and ask before installing:
   - Windows, with winget: Node LTS (`OpenJS.NodeJS.LTS`), FFmpeg (`Gyan.FFmpeg`), yt-dlp (`yt-dlp.yt-dlp`), Deno (`DenoLand.Deno`) and Python 3.12 (`Python.Python.3.12`).
   - Mac, with Homebrew: `node`, `ffmpeg`, `yt-dlp`, `deno` and `python@3.12`. If Homebrew is missing, installing it needs the user once: give them the one line from brew.sh to paste into Terminal, and tell them it asks for their Mac password and installs Apple's command line tools too.

   Deno is the JavaScript runtime yt-dlp needs for YouTube (or have the clip script pass `--js-runtimes node`).
3. Install Remotion (latest, TypeScript, blank) in the chosen folder:
   - Core: `remotion`, `@remotion/cli`, `react`, `react-dom`
   - Extras: `@remotion/bundler`, `@remotion/renderer`, `@remotion/google-fonts`, `@remotion/fonts`, `@remotion/layout-utils`, `@remotion/transitions`, `@remotion/media-utils`, `@remotion/shapes`, `@remotion/paths`, `@remotion/noise`, `@remotion/captions`, `@remotion/zod-types`, and the zod version `@remotion/zod-types` is built against
   - Dev dependencies: `typescript`, `tsx`, `playwright` (then `npx playwright install chromium`), `@mozilla/readability`
   - Remotion's official agent skill: `npx skills add remotion-dev/skills --skill remotion-best-practices -a claude-code -y`. The animator and editor agents preload it, so they use current Remotion APIs.
   - The `tubeai-mcp` skill, if it isn't installed yet (it comes with the `tubeai` plugin): `npx skills add tubeai-app/tubeai-skills --skill tubeai-mcp -a claude-code -y`. The researcher agent preloads it.

   Every `@remotion/*` package must be on exactly the same version as `remotion`.
4. Set up the Python environment in `.venv/`:
   - PyTorch: on Windows, the CUDA build when the hardware check found an NVIDIA GPU (confirm with `torch.cuda.is_available()`), otherwise the CPU build. On a Mac, the standard build, which on Apple Silicon includes the `mps` GPU device (confirm with `torch.backends.mps.is_available()`).
   - `qwen-tts`. It pins its own `transformers` version, which is one reason it gets its own environment.
   - `crisperwhisper[transformers]`, for transcripts, with Whisper running through the same `transformers`. CrisperWhisper only needs `transformers` 4.40 or newer, so it shares this environment with `qwen-tts`; if their versions ever clash, give it its own. Its faster CTranslate2 runtime only exists for Linux.
   - `opentimelineio` with its Final Cut Pro 7 XML and Final Cut Pro X adapters (`otio-fcp-adapter`, `otio-fcpx-xml-adapter`), to read every exported timeline back as a check.
   - The voice model size, device and precision from the hardware check. Download only its CustomVoice model now, in the background; Base and VoiceDesign download the first time they're needed. FlashAttention is optional.
5. Download the transcription models the hardware check picked, in the background, then transcribe a 10-second test clip. They should run on the GPU where there is one (CUDA on NVIDIA, `mps` on Apple Silicon), and on the CPU otherwise.
6. Create the layout above. The entry point `core/index.ts` registers `core/Root.tsx`, which mounts a `<Folder>` per channel plus a `templates` folder of demos.
7. In `core/lib/render-settings.ts`, which `remotion.config.ts` and the render script both read, set the public dir to `media`, `angle`, Chrome for Testing, the hardware acceleration the hardware check picked, and the video bitrate. The concurrency comes in step 9, once there's a template demo to benchmark.
8. Write the scripts in `core/scripts/`. None of them write into `user-provided/`. `capture` and `clip` log their files in `SOURCES.md` when given `--note` (and `--scene`).
   - `capture`: saves a URL as a PNG at 2× device scale, either the full page or a CSS selector, in light or dark mode, with cookie and consent overlays hidden. It also saves the article text as JSON (Readability).
   - `clip`: runs `yt-dlp --download-sections "*<start>-<end>" --force-keyframes-at-cuts -S "vcodec:h264,res,acodec:m4a" --merge-output-format mp4 -o <path> <url>`. H.264 MP4 cuts cleanly into an edit. Drop the h264 preference when 4K is needed. Without a section, it downloads the whole video (for style references). When a download fails, it loops through YouTube clients (see Troubleshooting) and prints which one worked and the resolution.
   - `voice`: Qwen3-TTS through the `.venv` Python, at the model size the hardware check picked. Options: `--speaker`, `--speed`, `--samples` (one line per preset speaker), `--instruct "…"` (tone, 1.7B only), `--design "…"` (a new voice from a description, 1.7B only), `--clone <ref.wav> --ref-text "…"` (the Base model; a real person's voice only with their permission).
   - `transcribe`: runs Whisper (prompted with the glossary) and CrisperWhisper's verbatim pass through the `.venv` Python on a 16 kHz mono WAV with word timestamps, and writes `transcript.json` (Remotion captions: `text`, `startMs`, `endMs`) and `timings.json`, with names and terms corrected against the glossary. It can also transcribe a short window, with or without a prompt, or `forced_align` it, for checking sentences and joins.
   - `cut`: finds the stumbles (see Cutting) and diffs the transcript against the script → a draft `cuts.json` (the removed ranges, each with its reason, and the word fixes) and `CUTS.md`, with every cut placed in the audio's silence. Its shared library in `core/scripts/lib/` turns `cuts.json` into the kept segments at the recording's exact frame rate, `words-cut.json` and a reference WAV of the cut, and re-transcribes about 2 s either side of every join.
   - `master`: the voice master (see Sound). It measures the cut, sets the make-up gain, writes the WAV, and reports its loudness, true peak and loudness range, and the moment where the limiter works hardest.
   - `assemble`: a video's windows (`insert-windows.json` for a recording, the `timings.json` beats for a voiceover), its renders, the moves and the voice master → the timeline spec, with the empty colour track, the held overlays and the effects' gain. It then runs `timeline`, and every check in the Timeline recipe on what it wrote.
   - `timeline`: a timeline spec (the kept segments of the recording with their moves, the voice master, the inserts with their clip gain, the sound effects and the markers) → Final Cut Pro 7 `.xml` (Premiere), `.fcpxml` (Final Cut Pro) and `.otio` (Resolve), written directly and following the Timeline recipe below. It then reads the `.xml` and `.fcpxml` back through OpenTimelineIO and checks the clip counts, positions and total length.
   - `render`: bundles once per run, renders each ID with the settings from `render-settings.ts` under the render lock, and, when encoding with NVENC, checks the NVIDIA driver first. `render:split` renders full-frame cards as head, body and tail (see Rendering).
   - `qa`: the flash scan and the structure pass on a render (see Rendering), over mid-grey for alpha files.
9. Build the core templates and their demos, then benchmark the concurrency on one of them (see Hardware check).
10. Create the channel(s) the user names (see Channels). Brand work needs the user's input, ideally with a style reference.
11. Write the three agent files below into `.claude/agents/`. In `video-researcher.md`, name the TubeAI skill exactly as the skill list shows it: `tubeai:tubeai-mcp` when it came with the plugin, `tubeai-mcp` otherwise.
12. Write the docs:
    - `README.md` for the user, in plain language with click-level steps. Claude normally runs everything, so this is only for doing it by hand; keep it short. It covers connecting TubeAI (in the Claude app, or with one line in the terminal), previewing with `npm run studio` (http://localhost:3000), one example each for render, alpha, still, capture, clip, voice and transcribe, where to put media, how to share a style reference (paste a YouTube link in the chat, or give the path to a video file), where to put a recording to edit (`recordings/<slug>/`), how to open the exported timeline in Premiere, Final Cut or Resolve, and where renders go.
    - `GUIDELINES.md`: these sections of this skill, adapted to what was actually installed and verified: The user isn't technical, Keep it fast and smooth, Workflow through Rendering, and Troubleshooting. Add a "This machine" section with the hardware check's findings and choices, and whether the user declined TubeAI, so later chats don't offer it again.
    - `PROJECTS.md`, from the template in the PROJECTS.md section, with the channel(s) just created and no videos yet.
    - `CLAUDE.md`, containing `@GUIDELINES.md` and `@PROJECTS.md`, each on its own line.
13. Verify each of these, then tell the user in plain language what works and what, if anything, needs them:
    - tool versions and the `npm run gpu` result
    - the TubeAI connector, if the user connected it: a `workspace_account` call answers with the user's plan
    - a template render with the encoding the hardware check picked; confirm NVENC in `nvidia-smi`, or VideoToolbox in the render's verbose log (`--log=verbose`)
    - a real news article through both Claude in Chrome (if connected, it opens and screenshots the page) and `npm run capture` (saves PNG + JSON)
    - a 10-second yt-dlp test clip, noting which client worked and at what resolution
    - a short `npm run voice` line, transcribed back with `npm run transcribe`: the words should come back right
    - a two-cut test: a short clip with two cuts, one move, one stand-in insert and its voice master, through `npm run assemble`. Every check passes, and the checks bite: they fail with the insert shifted by one frame, and again with V1 off by one frame

    If anything fails, say so and include the error.

## Troubleshooting

Fix these without the user where possible, telling them in a line what's happening as you go. When one needs the user (like a driver update), give click-level steps.

- **YouTube downloads fail with HTTP 403.** The list of qualities loads, but the download itself is refused. YouTube asks some of the players yt-dlp imitates for a PO ("proof of origin") token, to prove the request comes from a real player. The clip script loops through players until one works:
  1. yt-dlp's defaults
  2. `web_embedded`: full quality, but only for videos whose owner allows embedding
  3. `tv`
  4. `mweb`, then `android`: usually 360p only, so check the printed resolution and tell the user

  Which players work changes whenever YouTube changes its checks. The lasting fixes are keeping yt-dlp updated (`yt-dlp -U`) and, if a PO-token plugin such as bgutil is installed, running the token server it relies on. The plugin does nothing without it.
- **TubeAI problems** (signing in, missing tools, the daily budget): see the `tubeai-mcp` skill.
- **NVENC won't start on Windows** ("The minimum required Nvidia driver for nvenc is 551.76 or newer"): the driver is too old. See Rendering.
- **A Mac render fails with `required` hardware acceleration:** VideoToolbox won't take that codec or format on this Mac. Use `if-possible` for it, and note it under "This machine".
- **`npm run gpu` reports software rendering:** check that it passes `--chrome-mode=chrome-for-testing --gl=angle`.
- **Claude in Chrome isn't connected:** ask the user to connect the extension before relying on it for logged-in or paywalled pages.
- **Single white frames or black flicker in a render:** a live CSS blur on a large photo. See Rendering.
- **Timeline clips run past the end of a render:** its audio stream is longer than its picture (AAC priming). Take lengths from the video stream's `nb_frames`.
- **Every render is slow and the disk fills up:** a raw recording sits inside `media/`, so every render copies it. Recordings belong in `recordings/`.
- **Letters cut off at the edge of a card** (the left of a first C, G, O or S, the ends of T and W): big type nudged left inside an `overflow: hidden` box. Clip only the axis the animation needs (see Inserts → Design).
- **A word with digits changes mid-animation** ("WIG20" shows "WIG10"): a rolling-number effect took it for a number. Words that contain digits land static.
- **A re-rendered insert won't replace the old file on Windows:** Premiere has it open. Ask the user to close the project in Premiere, render again under the same name and frame count, then reopen it.

## Timeline recipe (Premiere XML, Final Cut FCPXML and OTIO)

The `timeline` script writes three files from one timeline spec: a Final Cut Pro 7 `.xml` for Premiere Pro, an `.fcpxml` for today's Final Cut Pro, and an `.otio` for DaVinci Resolve. The Premiere recipe, moves and voice master included, was confirmed in Premiere Pro 2025 (File → Import). The Final Cut recipe hasn't been through a real import yet, so ask the first Final Cut user to confirm it, and fix the recipe from what they report.

**Premiere Pro (Final Cut Pro 7 XML)**

- **Why Final Cut Pro 7 XML** (`xmeml` version 4): Premiere's own `.prproj` isn't documented, while FCP7 XML imports as a ready sequence in every Premiere version. OTIO import only exists in recent Premiere builds; Resolve reads OTIO.
- **Sequence:** `<rate>` with `<timebase>` and `<ntsc>` (29.97 = timebase 30, NTSC TRUE; 23.976 = 24, TRUE), a `<timecode>`, a video `<format>` with the recording's width and height, and audio `<numOutputChannels>2` with an `<outputs>` block of two mono groups.
- **Files:** define each `<file id>` once in full (name, `pathurl`, rate, duration, a timecode, and a media block with the video and audio sample characteristics and channel count), then refer to it as `<file id="…"/>`.
- **Paths** are absolute, so the editor asks to relink if files move. On Windows: `file://localhost/C%3a/Users/...`, with the drive's colon as `%3a` and each path segment URI-encoded, as Premiere writes it. On a Mac: `file://localhost/Users/...`, each segment URI-encoded.
- **Clip items:** `start`/`end` are timeline frames, `in`/`out` are source frames, and `duration` is the source file's total length. Set `masterclipid` per file, and link each video item to its audio items with `<link>` entries (`linkclipref`, `mediatype`, `trackindex`, `clipindex`, plus `groupindex` for audio), so they move together.
- **Stereo audio**, the way Premiere exports it: two "exploded" mono tracks. Each track carries `premiereTrackType="Stereo"`, `currentExplodedTrackIndex` (0 or 1), `totalExplodedTrackCount="2"` and its `<outputchannelindex>`. Each clip item carries `premiereChannelType="stereo"` and a `<sourcetrack>` with `trackindex` 1 or 2. Premiere rebuilds them as one stereo track.
- **Markers:** sequence markers (`<marker>` with `<name>`, `<comment>`, `<in>`, `<out>-1`), one per insert and one per move, plus a clip marker on each insert, carrying the insert's ID and its cue line, so the editor can find their way around the edit.
- **Lengths come from the picture, not the container.** AAC priming (2048 samples, about 43 ms at 48 kHz) makes a render's audio a few frames longer than its video. Use the video stream's `nb_frames`, or out-points run past the last frame.
- **Cuts as kept segments:** every kept segment of the recording sits on V1, back to back, each linked to the original file with its in and out points, so every cut stays a normal edit point the editor can roll open. In the spec: `"main": { "file", "segments": [{ "in", "out" }], "audio", "moves" }`, with `audio` the voice master and `moves` the moves on the speaker. For a video with no recording, `main` is the voiceover as one audio-only segment, or is left out.
- **Moves** go in FCP7's `Basic Motion` filter on the V1 clip items (see Moves on the speaker):
  - `scale` is a percentage of the frame-filling size. `center` is the recording's centre as normalised offsets: `horiz` = px ÷ sequence width and `vert` = px ÷ sequence height, with 0, 0 centred.
  - A cut to a new framing is a through-edit: split the V1 clip at that frame, so both halves continue the same source, and give the second half its own static Scale and Center.
  - An eased move is one linear keyframe on every frame, along Premiere's Ease In/Out curve (Bezier at 33.33% influence: cubic-bezier(1/3, 0, 2/3, 1), which is 3p² − 2p³). That's exact in any editor, though fiddly to adjust by hand. `<keyframe><when>` counts in the clip's source frames (its in point plus the offset), as Premiere's own FCP7 export writes it.
  - Two keyframes with Bezier handles (`<inbez>`/`<outbez>`) should work for Scale, but FCP7 keeps Center's 2D ease in `<inscale>`/`<outscale>`, which Premiere's importer probably doesn't read. Both are untested in Premiere, so write the per-frame keyframes.
  - A move across a V1 join is split across the two clips, with the same value on both sides of the join.
  - Read the Motion values back from the XML and compare them with the plan: they should match to within 0.00005% in scale and 0.001 px in position.
- **The voice master** is double-system sound: its own `<file>` on A1/A2, each clip item cut exactly like its V1 clip (the same in and out points, at offset 0) and linked to it.
- **Clip gain** is Premiere's Audio Levels filter on the clip item (`<effectid>audiolevels</effectid>`). Its `level` parameter is a linear gain: +4 dB is 1.584893, and the maximum is about 3.981 (+12 dB). In the spec, it's each insert's `level`, in dB.
- **Sound effects** are clips of their own on the SFX tracks, each at its frame.
- **Track layout,** from the bottom up:
  - V1: the speaker, with the cuts and moves. The voice master is on A1/A2.
  - V2: empty, for the editor's colour adjustment layer (see Colour).
  - Next, the overlays that hold under a card.
  - Then the full-frame cards. A card that dissolves in sits on the track right above the one it dissolves from, and a card that would overlap another moves up a track.
  - Above the cards, the overlays (lists, lower thirds, CTA, tags), with the end dip to black on top.
  - Audio, above A1/A2: the overlays' own audio, each on the next free stereo pair, then the sound effects on a stereo pair of their own.

  No empty tracks except V2, and no overlaps on any track. With no recording, V1 holds the full-frame scenes at their start times, over the voiceover on A1/A2 when there is one, and there's no colour track.

**Final Cut Pro (FCPXML)**

Final Cut Pro 10 and 11 can't open the Final Cut Pro 7 XML above. They read FCPXML, Apple's documented format (File → Import → XML).

- **Version:** write one from a few years back, such as 1.10. Newer Final Cut reads older FCPXML, not the other way round. A plain `.fcpxml` file is enough; no `.fcpxmld` bundle is needed.
- **Structure:** `<fcpxml version>` → `<resources>` (one `<format>`, and one `<asset>` per file) → `<event>` → `<project>` → `<sequence>` → `<spine>`.
- **Format:** `frameDuration` from the frame rate (23.976 = `1001/24000s`, 24 = `100/2400s`, 25 = `100/2500s`, 29.97 = `1001/30000s`, 30 = `100/3000s`, 59.94 = `1001/60000s`), plus `width` and `height`. Use Apple's standard format name when there is one (such as `FFVideoFormat1080p2997`); otherwise leave the name out.
- **Time** is always rational seconds on frame boundaries (a whole number of `frameDuration`s, such as `3003/30000s`), never decimals, so NTSC rates stay exact.
- **Assets:** `start` is the file's own start timecode (`0s` when it has none), and every in-point is measured from it. Add `duration` (from the picture, as above), `hasVideo`, `hasAudio`, `audioSources`, `audioChannels`, `audioRate` and `format`, and a `<media-rep kind="original-media" src="file:///…"/>` with the absolute path, each segment URI-encoded. Final Cut asks to relink if files move.
- **The cut:** every kept segment is an `<asset-clip>` on the spine, back to back. Its `offset` is its position in the timeline, `start` its in-point and `duration` its length. The spine is Final Cut's main storyline, so every cut stays a normal edit the editor can adjust.
- **Inserts** are connected clips: an `<asset-clip>` nested in the spine clip under its first frame, with `lane="1"` for full-frame inserts and higher lanes for overlaps and overlays. A connected clip's `offset` is in its parent's local time: `offset = parent's start + (the insert's timeline position − parent's offset)`. Getting this wrong shifts every insert.
- **Markers:** a `<marker>` inside each insert's clip, with `start` at the clip's own `start`, one frame long, the insert's ID as `value` and its cue line as `note`.
- **Sound effects** are connected clips below the spine (negative lanes), and clip gain is an `<adjust-volume amount="…dB"/>` inside the clip.
- **Not mapped yet:** the moves on the speaker and the voice master. Until a Final Cut user has tested them, Final Cut gets the cut with the recording's own audio, and no moves.
- **Sequence:** `format`, the total `duration`, `tcStart="0s"`, `tcFormat="NDF"`, `audioLayout="stereo"` and `audioRate="48k"`. With no recording, the voiceover is the spine (or a `<gap>` when there isn't one), and the scenes connect to it the same way.

**All three**

- **Validate every export:** read the `.xml` back with OpenTimelineIO's Final Cut Pro 7 XML adapter and the `.fcpxml` with its Final Cut Pro X adapter (independent implementations), and check the clip counts, positions and total length.
- **The checks on every timeline**, all passing before delivery (a video without a recording skips the ones about it):
  1. The sequence's name and size, and one frame rate for every clip and file.
  2. V1 equals the cut list once the through-edits are joined back, and it ends on the last insert's frame.
  3. A1 and A2 are cut exactly like V1, with the voice master at its offset.
  4. Every insert is on its frame, with the right file, length and alpha, and its parts back to back. There's one insert per timing-table window.
  5. Starts, ends, roles and exits match the timing table, and each dissolving card is right above the one it dissolves from.
  6. The tracks stack as planned (the empty colour track, held overlays, cards, overlays, the end dip), with no overlaps on any track.
  7. Overlays carry their audio at the set gain, card parts carry none, and the sound effects are on their frames.
  8. There's one marker per insert and per move, and the moves read back from the XML match the plan.
  9. The `.otio` has the same clips.
  10. The checks in Sound pass.
- **Prove the checks bite,** at setup and whenever they change: run them once with an insert shifted by one frame, and once with V1 off by one frame. Both runs must fail.
- **Test the assembly before the renders exist,** with tiny stand-in files at the planned frame counts.
- **OTIO** has no schema for keyframes, so Resolve gets the cut without the moves. Its pitfalls: `Marker.1` uses `range`, while `Marker.2` uses `marked_range` plus `comment`. Use `Clip.1` with `media_reference` for older readers.

## Agent files

`.claude/agents/video-researcher.md`:

```markdown
---
name: video-researcher
description: Finds and captures media for one scene of a channel video (news articles, X/Reddit posts, data, images, YouTube and other web video clips), or pulls the style out of a reference YouTube link or video file, into the channel's automated-research folder, with sources.
model: claude-opus-5-5
effort: xhigh
skills:
  - tubeai-mcp
---
You research media for one scene of a YouTube video in this Remotion project, or pull the
style out of a reference video.

First read in full: README.md, GUIDELINES.md, and channels/<slug>/ (CHANNEL.md, theme.ts,
animations/, and, for a video, videos/<video>/ with BRIEF.md and existing scenes). List
media/<slug>/ and open only the files you need.

Then do what the main chat asks, following GUIDELINES.md → Media and Style references, and
the tubeai-mcp skill for anything TubeAI:
- Write media only to media/<slug>/automated-research/ (<video>/ for a scene, style-refs/ for
  a style reference). Never touch user-provided/.
- Articles, posts and documents: save the real text and data as JSON for our templates, plus
  a 2× reference screenshot (npm run capture). Documents get rebuilt as text, never shown as
  screenshots. Read paywalled or bot-walled pages only through the user's signed-in browser
  (Claude in Chrome): never work around a paywall.
- Figures: check each one against a primary source, and report any that are imprecise or
  unsupported, with options and a recommendation.
- Photos: log each one's credit in SOURCES.md.
- YouTube: find the video first through TubeAI when it's connected (the tubeai-mcp skill has
  the playbooks). Every call counts against the user's daily TubeAI budget, so make few,
  well-aimed ones. Then download only the needed section
  (npm run clip). If it fails with 403, follow
  GUIDELINES.md → Troubleshooting.
- Style references: contact sheets and frame measurements, then the findings in CHANNEL.md
  under "Style reference".
- Log every file in SOURCES.md. Never invent or alter quotes, posts, headlines or figures,
  and flag anything you couldn't verify.

Report back: each file with a one-line description, your recommended picks (or the style
findings), gaps and questions.
```

`.claude/agents/video-animator.md`:

```markdown
---
name: video-animator
description: Builds and renders scenes and reusable channel animations in Remotion, reusing the channel's theme, its animations and the core templates, matching any style reference and syncing to timings.json.
model: claude-opus-5-5
effort: xhigh
skills:
  - remotion-best-practices
---
You build one scene of a YouTube video, a group of a recording's inserts, or a channel's theme
and reusable animations, in this Remotion project.

First read in full: README.md, GUIDELINES.md, channels/<slug>/ (CHANNEL.md, theme.ts,
animations/, and, for a video, videos/<video>/ with BRIEF.md and existing scenes), and the
core templates you'll use. List media/<slug>/ and open only the files you need.

Then do what the main chat asks, following GUIDELINES.md:
- Reuse first, then derive, then create: use channel animations and core templates first,
  and make variants through props or wrappers. Something reused a second time goes to
  channels/<slug>/animations/ with a one-line CHANNEL.md catalog entry. No demos, mock sets
  or renders nobody asked for.
- Take brand values only from theme.ts, and load media only through staticFile() from media/<slug>/.
- If the brief has a reference video, or CHANNEL.md has a style reference for this kind of
  animation, match it and include side-by-side stills (the reference frame next to ours).
- Time the scene from its window: cut/insert-windows.json for a recording, timings.json for a
  voiceover. It starts on the start word, exits right after the end word (or holds or
  dissolves, as the window says), and every element lands on its beat word. Never hard-code
  a time.
- Follow GUIDELINES.md → Inserts: fill the card, nothing cropped at its edges, no box open
  before its content, numbered lists running down, light highlights behind dark text, no
  rolling words with digits, clipping only on the axis an animation needs, charts computed
  from the data (the build fails on a mismatch), and the channel's transitions baked in.
  Backgrounds crossfade to a pre-blurred copy: never a live CSS blur. Cards render silent:
  list each sound and its frame for the timeline.
- Register the composition as <CODE>-<video>-s<nn> and get `npx tsc --noEmit` clean. Render
  stills at its start, middle and end (npm run still, one at a time). A scene then gets its
  GPU render (npm run render, or render:alpha for an overlay) and npm run qa. A recording's
  inserts stop at the stills: their final renders run in one batch at the end.

Report back: files changed, composition ID, still and render paths, the QA result, each card's sounds and their frames, and anything the user should look at.
```

`.claude/agents/video-editor.md`:

```markdown
---
name: video-editor
description: Edits a creator's raw recording into a timeline for Premiere Pro. Transcribes it, finds every stumble, cuts only clear mistakes against the script, verifies every join, masters the voice, builds the timing table the inserts sync to, plans the moves on the speaker, and assembles and checks the timeline for Premiere Pro (XML), Final Cut Pro (FCPXML) and DaVinci Resolve (OTIO), with a review list.
model: claude-opus-5-5
effort: xhigh
skills:
  - remotion-best-practices
---
You edit one raw recording in this Remotion project into a timeline the user finishes in
their own editor.

First read in full: README.md, GUIDELINES.md, and channels/<slug>/ (CHANNEL.md with its
pacing and glossary, theme.ts, and videos/<video>/ with BRIEF.md). List recordings/<slug>/
and media/<slug>/, and open only the files you need. Never change the recording itself.

Then do what the main chat asks, following GUIDELINES.md → Editing a recording, Moves on the
speaker and Sound. If the transcriber or FFmpeg is missing, install it as you go; don't stop
to ask.
- Transcribe it (npm run transcribe): Whisper with the glossary for the words, and
  CrisperWhisper's verbatim pass for the stumbles. Correct names and terms against the
  glossary and the script. At a cut or a retake, check the audio too.
- Find every stumble: the verbatim pass against Whisper, each sentence re-transcribed with
  the verbatim-leaning prompt, and the audio where the text looks clean. Cut only clear
  mistakes (false starts, flubs, retakes) and notes to the editor said on camera, diffing
  the transcript against the script. Keep rants and ad-libs, and the pauses the host's policy
  in CHANNEL.md keeps; for a new host, measure their pauses and report back with a
  recommended policy before cutting. Place every cut by the silence in the audio, and verify
  every join by re-transcribing about 2 s either side. Write cut/cuts.json (only the
  decisions) and cut/CUTS.md.
- Export the cut-only timeline (npm run timeline) with its REVIEW.md, and check it reads back
  through OpenTimelineIO.
- Master the voice (npm run master), and build the timing table: cut/words-cut.json, and
  cut/insert-windows.json for the inserts in BRIEF.md, anchored on the words actually said.
- Plan the moves on the speaker around the inserts.
- Render nothing unless the main chat asks for it. When it asks for the full timeline, test
  the assembly with stand-in files, render every insert in one batch (render:split for
  cards, render:alpha for overlays), and run npm run assemble until every check passes.
  Then write REVIEW.md.

Report back: files written, the length before and after with cuts counted by reason, the
"your call" and "listen to these" lists, the voice master's loudness and true peak, any
check that failed, and the timeline's path.
```

## Keep the docs current

- `GUIDELINES.md` when a convention or the machine changes, including new Troubleshooting entries
- `CHANNEL.md` when the brand, voice, glossary, style reference or animation catalog changes
- `BRIEF.md` after every scene step (idea → researched → animated → rendered) and every decision
- `PROJECTS.md` whenever a channel or video is added, a video's status or next step changes, or a video ships
- `CUTS.md` whenever the cut changes, with each delivered version's `cuts.json` kept in `cut/versions/`
- `REVIEW.md` with every delivered timeline
- `README.md` when commands change
