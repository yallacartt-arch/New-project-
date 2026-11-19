---
name: post
description: Schedules a finished video or post to Instagram, TikTok, YouTube, Facebook, LinkedIn, X, Threads, Pinterest, Bluesky or Telegram through PostZen. Uploads the file, takes or writes the caption, confirms the time in the user's timezone, schedules it and returns the post id. Requires: the PostZen MCP connected to Claude, at least one social account connected in PostZen, and a finished video file or public URL. Use for "/post", "schedule this", "post this tomorrow at 9", "put this on Instagram and TikTok".
argument-hint: "[video file] [platforms] [time]"
---

# /post

Get a finished video scheduled on the right accounts at the right time, and prove it with a post id. Don't edit the video. Don't rewrite a caption the user gave you.

Everything goes through the PostZen MCP tools: `listAccounts`, `createMediaPresign`, `createPost`, `listPosts`, `createConnectUrl`.

## 1. Check PostZen

Call `listAccounts`.

- No such tool: PostZen isn't connected. Follow "Connect PostZen" at the bottom, then continue.
- Auth error: have them sign in again (`/mcp`, pick postzen, Authenticate).
- An account is missing: call `createConnectUrl` for that platform and give them the link. Instagram must be a Business or Creator account.

## 2. Get three things

Ask only for what's missing.

1. **The video.** A file path or a public URL. If `ffprobe` is installed, check it: vertical 9:16, has audio, under 3 minutes for Reels, under 60 seconds for a YouTube Short. Report problems, don't block on them.
2. **Where.** List their connected accounts and ask which ones.
3. **When.** Accept anything: "tomorrow 9am", "Friday 6pm", "next free slot", "now". Work out their timezone (`date +%Z`, or ask).

Then the caption. Use theirs, or offer to write one: a first line that says what the viewer gets, 2 to 4 short lines, one call to action. 3 to 5 hashtags on Instagram and TikTok, none elsewhere. If a comment-to-DM runs on this post, the call to action is `Comment "KEYWORD" and I'll send it to you`.

## 3. Confirm before scheduling

Say it back in one message and wait for a yes:

```
Instagram @handle, TikTok @handle
Tomorrow, Wed Nov 18, 9:00 AM Eastern
Caption: <first line>...
```

Fix these first if they apply: YouTube needs a title under 100 characters, X is 280 characters, Bluesky is 300, Pinterest needs a board, the time must be at least a minute in the future.

## 4. Upload (local files only)

1. `createMediaPresign` with `filename`, `contentType` (for example `video/mp4`) and `size` in bytes. It returns `uploadUrl` and `publicUrl`.
2. `curl -X PUT -H "Content-Type: video/mp4" --upload-file "<path>" "<uploadUrl>"`
3. Use `publicUrl` from here on.

A public URL under 100 MB can go straight into `mediaItems`.

## 5. Schedule

One `createPost` call covers every platform:

```
createPost({
  content: "<caption>",
  mediaItems: [{ url: "<publicUrl>" }],
  scheduledFor: "2026-11-18T09:00:00-05:00",
  timezone: "America/New_York",
  platforms: [
    { platform: "instagram", accountId: "<id>", settings: { postType: "reel" } },
    { platform: "tiktok", accountId: "<id>", settings: { privacyLevel: "publicToEveryone" } },
    { platform: "youtube", accountId: "<id>", settings: { title: "<title>", privacyStatus: "public" } }
  ]
})
```

- "Now" means `publishNow: true` in place of `scheduledFor`. Only when they said now.
- "Next free slot" means `queuedFromProfile: "<profileId>"` in place of `scheduledFor`.
- Different caption on one platform: `customContent` on that platform.
- TikTok video made with AI: `isAigc: true`.

## 6. Hand back proof

```
Scheduled

| Platform  | Account  | Goes live                   | Post id |
|-----------|----------|-----------------------------|---------|
| Instagram | @handle  | Wed Nov 18, 9:00 AM Eastern | abc123  |
```

No post id means it isn't scheduled. If a platform failed, say which one and why, and keep the rest.

## Rules

- Never publish immediately unless they said "now".
- Never guess an account. Ask.
- Never pass a local file path to `createPost`. Upload it first.
- Always say the time back in their timezone before scheduling.

## Connect PostZen

1. Create a free account at https://www.postzen.dev/?utm_source=jesseeisenbart.com&utm_medium=guide&utm_campaign=claude-social-skills&utm_content=skill-post and connect the social accounts.
2. Claude Code in a terminal: run `claude mcp add --transport http --scope user postzen https://mcp.postzen.dev/mcp`, restart Claude Code, type `/mcp`, pick postzen, choose Authenticate.
3. Claude desktop app or claude.ai: Settings, Connectors, add a custom connector with the URL `https://mcp.postzen.dev/mcp`, then sign in.
4. Ask "list my PostZen accounts" to check.
