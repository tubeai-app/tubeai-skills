---
name: tubeai-mcp
description: YouTube research through the TubeAI connector (MCP). Explores 400M+ YouTube videos and 4M+ channels to find video ideas and outliers, research niches and competitors, work out titles, study transcripts and draft scripts, makes thumbnails, and saves ideas and folders to the user's TubeAI workspace. Helps the user connect and sign in when they want it. Use when the user wants video ideas, what's working in a niche or on a channel, competitor research, title or thumbnail help, a transcript or a script draft, or mentions TubeAI.
---

# TubeAI connector

TubeAI connects to Claude as a connector (an MCP server) at `https://beta.tubeai.app/api/mcp`. It gives Claude a database of 400M+ YouTube videos and 4M+ channels, plus the user's own TubeAI workspace: ideas, folders, boards and past work. Use it to explore YouTube data, find video ideas and draft scripts with the user.

Assume the user isn't technical: do the technical parts yourself, give click-level steps when they have to act, and talk in plain language.

Once connected, TubeAI sends its own instructions and a full description of every tool; follow those for each tool's exact inputs. This skill covers getting connected, which tools to use for each job, and the rules that keep the user's account safe.

## Connect first

TubeAI is optional, but this skill runs on it. Before the first TubeAI task in a session:

1. Check the connection with a `workspace_account` call. When it answers with the user's plan, TubeAI is connected.
2. If it isn't, offer it in a line: what it gives them (video ideas, packaging and what's working in their niche) and that it needs a free TubeAI account. If they want it, walk them through one of the two ways:
   - **In the Claude app** (claude.ai, the desktop app or mobile). A connector added there also works in Claude Code when they're signed in with the same Claude account.
     1. Open **Settings → Connectors** (under Customize).
     2. Click **Add → Add custom connector**, name it **TubeAI**, and paste `https://beta.tubeai.app/api/mcp` as the server URL. Leave Advanced settings empty.
     3. Click **Add**, then **Connect**. TubeAI's authorization page opens: check the account shown and press **Approve**. They need to be logged in to TubeAI in that browser first.
   - **Or in the terminal** (Claude Code): run `claude mcp add --transport http --scope user tubeai https://beta.tubeai.app/api/mcp` for them. Then they type `/mcp`, pick **tubeai**, choose to authenticate, and sign in to TubeAI in the browser that opens.
