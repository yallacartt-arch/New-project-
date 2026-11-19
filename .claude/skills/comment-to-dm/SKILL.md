---
name: comment-to-dm
description: Builds and checks Instagram and Facebook comment-to-DM automations through PostZen. Someone comments a keyword and gets a DM with a link, optionally only after they follow. Two jobs: build a new one, or check whether a live one is working. Requires: the PostZen MCP connected to Claude, an Instagram Business or Creator account or a Facebook Page connected in PostZen, a keyword and a link. Use for "/comment-to-dm", "comment X and I'll send it", "set up the DM automation for this reel", "is my DM automation working".
argument-hint: "[build | check] [keyword] [link]"
---

# /comment-to-dm

Someone comments the keyword. They get a DM with the link. Use it to send a guide, a template, a product page or a newsletter signup.

Two jobs. **Build** a new automation. **Check** one that's live. If it isn't clear which, ask.

Before either: call `listAccounts`. No such tool means PostZen isn't connected (the `/post` skill has the steps). Instagram must be a Business or Creator account. A 403 `platformCapabilityMissing` means the account needs reconnecting: call `createConnectUrl` with its `accountId` and give them the link.

## How PostZen automations behave

- **One automation answers each comment.** An automation on one post beats one that covers every post.
- **Each person gets the DM once** per automation, unless you change `repeatPolicy`.
- **A comment allows one private reply, within 7 days.**
- **Keywords ignore case.** `matchMode: "word"` matches the keyword as a whole word. `"contains"` also matches it inside other words.
- **Follow first is Instagram only.** Instagram reveals whether someone follows only after they message the account. So the flow is: opening DM with a button, they tap it, PostZen checks, then sends the link or a follow request.
- **Facebook**: DM and buttons, no follow rule. **TikTok**: a public reply only, no DM.
- **New automations are live on create** unless `isActive: false`. Edits to a live one apply immediately.

## Build

### 1. Collect four things

1. **Keyword**: one word people will comment.
2. **Link**: where the DM sends them.
3. **What they get**, in a short phrase, the way the user would say it.
4. **Which post**: one post, or every post.

Defaults: `matchMode: "word"`, follow first on (Instagram), one link button, a short public reply.

### 2. Check the keyword

Call `listCommentAutomations` for the account.

- Same keyword already live on every post: scope the new one to its post, and tell them about the old one.
- A short or common word (AI, ME, LINK) on every post will fire on ordinary comments. Warn them. On a single post it's usually fine.
- If the video already says the keyword out loud, keep the keyword and scope to the post.

### 3. Find the post (if scoping)

- Posted or scheduled through PostZen: `listPosts`, match the caption, use its `postId`. This works before the post is live.
- Posted from the Instagram app: `syncExternalPosts` with the account and the reel's URL, then use its `platformPostId`.

Confirm the match with the user.

### 4. Show it and wait

```
Account:       Instagram @handle
Keyword:       GUIDE (whole word)
Post:          <caption start>, or every post
Opening DM:    "Hey! Tap below and I'll send it over."  [Send me the link]
Follow first:  on
DM:            "Here's the guide. Tap below and it's yours."
Button:        [Get the guide] -> <link>
Public reply:  "Sent! Check your DMs."
```

Write the DM the way a person talks, and keep it short. Button labels are 20 characters or fewer. A DM with buttons is 640 characters or fewer.

**Create nothing until they say go.**

### 5. Create it

```
createCommentAutomation({
  accountId: "<id>",
  name: "GUIDE - comment to DM",
  keywords: ["GUIDE"],
  matchMode: "word",
  postId: "<PostZen post id>",        // or platformPostId, or neither for every post
  openingDm: {},
  audience: { followerStatus: "follower", whenUnknown: "verify" },
  dmMessage: "Here's the guide. Tap below and it's yours.",
  buttons: [{ type: "url", title: "Get the guide", url: "<link>" }],
  commentReply: "Sent! Check your DMs.",
  isActive: true
})
```

Leave out `audience` and `openingDm` when follow first is off, and always on Facebook.

Report the automation id, account, keyword and scope. Then tell them: **test it from a second account.** An account can't DM itself, so their own comment won't show them anything.

Want emails too? Point the link at a page with a signup form.

## Check

1. `listCommentAutomations`. Match on name or keyword. No match: list them and ask.
2. **Is it active?** A paused one never fires. This is the most common cause.
3. `listCommentAutomationLogs` for it, once per status: `sent`, `gated`, `pending`, `skipped`, `failed`.
   - **sent**: the DM went out.
   - **gated**: waiting on the person to tap the button or follow. Not a failure.
   - **skipped**: usually someone who already got it.
   - **failed**: quote the error and give the fix.
4. **No logs at all**: nothing matched. Check the keyword and match mode, that it's scoped to the right post, that another automation isn't winning the comment, and that they didn't only test from their own account.
5. Answer plainly: is it live, how many people got the DM, how many are waiting at a gate, and what failed with the fix.

## Rules

- Create or edit nothing before an explicit go.
- Never skip the keyword check.
- Don't call gated or skipped runs broken.
- Never change a keyword that's already been filmed. Scope it.
- No follow rule on Facebook or TikTok.
