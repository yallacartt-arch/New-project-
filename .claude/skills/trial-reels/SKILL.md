---
name: trial-reels
description: Turns one finished video into 5 Instagram trial reels with 5 different hooks, schedules them through PostZen for tomorrow so non-followers see them first, then about 3 days later reads the numbers and names the winner. Requires: the PostZen MCP connected to Claude, a public Instagram Business or Creator account connected in PostZen, 1,000+ followers, and 5 versions of the video with different openings. Use for "/trial-reels", "test hooks on this", "make 5 versions of this", "which hook won".
argument-hint: "[finished video, or 5 versions] | results"
---

# /trial-reels

Instagram shows a trial reel to people who don't follow the account first. Followers only see it if it graduates. That makes it a free test: post 5 versions, see which one strangers watch, show followers the winner.

Two jobs. **Launch** when they have a finished video. **Results** when they say "results" or ask which one won.

Say this once, up front: you write the hooks and schedule the trials, but you don't re-edit the video. The 5 versions have to start differently, because people decide in the first 2 seconds, before they read a caption. Five copies of one file only tests the caption.

## Before either job

1. Call `listAccounts`. No such tool means PostZen isn't connected: the `/post` skill has the steps.
2. Instagram must be connected and be a public Business or Creator account.
3. Ask if they have 1,000+ followers. Instagram generally doesn't offer trial reels below that. If they don't, offer `/post` for a normal reel and stop.

## Launch

### 1. The video

Ask for the video, or their 5 versions, and what it says: the script, or the first three lines and the main point. Don't guess.

### 2. Five hooks

Five different reasons to keep watching, not one sentence reworded:

1. **Result**: lead with the outcome
2. **Secret**: something few people know
3. **Mistake**: what they're doing wrong
4. **Number**: a real figure from the video, never made up
5. **Callout**: name who it's for

For each, write the spoken hook (under 2.5 seconds out loud), the on-screen text (3 to 7 words) and a caption. Label them H1 to H5. Show all five and take edits.

### 3. Match files to hooks

- **5 files**: ask which is which. Don't guess from file names.
- **1 file plus extra hook clips**: list the clips and let them match each to a hook.
- **1 file**: give them the five on-screen hooks to add in their editor (about 30 seconds each in CapCut or Edits) and wait for the files. If the `/edit` skill is set up, offer to make the versions.

### 4. Schedule

Default: tomorrow in their timezone at 9:00 AM, 11:30 AM, 2:00 PM, 4:30 PM and 7:00 PM. Say the times back and wait for a yes.

For each version, upload it (`createMediaPresign`, then `curl -X PUT -H "Content-Type: video/mp4" --upload-file "<path>" "<uploadUrl>"`), then:

```
createPost({
  content: "<that version's caption>",
  mediaItems: [{ url: "<publicUrl>" }],
  scheduledFor: "<ISO 8601 with their offset>",
  timezone: "<their timezone>",
  platforms: [{
    platform: "instagram",
    accountId: "<id>",
    settings: { postType: "reel", trialParams: { graduationStrategy: "MANUAL" } }
  }]
})
```

`MANUAL` keeps every trial away from followers until the user picks. `SS_PERFORMANCE` lets Instagram graduate a trial that does well. Default to `MANUAL`.

### 5. Log it

Write `trial-reels-log.md` in the current folder:

```
# Trial reels: <video>
Scheduled: <date>   Check results: <3 days after the last one>

| Version | Goes live   | Post id | Spoken hook | On-screen text | File |
|---------|-------------|---------|-------------|----------------|------|
| H1      | Wed 9:00 AM | abc123  | ...         | ...            | ...  |
```

Show the table and tell them to come back in about 3 days and type `/trial-reels results`.

## Results

1. Read `trial-reels-log.md`. If it's missing, ask which posts to check.
2. Call `getAnalytics` with each `postId`. For Reels it returns views, likes, shares, saves, `skipRate` and `avgWatchTimeMs`.
3. No numbers yet? Don't guess. Ask them to paste each trial's insights from the Instagram app.
4. Under 48 hours since the last trial went live: say the numbers are early.
5. Show a table: version, hook, views, skip rate, average watch time, shares, saves.
6. Pick the winner. **Views decide. Skip rate breaks ties**, because it measures the hook directly: the share of viewers who left in the first 3 seconds. If the top two are within 15% on views, call it a tie and let them choose.
7. Tell them what to do: graduate the winner from the Instagram app, or post that file fresh with `/post`.
8. Add the results and the winning angle to `trial-reels-log.md`.

## Rules

- No trials for an account that isn't eligible.
- Five hooks means five angles.
- Never say the video was re-edited when it wasn't.
- No winner from estimated numbers.
- The user graduates the winner. You don't.