3. Never add a second TubeAI connection. If TubeAI is already added (in the app's Connectors, or in `/mcp` in Claude Code), it only needs signing in. On Claude Team and Enterprise plans only admins can add connectors in the app, so use the terminal line there.
4. Tools not showing up: in the Claude app, open the TubeAI connector in Settings → Connectors and choose **⋮ → Refresh tools list**, or Disconnect and Connect again. A connector added during a session may only show up in a new chat.
5. Check again with `workspace_account`. If they'd rather not connect, carry on without it (WebFetch, Claude in Chrome and yt-dlp for research), and don't offer it again unless they ask.

## Budget and credits

- Every tool call counts against the plan's daily request budget, which the TubeAI app's database pages share. It resets at 00:00 UTC. Going over it locks the user's database access, in the app too, for 2 days. So:
  - make one well-aimed call rather than many small ones
  - keep `limit` small, and ask for extra `fields` only when you need them (results are slim by default)
  - for many channels at once, use one `explore_videos` call with a channel set and `perChannel`, never one call per channel
  - get the niche labels from `explore_niches` once, then reuse them
- `workspace_account` shows the plan, the credits, what each paid tool costs, which features the plan lacks, and today's budget. Check it before anything that spends credits, when a tool says a feature is locked, or when the user asks about their plan.
- Spending credits needs the user's explicit OK after they've seen the cost: `thumbnail_generate`, `thumbnail_edit` and a board's `dataSources` (Google Trends + X posts). Everything else is free, the other thumbnail tools included.
- A section or list that comes back locked names the plan that includes it. Mention the upgrade only when it matters to what the user wants.

## Playbooks

**Find video ideas** (for the user's channel, a niche, or a saved folder)

1. `mini_ideas` with exactly one scope: `channel` (an @handle or URL), `niche` (a label from `explore_niches`) or `folderId` (from `workspace_folder_get`). Defaults: the last 90 days, videos at 2× their channel's average or more, long-form only (`includeShorts: true` adds Shorts).
2. It returns the outlier videos, the title formats that perform best in that niche, and the words those outlier titles share. Draft ideas from these patterns: a title, the format it borrows, and the angle that makes it the user's own.
3. Check the draft titles with `mini_titles` (`titles`, up to 20). It marks which words and phrases carry proven performance.
4. Save the keepers (see Save the keepers).

**Research a niche**

1. `explore_niches` without arguments lists every niche with its outlier stats. With `niche`, it gives that niche's top channels, top videos and sub-niches.
2. `explore_videos` with `niches` and a `minDate` window, sorted by `outlier_multiplier`, shows what's working there now. `minDuration: 181` leaves Shorts out.
3. `explore_channels` with `niches` plus size and performance filters (`minSubscribers`, `minAvgViews`, `minOutlierRate` as a percent, `isActive`) finds the channels worth watching.

**Study a competitor, or the user's own channel**

- `explore_channel` gives the profile, plus the `include` sections: `recent` and `outliers` by default, and `momentum` and `performance` when the plan has them.
- `explore_similar_channels` finds channels like it.
- A creator's recent outliers: `explore_videos` with `channelHandles: ["@x"]`, `minOutlier: 1`, `minDate` six months back, `sortBy: "outlier_multiplier"`.
- Many channels at once: one `explore_videos` call with `channelHandles` or `channelIds` (up to 50) and `perChannel` (the top N videos of each).

**What's taking off right now**

- `explore_videos` with `similarToHandle: "@x"` (the 100 channels most like it), `minOutlier: 2`, `minDate` 30 days back, `sortBy: "published_at"`.

**Titles and packaging**

- `mini_titles` with `show: ["formats", "words"]` gives proven title structures and high-performing title words. Narrow them with `search` and (formats only) `niches`, and rank them `best_performing`, `most_used` or `trending`.
- To see the packaging: `showThumbnails: true` on `explore_videos` (the first 12 videos), or `showThumbnail: true` on `explore_video`.
- `explore_similar_thumbnails` finds videos whose thumbnails look like one video's thumbnail. It only works for videos TubeAI has analyzed.

**Thumbnails** (they cost credits, so confirm the cost first)

1. Agree on the concept with the user: what the thumbnail shows, and any text on it.
2. `thumbnail_generate` with the video's title or idea (`videoIdea`) and the concept (`thumbnailConcept`). `count` is 2 or 4, and only one generation runs at a time, so ask for every option in one call. With `channelHandle`, TubeAI matches that channel's style from its real thumbnails: then describe only the idea, never colors, fonts or layout.
3. `thumbnail_status`, called once, shows the gallery, which fills in by itself in 1–2 minutes. `thumbnail_view` lets you see the images and gives their `historyId`s.
4. `thumbnail_edit` changes one. The user names it by its number in the gallery, which is the `historyId` at that position.
5. The user's own face: `thumbnail_face_upload` shows an uploader. When they say it's done, generate with `useUploadedFace: true`.

Style matching and the user's face depend on the plan. A locked feature says which plan includes it.

**Draft a script**

- Start from the idea the user picked and the patterns behind it: the hook, the promise the title makes, and the beats that deliver it. Draft it with the user one part at a time, in their channel's voice.
- To study how a hit is scripted, or how a creator actually talks, get transcripts with `mini_script`: `videos` (1–5 ids or URLs, any public video) or `channel` with `count` (its latest uploads). Transcripts TubeAI already has are free; new ones count against a daily transcript cap. An entry marked `aiSummary: true` is a summary of the video, not the words spoken, so never learn a voice from it.
- The user's own past work in TubeAI helps: `workspace_history` with `section: "scripts"` (or `viral`, `metadata`, `analytics`) lists what they made in the app. A script's `historyId` is what `workspace_idea_save` takes as `scriptProposal`.

**Save the keepers**

- Ideas: `workspace_idea_save` creates a private draft in the user's My Ideas planner. Never say it's published: the user reviews and publishes it in TubeAI.
  - A title of 24–58 characters is the planner's sweet spot.
  - Add 2–4 sentences of notes on the angle and why it works now.
  - Add up to 6 reference videos, only ones your research returned.
  - `ideaId` edits an existing idea, and each list you pass replaces the stored one. `workspace_idea_get` lists or opens ideas.
- Channels and videos: confirm the folder name and its contents with the user, then use `workspace_folder_create` or `workspace_folder_add`. A folder holds channels or videos, never both, and takes up to 200 items per call. `workspace_folder_get` lists folders and their ids.
- Anything that didn't save comes back under `unparseable`, `wrongType` or `notFound`. Tell the user instead of reporting a clean save.
- Boards, when the plan has them: `workspace_board_create` builds a planning board from a folder, a list of videos or the channel's niche, and labels it for free. A niche scan can come back as `status: "scanning"`. Tell the user it's in progress, then call again with the returned `resumeOperationId` and every field of `resumeWith`.

## Reading the numbers

- `outlier_multiplier`: a video's views against its own channel's average (3 = three times the usual).
- `runner_multiplier`: views against the channel's subscriber count.
- `outlier_rate` (channels): the percent of a channel's videos that are outliers.
- Shorts run up to 3 minutes, so `minDuration: 181` leaves them out.
- `unknownNiches` and `unresolvedHandles` list what didn't match. Say so rather than treating the results as complete.

## Good to know

- The tool list can show TubeAI's tools with a prefix (like `mcp__tubeai__explore_videos`). Use the name it shows.
- `explore_videos` refuses a call without at least one narrowing filter: `search`, `niches`, a channel set (`channelIds`, `channelHandles` or `similarToHandle`) or `minDate`.
- Name channels by @handle, channel URL or UC id, and videos by id or URL.
- Results page with `cursor` and `nextCursor`.
- A query that comes back as too broad: narrow it (a shorter window, fewer channels, a smaller limit) and try once more.
- **A tool says it needs authentication, or the TubeAI tools are missing:** go back to Connect first.
- **The daily budget is used up:** it resets at 00:00 UTC. Carry on with the fallbacks and tell the user in a line.
- **A tool says database access is paused:** the budget was exceeded. Tell the user; TubeAI support can lift it early.
