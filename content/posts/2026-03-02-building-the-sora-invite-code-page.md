---
title: "From Zero to 45K+ PV: How I Built a Free Sora2 Invite Code Page"
date: 2026-03-02T00:00:00-08:00
draft: false
---

Today, I shut down https://sora.langbit.ai. It's a single-page webiste that helps people to get free Sora2 invite codes.

Here's what I achieved:
- **45k+** PV
- **23K+** UV
- **1.5k+** people got a valid Sora2 code
- **130+** followers on [Twitter/X](https://x.com/shayne_snape/)
- Donations: two coffees
![Sora invite page screenshot](/images/202603/01.png)

It only took me a few hours to build it with AI on a trip during the Chinese National Day holiday in 2025. I wanna make a quick recap and share how I built it.

Sora2 was amazing at that time, but people needed a invite code to use it.

On the first day, I saw some users selling their invite codes for money which OpenAI didn't encourage. So I decided to build a simple website to help people get free Sora2 invite codes.

## Marketing

I didn’t focus enough on marketing and conversions at first. Here’s what I tried:

- **X**: tweets + replies (Chinese and English both worked; English traffic ~30%)
- **Reddit**: so many scams,  users are cautious.
- **V2EX**: brought some users
- **Bilibili**: commenting under videos brought traffic, and I wasn’t banned 👍
- **Xiaohongshu / Zhihu**: basically impossible. My posts didn’t pass review on Xiaohongshu. On Zhihu, I left a few comments and then was banned for 7 days.
- Some traffic also came from users sharing the link on other forums.

## Technical Details


Data Flow

{{< mermaid >}}
graph TD
  A["YouTube | Bilibili | Reddit | X"] --> B[Crawler Layer]
  B --> C[Normalize + De-dupe]
  C --> D[Upload to Cloudflare Worker]
  D --> E[Durable Objects]
  D --> F[KV Store]
  E --> G[API]
  F --> G
  G --> H[Frontend UI]
{{< /mermaid >}}

#### Multi‑platform crawling

I used Python to build a crawler that supports multiple sources out of the box: YouTube, Bilibili, Reddit, and Twitter/X. The first version ran on my laptop using Selenium to simulate my personal account. It worked well! The crawler normalizes comments, extracts candidate codes, and then removes duplicated invite codes before upload.

#### Render deployment

Then I realized that Bilibili, Reddit, and X were bad data sources(Too many scams and bots). YouTube was a relatively good option. There's GCP YouTube Data API and it offers generous quota,so running it on a server platform became the default choice.

The crawler runs on Render. A lightweight web server exposes a `/trigger` endpoint.This gives me a cheap always‑on way to long‑running jobs without keeping my laptop up.

#### Storage and UI

The Cloudflare Worker is both the API backend and the front‑end renderer. It uses Durable Objects as the primary storage for invite codes (To do consistent updates), and KV for state (votes, user events). The Worker exposes JSON APIs (`/api/add`, `/api/codes`, `/api/stats`, etc.) and also serves the UI.

It's a simple page with a few features, so introuding a front-end web framework wasn't a good choice. I just used native JavaScript.

#### CI/CD

I wired GitHub Actions to auto‑deploy the Cloudflare Worker, crawler on Render, and auto-upload static assets to R2.


![Sora invite page screenshot](/images/202603/02.png)


## Observations & Experiments

- I ran a small ad test by adding an ad bannner in the bottom-right. Result: **brutal**. This kind of traffic won’t pay the bills.
- People who actually share their invite codes are minority.

## Thoughts & Open Questions

I had to choose between “don’t annoy users” and “capture seed users for a future product.”, In the end, it's no login requied!
Later, someone DM’d me about how to better leverage the traffic. I’m still not sure what's the perfect way.

I didn’t figure out a better design to encourage more people to share their invite codes back.

I keep asking:
- How do I balance helping users vs. retaining seed users for the next product?
- Which path works better: a standalone web page, a Twitter thread, or a Telegram/Discord bot?
- Should I add Gmail login? I tried a few similar tools. Most used Gmail login. What would I do?

## The End

The real value comes from the community sharing. But the building process was fun, and it reignited my motivation to build more projects. Thanks to everyone who tried it out, gave feedback, voted, or donated. **Special thanks to those who actually shared invite codes — you made this work**.

And massive shoutout to Cloudflare and Render's free tier. I wrote the code in a few hours and deployed in minutes. They're easy to use.
