# TubeAI for Claude Code

One Claude Code plugin for YouTube creators: a video editor, plus optional YouTube research from [TubeAI](https://tubeai.app).

## A video editor in Claude Code

Tell Claude what you want in your video, and it does the technical work: it sets up a video project on your computer, builds the animations, and cuts and renders. It's made for creators and editors who aren't technical.

- **Setup in one request.** Claude installs everything and fits the settings to your computer's processor, memory and graphics.
- **Your channel's look, every time.** Colors, fonts, logo and reusable animations are set once per channel. Share a YouTube link or a video file, and Claude matches its style.
- **Animated scenes.** Title cards, lower thirds, charts, montages, and news articles or X and Reddit posts rebuilt as sharp, animated cards, all built with [Remotion](https://www.remotion.dev).
- **Media, found for you.** Claude searches for what a scene needs and gathers it: clips from YouTube and other video sites (just the seconds you need), photos and videos from the web, news articles, documents, posts from X and Reddit, and the data behind charts. Every source is logged.
- **Voiceovers and transcripts** with Qwen3-TTS and CrisperWhisper, running on your own computer. Transcripts are verbatim, with every filler, stutter and retake in place, and word timings accurate to a few hundredths of a second. With strong enough graphics, the voice also takes tone directions, and Claude can design a new voice from a description.
- **Raw recordings, edited.** Claude cuts the false starts, flubs and retakes (your natural pauses stay), times every insert to the words it illustrates, and hands you a Premiere Pro timeline to finish, or an OTIO for DaVinci Resolve.
- **It remembers the work.** Every channel and video keeps its notes and progress, so a new chat picks up where you left off.

**You need** a Mac or a Windows PC, and Claude Code (the Code tab in the Claude desktop app, or the terminal). It's fastest on a Mac with Apple Silicon (M1 or later) or a PC with an NVIDIA graphics card, and still works, more slowly, on other machines.

## Install

In Claude Code, enter these two lines, one at a time:

```text
/plugin marketplace add tubeai-app/tubeai-skills
/plugin install tubeai@tubeai-skills
```

That installs everything. Then open Claude Code in an empty folder and say **"Set up my video project"**. Once it's ready, ask for things like:

- "Add my channel"
- "Match the style of this video: <YouTube link or file>"
- "Make an animated scene for this part of the script"
- "Edit this recording: <file>"

## Optional: connect TubeAI for YouTube research

The TubeAI connector opens TubeAI's database of 400M+ YouTube videos and 4M+ channels to Claude, so it can research your next videos with you:

- **Video ideas:** outliers (videos that far beat their channel's usual views) in your niche or on any channel, and what's taking off right now.
- **Packaging:** viral vectors (the title formats and words that keep outperforming), which words in your draft titles carry proven performance, and the thumbnails behind the outliers.
- **Channels:** any channel's best videos, and the channels most like it.
- **Transcripts** of any public video, to study how a hit is scripted.
- **Thumbnails:** generate a set in a channel's style, with your own face, then edit them.
- **Your TubeAI workspace:** save ideas to My Ideas, and channels and videos to folders.

It needs a free [TubeAI account](https://tubeai.app). Exploring is free within a daily limit. Thumbnails and a few other extras use TubeAI credits, and Claude asks before spending any. Some features depend on your plan.

Claude offers to connect it once. If you'd rather not, the video editor still works, Claude researches on the web instead, and it won't ask again.

**In the Claude app** (claude.ai, desktop or mobile; it then works in Claude Code too, with the same Claude account)

1. Open **Settings → Connectors** and click **Add → Add custom connector**.
2. Name it **TubeAI** and paste `https://beta.tubeai.app/api/mcp` as the server URL.
3. Click **Add**, then **Connect**, and approve on the TubeAI page that opens (sign in first if it asks).

**Or in the terminal** (Claude Code)

```bash
claude mcp add --transport http --scope user tubeai https://beta.tubeai.app/api/mcp
```

Then type `/mcp` in Claude Code, pick **tubeai** and sign in.

## Under the hood

The plugin is two skills that work together. Users never pick between them: Claude uses the right one for each request.

| Skill | What it does |
|---|---|
| `tubeai-video` | The video editor: sets up a Remotion project on a Mac or Windows PC, keeps each channel's branding and reusable animations, matches styles from a YouTube link or video file, researches media (articles, X/Reddit posts, YouTube clips), makes Qwen3-TTS voiceovers and CrisperWhisper transcripts, edits raw recordings into a Premiere Pro XML / OTIO timeline, and renders on the GPU. |
| `tubeai-mcp` | YouTube research through the TubeAI connector: video ideas and outliers, niches, competitors, titles, transcripts, thumbnails and script drafts, saved to the user's TubeAI workspace. The video editor uses it for ideas, scripts and finding clips when TubeAI is connected. |

## Tutorial: from setup to a finished video

You talk to Claude in plain words, and it runs every command. Here's how a project goes, and what to ask at each step.

### 1. Set it up (once)

Open Claude Code in an empty folder and say **"Set up my video project"**.

- Claude checks your computer, lists everything it will install, and asks once. Windows asks for permission a few times (click **Yes**); a Mac asks for your password.
- While things install, it asks about your channel: its name, logo, colors and fonts, and a YouTube link or video file with the look you want.
- It proposes the channel's look and a starter set of animations (title card, lower third and so on), and builds them once you say yes. For voiceovers, it plays sample voices so you can pick one by ear.
- It offers to connect TubeAI once. Say no if you don't need it.

### 2. Make a video

**Scenes for your own edit:** describe the scene, or paste that part of the script ("Make a scene for this part: …"). Claude finds the media (articles, posts, clips), builds the scene in your channel's style and sends you stills. With no recording, it can also voice the script in the channel's voice and time the scenes to it.

**A raw recording:** put it in the project's `recordings/<channel>/` folder (or give Claude its path), with the script if you have one, and say **"Edit this recording"**.

1. Claude transcribes it and proposes the inserts (the cards, charts, clips and graphics that go on top), each with the sentence it goes on. Approve the list, and add or drop anything.
2. It cuts the false starts, flubs and retakes, keeps your natural pauses, and sends a cut-only timeline you can check in Premiere while it builds the inserts.
3. It builds the inserts, each timed to the words it illustrates.

### 3. Preview everything before rendering

Ask Claude to **open the preview**, or to run `npm run studio`. Remotion Studio opens in your browser at http://localhost:3000, with every scene and template, and the media Claude gathered. Play them, scrub through frame by frame, and check everything before anything gets rendered.

### 4. Ask for changes

Say what to change, in plain words: "make the title bigger", "a slower fade-in", "use the second photo", "bring the chart in when they say the number". Claude edits the scene, and the preview updates by itself. Repeat until it's right, or share a YouTube link to match a specific look. A correction that applies to the whole channel becomes one of its rules, so you only make it once.

### 5. Render

When you're happy, say **"Render it"**. Claude renders, checks every file before handing it over, and tells you where each one is, in the project's `out/<channel>/<video>/` folder:

- **Scenes:** an MP4, or a transparent overlay to drop into your editor.
- **The professional output:** if you finish videos in Premiere Pro or DaVinci Resolve, ask for the timeline. You get an `.xml` for Premiere (File → Import) and an `.otio` for Resolve (File → Import → Timeline), with every scene already on its track at the right moment and a marker on each. For a recording, it's what you get by default, with every cut still adjustable; ask if you also want an MP4.

### 6. Pick up where you left off

Open Claude Code in the project folder and say what you want to work on ("Let's continue the video about …"), or just ask **"What's next?"**. Claude knows where every channel and video stands, sums it up and suggests the next step.
