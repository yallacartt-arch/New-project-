---
name: analyze-competitors
description: Finds the video ideas that are working in the user's niche right now and turns them into a ranked plan for their next 7 videos, each with a hook. Pulls competitors' recent posts with real view counts, scores each post against that account's own median, keeps the outliers and groups them into patterns. Never invents numbers. Requires: web search, 3 to 10 competitor accounts, and real view counts from Apify (paid), public pages, or numbers the user pastes. PostZen is optional. Use for "/analyze-competitors", "what's working in my niche", "what should I post next", "study these creators".
argument-hint: "[your niche] [@account1 @account2 ...]"
---

# /analyze-competitors

Find what is provably working in the user's niche, then tell them which 7 videos to make next.

The idea behind it: views only mean something next to what that account normally gets. 300K views is an ordinary day for a big account and a breakout for a small one. So score every post against its own account's median and keep the outliers.

**Never invent a view count, date or link.** Every number comes from a scraper, a page you read, or the user. If you can't get real numbers, say so and ask.

## 1. Inputs

Ask for what's missing, one question at a time.

1. **Niche**, in one sentence: who the videos are for and what they're about.
2. **Platform**: Instagram Reels (default), TikTok or YouTube Shorts.
3. **3 to 10 accounts** in that niche. Include some near the user's size. No list? Offer to find accounts by web search, and only suggest ones you confirmed exist and posted in the last 30 days.
4. **Their own handle** (optional), so you skip ideas they already made.

Save the list to `competitors.md` in the current folder.

## 2. Get real posts

For each account you want the last 20 to 30 posts: link, date, views, and what the video is about. Try in this order and say which one you're using.

**A. Apify**, if the user has the Apify MCP or an `APIFY_TOKEN`. Instagram: `apify/instagram-reel-scraper`. TikTok: `clockworks/tiktok-scraper`. Read the actor's input schema first. State the estimated cost and wait for a yes before running.

**B. Public pages.** Search and fetch profile and post pages. TikTok and YouTube usually show views. Instagram usually doesn't to logged-out visitors. Keep a number only if you read it on the page.

**C. The user pastes them.** Always works, about 2 minutes per account. Ask them to open the account's Reels tab, where each reel shows its views, and paste:

```
@handle
https://www.instagram.com/reel/ABC123/  412K  "what it's about"
```

If PostZen is connected, `getAnalytics` with `sortBy: "views"` shows the user's own best posts. Use it to compare, and to avoid repeating what they already made.

Flag any account with fewer than 8 posts as thin data.

## 3. Score

Per account:

1. Take the **median** views.
2. Score each post: views divided by median.
3. Keep posts at **3x or more**. Fewer than 6 outliers in total? Drop the bar to 2x and say so.
4. Drop posts under 2 days old, paid ads, and collabs with much bigger accounts. Say what you dropped.

## 4. Patterns

For each outlier note the idea, the angle (money, shortcut, mistake, secret, before and after, tool reveal), the format, and the hook.

A pattern needs **2 or more outliers from 2 or more accounts**. One viral video is not a pattern. Rank patterns by how many outliers back them and how high the scores are.

Also list what keeps getting posted and never beats the median. Those are ideas to skip.

## 5. The next 7 videos

Turn the patterns into 7 videos for the user's audience. Same winning idea, their angle. Not copies.

For each:

- **The video**, in one line
- **Why it should work**: the pattern and the real posts behind it (account, link, views, score)
- **Spoken hook**: the first sentence, word for word
- **On-screen hook**: 3 to 7 words
- **What to show**: one or two lines

Strongest evidence first. Mix the formats.

## 6. Write the file

Save `competitor-analysis-YYYY-MM-DD.md` in the current folder with these sections: where the numbers came from, the accounts (posts checked, median, outliers), every outlier, the patterns, what isn't working, the next 7 videos.

In chat, give the top 3 patterns and the 7 titles. Offer a full script for any of them.

Next run: read `competitors.md` and the last report first, pull only newer posts, and say what changed.

## Rules

- Real numbers or ask. Never estimate.
- No paid scrape without a stated cost and a yes.
- Rank by score against the account's median, never by raw views.
- One viral video is not a pattern.
